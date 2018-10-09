---
published: true
date: '2018-10-09 14:33 -0600'
title: Simpler Segment Routing with NSO and IOS-XR
author: Shelly Cadora
excerpt: Back story for the creation of SR migration service packs
tags:
  - iosxr
  - NSO
position: hidden
---
## How I Learned to Love NSO
When I started working on the [Core Fabric Design](https://xrdocs.io/design/blogs/latest-core-fabric-hld) (a simple, highly available, scalable reference design for the core), I had one overriding goal: whatever features and use cases we covered, the whole thing had to be be model-driven.  CLI is dead to me.  If I can’t do something with a model, I won't do it at all.  I’m happy to say that I (mostly) succeeded.  I also learned some cool tricks along the way and came to appreciate the power of the Network Services Orchestrator ([NSO](https://www.cisco.com/c/en/us/solutions/service-provider/solutions-cloud-providers/network-services-orchestrator-solutions.html)).  

## Yes, I Drank the Model Koolaid.
Eons ago, as a fresh, young engineer at Cisco, I cut my coding teeth on test automation, all of which was done via CLI and screen-scraping.  I learned to love Tcl, expect, and regular expressions of all kinds, and it was a lot of fun to write the test scripts.  What I didn’t like was maintaining them across platforms and releases.  CLI was just too fragile for robust automation.  And I don’t even want to tell you the number of script runs that failed because my script accidentally shut the interface I was connected to (blush).  So when I started working with IOS XR and discovered the power of [YANG](https://tools.ietf.org/html/rfc6020) data models, I was hooked.

Data models provide a way for a device to announce what kind of configuration and operational data it supports, including the type of data (string, integer, IP address, etc) that it will send or receive.  You can work with data models offline with a variety of tools and data models announce changes in advance -- all good things for people writing automation software.

Things get even better when you combine those data models with a standard protocol like [NETCONF](https://tools.ietf.org/html/rfc6241.html), which defines many useful operations.  For instance, if even one line of the config fails, NETCONF can “rollback-on-failure.”  If you request a “confirm-commit," the router will rollback the config changes if it doesn’t get a follow-up confirmation (e.g. because you mistakenly shut the interface that you’re talking to!).   All that makes automation easier and more robust.  No, data models aren’t perfect.  Yes, YANG can be abstract and NETCONF’s reliance on XML can be irritating.  Nevertheless, this clearly is a better way to automate. 

## Modeling Best Practices With NSO
Initially, I started the Validated Core automation work with useful open source tools like [ncclient](https://github.com/ncclient/ncclient) and [ANX](https://github.com/cisco-ie/anx).  But it got tedious after a while, manipulating all that XML and talking to one device at a time, especially as my testbed grew.  Take the [LDP to Segment Routing (SR) Migration](https://xrdocs.io/design/blogs/latest-core-fabric-hld#ldp-to-sr-core-migration) use case.  The basic SR config consists of mostly static content with a handful of variables (IGP instance name, Loopback interface, SR Global Block (SRGB), and the device SID).  What I needed was template that could take the variable data, combine it with the static content and apply the resulting config to many devices at the same time.   It turns out that this is one of the most basic things that NSO can do.  Knowing the ISIS YANG config data model, it didn’t take much time to whip up a basic NSO service using a [NETCONF NED for IOS XR](https://github.com/NSO-developer/nso-xr-segmentrouting/tree/develop/packages/prouter-ned).  Another bonus: after I created the template, I never had to look at XML again.  Life got easier. 

Templates are nice, but what I really needed was an _intelligent_ template, something that could embed best practices in the service itself. For example, it is a common (and best) practice to have the same IGP instance name, Loopback interface and SRGB on every SR device in a given domain.  But nothing in CLI or NETCONF prevents you from accidentally configuring a different SRGB on different routers, which can cause all sorts of problems.  If I could define the common variables once and reuse them whenever the service was deployed, NSO could prevent problems from happening in the first place.  That was the genesis of the “sr-infrastructure” resource in NSO:

**Warning** YANG model follows!
{: .notice--info}

```
module: infrastructure
    +--rw sr-infrastructure!
       +--rw sr-global-block-pools* [name]
       | +--rw name    -> /ralloc:resource-pools/idalloc:id-pool/name
       +--rw instance-name?           string
       +--rw loopback?                uint32
```

In NSO CLI terms, the resources are configured like this:

```
resource-pools id-pool SRGB-POOL1 
  range start 17000 
  range end 18000
sr-infrastructure
  instance-name ISIS-CORE 
  loopback 0 
  sr-global-block-pools SRGB-POOL1
```

Yes, I did say CLI was dead to me...just not NSO CLI!  Seriously though, I put the CLI just for readability.   If you can read XML, knock yourself out:

```
<config xmlns="http://tail-f.com/ns/config/1.0">
  <sr-infrastructure xmlns="http://cisco.com/ns/tailf/cf-infra">
    <sr-global-block-pools>
      <name>SRGB-POOL1</name>
    </sr-global-block-pools>
    <instance-name>ISIS-CORE</instance-name>
    <loopback>0</loopback>
  </sr-infrastructure>
  <resource-pools xmlns="http://tail-f.com/pkg/resource-allocator">
  <id-pool xmlns="http://tail-f.com/pkg/id-allocator">
    <name>SRGB-POOL1</name>
    <range>
      <start>17000</start>
      <end>19000</end>
    </range>
  </id-pool>
  </resource-pools>
</config>
```

The ```sr-infrastructure``` resource by itself is an internal NSO construct.  Nothing is pushed to the router when I commit that to NSO.  I still need to create a service that references this resource.  

Once I realized I could enforce the same SRGB everywhere by using ```sr-infrastructure```, I started thinking about other problems I could prevent.  For example, if you unintentionally assign the same SID to different devices or assign a SID that is out of the SRGB range, unpleasant things will happen (trust me, I’ve done it).  But if my NSO service could auto-assign a unique and valid SID from the defined SRGB range, then I could avoid both of those problems.   

Talking to one of our friendly NSO engineers, I quickly realized that NSO can do all that and more.  Throw a little Java into your service and magical things start to happen.  Not only could NSO auto-assign a unique and valid SID from the SRGB, I could also exclude SIDs from the assignment block (e.g. because I’d already manually assigned some values from the block).  We ended up implementing a simple auto-assignment with exclusions, but the idea could easily be extended to more sophisticated auto-assignment algorithms (e.g. based on some information in a Loopback address or metadata associated with a device’s location or role).

The ```sr``` service ended up looking like this from a YANG perspective:

```
module: sr
  augment/ncs:services:
       +--rwsr* [name]
      +--rwname                        string
       +--rwrouter* [device-name]
          +--rwdevice-name            -> /ncs:devices/device/name
          +--rwprefix-preference
          | +--rw(prefix-choice)?
          |    +--:(auto-assign-prefix-sid)
          |    |  +--rwauto-assign-prefix-sid?   empty
          |    +--:(assign-prefix-sid)
          |        +--rwassign-prefix-sid?       uint16
          +--rwinstance-preference
             +--rw(instance-choice)?
                +--:(use-sr-infrastructure)
                |  +--rwuse-sr-infrastructure?   empty
                +--:(custom-instance)
                   +--rwcustom-instance
                      +--rwinstance-name?   string
                      +--rwloopback?        uint32
```

Which you can configure like this:

```
 services sr Denver 
  router P3 
    instance-preference use-sr-infrastructure 
    prefix-preference auto-assign-prefix-sid
  router P4 
    instance-preference use-sr-infrastructure 
    prefix-preference auto-assign-prefix-sid
```

As you can see, both routers in the ```sr``` service named ```Denver```  will use the same IGP instance name, the same loopback and the same SRGB because the service calls the ```sr-infrastructure``` (“use-sr-infrastructure”).  In addition, the prefix SID will be auto-assigned for both routers (“auto-assign-prefix-sid”). Once committed, this service will roll out SR in a consistent way across the routers in the Denver region.

Another optimization occurred to me while configuring Transport Independent-Loop Free Alternative (TI-LFA).  In ISIS, TI-LFA has to be configured on individual interfaces under the IGP.  Well, guess who forgot to enable it on a couple interfaces and spent an embarrassing amount of time troubleshooting the network?  Yep.  So I asked my NSO buddy if NSO could automatically apply the TI-LFA config to all the interfaces already configured under the IGP.  And while you’re at it, please usethe instance-name I defined in ```sr-infrastructure``` (because, surprise, “isis instance-name Core” is in fact completely distinct from “isis instance-name core” from a router's perspective).  That’s how we ended up with this [service for TI-LFA enablement](https://github.com/NSO-developer/nso-xr-segmentrouting/tree/develop/packages/ti-lfa):

```
services ti-lfa Denver-LFA
 address-family ipv4
 router P3
  instance-name-preference use-sr-infrastructure
  interface-preference all-interfaces
 !
 router P4
  instance-name-preference use-sr-infrastructure
  interface-preference all-interfaces
```

A industry leading network operating system like IOS-XR is both powerful and dangerous: it gives you knobs for just about every functionality imaginable.  But really, most networks want a small and well-defined subset of those knobs (and if yours doesn’t, you might want to start rethinking your design!).  By enforcing those in the NSO service models, you make sure your entire network follows best practices.


## My Life, Simpler
Pretty soon, I had a useful [set of simple services](https://github.com/NSO-developer/nso-xr-segmentrouting/tree/develop/packages) that incrementally rolled out SR in an ISIS/LDP core network.  Along the way, I started using other NSO capabilities.  For example, I could sync the starting config of my network, tinker with it for debugging or optimization, and re-sync to get back to where I started.  Literally, it's as easy as:

```
admin@ncs# devices sync-from
```

Do Bad Stuff Everywhere Outside NSO
{: .notice--danger}

```
admin@ncs# devices sync-to
```

Think "rollback" for your whole network.  Super useful when someone messed up your testbed right before a live demo (I’m looking at you, Wolverine).  

I also discovered NSO's northbound API which enabled me to build RESTCONF calls for all my services.  For example, here is the HTTP to create an SR service via RESTCONF to NSO:

```
POST /restconf/data/services HTTP/1.1
Host: 10.30.111.12:8080
Content-Type: application/yang-data+xml

<sr xmlns="http://cisco.com/tailf/sr">
    <name>Denver</name>
    <router xmlns="http://cisco.com/tailf/sr">
        <device-name>P3</device-name>
        <prefix-preference>
           <auto-assign-prefix-sid/>
        </prefix-preference>
        <instance-preference>
            <use-sr-infrastructure/>
        </instance-preference>
    </router>
</sr>
```

Every resource and service can be configured this way.  Throw it all in a tool like a Postman Runner, and boom, all the services are deployed and the whole LDP to SR migration use case is set up with a click.  Delete the services with a few more RESTCONF calls and boom, a few seconds later, the network is back to its original state.  Now I had an API-driven Ctrl-Z for my network!

I know I only scratched the surface of NSO with the simple services we created for the SP Validated Core Design use cases.  This is a truly powerful platform.  Once you get started, the ideas just keep flowing.   Encode your best practices in service models.  Cover your back with Ctrl-Z for the whole network.  Tie it all into your OSS system with the northbound APIs.  Do all this knowing you have the robustness of data models and NETCONF to keep you out of trouble.  One thing I know for sure is that I’m not going to be doing any design validation without NSO at my side.  

Check out what NSO and IOS XR data models can do for you:
### IOS-XR 
[Data-Model Overview](https://xrdocs.io/programmability/tutorials/2016-09-15-xr-data-model-overview/)
[Model-Driven Programmability](https://xrdocs.io/programmability/blogs/2016-09-12-model-driven-programmability/)
[YANG Models by Release](https://github.com/YangModels/yang/tree/master/vendor/cisco/xr)

### NSO
[NSO on DevNet](https://developer.cisco.com/site/nso/)
[NSO Example Services for Segment Routing](https://github.com/NSO-developer/nso-xr-segmentrouting)


