---
published: true
date: '2023-03-01 00:00 -0600'
title: Cisco Converged SDN Transport SRv6 Transport High Level Design 
author: Phil Bedard 
excerpt: CST SRv6 HLD 
permalink: /blogs/latest-converged-sdn-transport-srv6
tags:
  - iosxr
  - design
  - sr 
  - 5g 
  - transport 
  - cst 
  - srv6 
  - routing 
position: top 
---
{% include toc %}

# Revision History

| Version          |Date                    |Comments| 
| ---------------- | ---------------------- |-----|
| 1.0       | 03/01/2023| Initial version  


# Solution Component Software Versions  

| Element          |Version                    |
| ---------------- | --------------------------|
| IOS-XR Routers (ASR 9000, NCS 5500, NCS 540)      | 7.8.1 | 
| Crosswork Network Controller | 4.1 | 

<br>

# Summary  

Segment Routing has become the de facto underlay transport architecture for 
next-generation IP networks. SR simplifies the underlay network while also 
enhancing it with additional capabilities to carry differentiated services and 
maintain end-to-end SLAs for network deployments such as 5G mobile services.  

Segment Routing is an architecture, with a base set of functions achievable using 
standards based technology. The architecture gives SR the flexibility to adopt 
the best technology for a specific network and the use cases it needs to support.  

One component where SR supports this flexibility is in the data-plane transport 
layer. At its core SR is an underlay transport technology allowing a network to carry 
overlay services, such as L2VPN, EVPN, and simple IP services. The most ubiquitous 
data-plane transport used today in packet networks is MPLS. MPLS originated as Cisco tag switching 
almost 25 years ago (1998) and powers both service provider and enterprise networks worldwide.  
Segment Routing supports MPLS using the SR-MPLS data-plane, where SR SIDs are allocated as 
MPLS labels and is widely deployed today.   

IPv6 is the next-generation of IP addressing, also available for more than two decades.
IPv6 promised simplified networks and services by utilizing the large amount of address 
space to ues IPv6 addressing to easily correlate packet to service. The vision has been 
there but the proper technology did not fully enable it. Segment Routing v6 is the 
technology which not only provides an IPv6-only data-plane across the network, but also creates a 
symmetry between data-plane, overlay services, and performance monitoring. 

SRv6 is the technology for enabling next-generation IPv6 based networks to
support complex user and infrastructure services.   


![](http://xrdocs.io/design/images/ron-hld/ron-cst-overview.png)

# SRv6 Technology Overview  

## SRv6 Benefits 

### Scale 
One of the main benefits of SRv6 is the ability to build networks at huge scale
through address summarization. MPLS networks require a unique label per node be 
distributed to all nodes requiring mutual reachability. In most cases there is also 
the requirement of distributing a /32 IP prefix as well. In the MPLS CST design we 
have the option to eliminate the IP and MPLS label distribution by utilizing a PCE to compute end to end 
paths. While this method works well and is also available for SRv6 it can lead to a large number of 
SR-TE or SRv6-TE policies.  

SRv6 allows us to summarize domain loopback addresses at IGP or BGP boundaries.
This means a domain of 1000 nodes no longer requires advertising 1000 IP
prefixes and associated labels, but can be summarized into a single IP
advertisement reachable via a simple longest prefix match (LPM) lookup.
Networks of tens of thousands of nodes can now provide full reachability with
very few IPv6 routes. See the deployment options section for more information.   

### Simple Forwarding 
In SRv6, if a node is not a terminating node it simply forwards the traffic using 
IPv6 IP forwarding.  This means nodes which are not SRv6 aware can also participate 
in a SRv6 network.  

### Forwarding and Service Congruency 
As you will see in the services section, the destination IPv6 address in SRv6 is 
the service endpoint. Coupled with the simple forwarding this aids in troubleshooting 
and is much easier to understand than the MPLS layered service and data plane.   


## Segment Routing v6 IETF Standards and Drafts 

|IETF Draft or RFC|Description| 
|--------|----|
|RFC 8754| IPv6 Segment Routing Header|
|RFC 8986| SRv6 Network Programming|
|RFC 9252| BGP Overlay Services Based on Segment Routing over IPv6| 
|RFC 9259| SRv6 OAM| 
|draft-ietf-spring-srv6-srh-compression| SRv6 compressed SIDs (uSID) |  
|ietf-lsr-isis-srv6-extensions|IS-IS extensions to support SRv6| 
|ietf-lsr-ospfv3-srv6-extensions|OSPFv3 extensions to support SRv6| 
|draft-ppsenak-lsr-igp-pfx-reach-loss|Unreachable Prefix Announcement| 

## IPv6 Segment Routing Header (SRH) 
Defined in RFC 8754, the SRv6 header includes the SRv6 SID list along with
additional attributes to program the SRv6 IPv6 data plane path. The SRH may be
inserted by the source node or a SRv6 termination point. The SRH is not required
in all SRv6 use cases such as a simple L3VPN or L2VPN with no traffic
engineering requirements. Unlike MPLS, the SRv6 end IPv6 address can be used to
identify the endpoint node and the service. 

## SRv6 Locator 
The SRv6 locator is part of the SRv6 SID structure of Locator:Function:Argument. 
The locator is allocated to nodes acting as SRv6 endpoints.  The locator is defined as
a specific amount of bits of the IPv6 address steering traffic to the endpoint node.  

## SRv6 Compressed SID (micro-SID / uSID)
SRv6 is made more efficient with the use compressed SIDs. Compressed SIDs are 
also known as micro-SIDs (uSID).  In the case of micro-SID, multiple SRv6 SIDs 
can be encoded in a single 128-bit SRv6 SID. Additional SRv6 SIDs can be included 
in the path by adding an additional SRH if necessary.  

### SRv6 micro-SID Terminology 

|Item|Definition|
|-----|----------|
|Compressed-SID (C-SID)|A short encoding of a SID in an SRv6 packet that does not include the SID locator block bits|
|uSID|A Micro SID. A type of Compressed-SID referred as NEXT-C-SID| 
|uSID Locator Block|A block of uSIDs|
|uSID Containe|A 128-bit SRv6 SID that contains a sequence of uSIDs. It can be encoded in the DA of an IPv6 header or at any position in the Segment List of an SRH|
|Active uSID|First uSID after the uSID locator block|
|Next uSID|Next uSID after the Active uSID|
|Last uSID|From left to right, the last uSID before the first End-of-Container uSID|
|End-of-Container (EoC) uSID|Reserved uSID (all-zero ID) used to mark the end of a uSID container.  All the empty uSID container positions must be filled with the End-of-Container ID|


## SRv6 Addressing with Compressed SID 
Using the compressed SID or micro-SID format requires defining the IPv6 address
structure segmenting the IPv6 address into a C-SID portion and Locator block 
portion. A dedicated IPv6 prefix should be used for SRv6 and SRv6 micro-SID allocation. 
RIR assigned public prefixes can be utilized or private ULA space defined in RFC4193.   

### SRv6 uSID Carrier Format 
Micro-SID requires defining a carrier format used globally across the network. 
A parent block size is dedicated to micro-SID allocations and the length of each 
micro-SID. IOS-XR supports the F3216 format, defining a 32-bit micro-SID block and 16-bit 
ID format.    

### Global C-SID Block (GIB) 
The global ID block defines the block of SIDs used to identify nodes 
either uniquely or as part of an anycast group. These addresses are used by other 
nodes on the network to send SRv6 service traffic to the endpoint with the Global 
C-SID assigned.  

### Local C-SID Block (LIB)
The local ID block is used to define SIDs which are local to a node. These are 
typically used to identify services terminating on a node.   
## Baseline SRv6 Forwarding Behavior
Forwarding in SRv6 follows the semantics of simple IPv6 routing. The destination 
is always identified as an IPv6 address. In the case of an SRv6 packet without an 
additional SRH, traffic is routed to the endpoint destination node hop by hop 
based on normal destination based prefix lookups. In the case of a SRH, the SRH 
is only processed by a node if it is the destination address in the outer IPv6 
header. If the node is the last SID in the SID list it will pop the SRH and process
the packet further.  If the node is not the last SID in the SID list it will replace
the outer IPv6 destination address with the next IPv6 address in the SID list.   

### Compressed SID without SRH  
Compressed SIDs have the ability to instantiate a multi-hop SRv6 path using a 
single 128-bit IPv6 address. Each micro-SID in the F3216 format uses 16 bits 
to identify the next node. If the node receives the packet with its own address 
as the IPv6 destination address it will further process the packet. It will either 
shift the micro-SID component of the address 16 bits to the left and copy the new 
address into the IPv6 destination address or if the locator is local to the node 
further process the service packet.  Using the F3216 carrier format in IOS-XR, 
upto 6 micro-SIDs can be encoded in a single 128-bit IPv6 address.  

The example below illustrates the forwarding behavior with no additional SRH. 

### Compressed SID with additional SRv6 Headers  
Using additional SRv6 headers increases the depth of the micro-SID list to support 
use cases with longer traffic engineered paths. This allows SRv6 with micro-SID to 
enable path hop counts greater than SR-MPLS.  

### TI-LFA Mid-Point Protection
SRv6 fully supports Topology Independent Loop-Free Alternates ensuring fast traffic
protection the case of link and node failures. A per-prefix pre-computed loop-free backup path 
is created. If the path requires traversing links which may end up in a loop, the 
protecting node will insert an SRH with the appropriate SIDs to reach the Q node 
with loop-free reachability to the destination prefix.   



# SRv6 Deployment Overview  
## Scalable Deployment using Domain Summarization and Redistribution 
In the CST design we utilize separate IGP instances to segment the network.
These segments can be based on scale, place in the network, or for other
administrative reasons. We do not recommend exceeding 2000 routers in a single
IGP domains.  

SRv6 and its summarization capabilities are ideal for building high scale
networks based on the CST design.  At each domain boundary the IPv6 locator 
blocks are summarized and redistributed across adjacent domains. The end to end 
redistribution of summary prefixes enabled reachability between any two nodes on 
the network by simply doing a longest-prefix match on the destination address.  



### Mutual Redistribution with Redundant Connectivity  
Redistribution between IGP domains interconnected by multiple links requires
additional consideration. While summary prefixes may not affect intra-domain
connectivity, if they are redistributed back into the domain initially
distributing them routing loops may occur. It's important to make sure the
router is properly configured to not violate the split-horizon rule of
advertising routes back to the same domain they received them from. See the IGP
implementation section for more details.   

### Unreachable Prefix Announcement
Summarization hides the state of longer prefixes within the aggregate
summary, leading to traffic loss or slower failover when an egress PE is
unreachable. UPA is an IGP function to quickly poison a prefix which has become
unreachable to an upstream node. It enables the notification of an individual
prefix becoming unreachable, outside of the local area/domain and across the
network in a manner that does not leave behind any persistent state in the
link-state database. When an ingress PE receives the UPA for an egress PE it can 
trigger fast switchover to an alternate path, such as a BGP PIC pre-programmed 
backup path.   

### SR Flexible Algorithms
Flexible Algorithms or Flex-Algo is an important component in SRv6 networks. SRv6 
enables advanced forwarding behavior without utilizing SR-TE Policies, increasing 
scale and simplifying network deployments.  Flex-Algo is used in the CST SRv6 
design to differentiate traffic based on latency or path constraints.   

## SRv6-TE Policies for Advanced TE  
SRv6 supports the same TE Policy functionality as SR-MPLS.  In cases where more advanced 
TE is required than Flex-Algo provides, such as defining explicit paths or requiring 
a path be disjoint from another path, SRv6-TE can be utilized. In CST SRv6 1.0, 
on-demand networking can be used for supported services with SR-PCE to compute both 
intra-domain and inter-domain paths.  Provisioning, visualization, and monitoring of 
SRv6-TE paths is available in Crosswork Network Controller 4.1.  

 

## SRv6 Network Functions and Endpoint Behaviors 
RFC 8986 defines a set of SRv6 endpoint behaviors satisfying specific network 
service functions. The table below defines a base set of functions and the identifiers 
used. See RFC 8986 for details on each behavior.   

|Behavior Identifier|Behavior Description| 
|--------|----|
|End| SRv6 version of prefix-SID|
|End.X| L3 cross-connect, SRv6 version of Adj-SID|
|End.T| IPv6 table lookup| 
|End.DX6| Decapsulate and perform IPv6 cross-connect, per-CE IPv6 L3VPN use case| 
|End.DX4| Decapsulate and perform IPv4 cross-connect, per-CE IPv4 L3VPN use case| 
|End.DT6| Decapsulate and perform IPv6 route lookup, per-VRF IPv6 L3VPN use case| 
|End.DT4| Decapsulate and perform IPv4 route lookup, per-VRF IPv4 L3VPN use case| 
|End.DT46| Decapsulate and perform IPv4 or IPv6 route lookup, per-VRF IP L3VPN use case| 
|End.DX2| Decapsulate and perform L2 cross-connect, P2P L2VPN use case| 
|End.DX2V| Decapsulate and perform L2 VLAN lookup, P2P L2VPN using VLANs use case| 
|End.DT2U| Decapsulate and perform unicast MAC lookup, L2VPN ELAN use case| 
|End.DT2M| Decapsulate and perform L2 flooding, L2VPN ELAN use case| 
|End.B6.Encaps| Identifies SRv6 SID bound to a SRv6 Policy, binding SID | 
|End.B6.Encaps.Red| End.B6 with reduced SRH| 
|End.BM| Endpoint bound to an SR-MPLS Policy| 

### SRv6 Compressed SID Behavior 

When SRv6 SID compression is used enhanced methods are used when processing 
End SRv6 packets. Since multiple SIDs are encoded in a single IPv6 address as the argument 
component of the SRv6 address, a portion of the argument equal to the SID length 
is copied into a specific portion of the IPv6 destination address matching the 
next node in the path. 

|Micro-SID Behavior Identifier|Behavior Description| 
|--------|----|
|uN| NEXT-CSID End behavior with shift and lookup (Prefix-SID) |
|uA| NEXT-CSID End.X behavior with shift and xconnect (Adj-SID) |
|uDT| NEXT-CSID End.DT behavior (End.DT4/End.DT6/End.DT2U/End.DT2M)|
|uDX| NEXT-CSID End.DX behavior (End.DX4/End.DX6/End.DX2) |


# SRv6 Network Implementation 

Implementing SRv6 in the network requires the following steps:  

* Network Domain Planning 
* SRv6 Network Address Planning 
* SRv6 Router Configuration 
* SRv6 Enabled Services Configuration  


## Network Domain Planning 
Networks require segmentation to scale. The initial step in designing scalable networks is 
to determine where network boundaries. This leads directly to IGP segmentation of the network 
utilized in the CST SRv6 design. In our example network we have three domains, two access domains 
and one core domain. Each of these IGP domains is assigned a unique instance identifier. 


![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-igp-layout.png)

## SRv6 Network Address Planning 
SRv6 using micro-SID requires allocating the appropriate parent IPv6 prefix and then further 
delegation based on the micro-SID carrier format. We will use the F3216 format, using 
a 32 bit block and 16 bits for each SID. It is recommended the parent prefix, such as a /24 
be allocated ONLY for SRv6 use. The block can be from public IPv6 resources or utilize a ULA 
private block.   

Specific segments of the IPv6 address can be used to represent areas of the network, flexible algorithms, 
or other network data.  It is important to use the specific bits or bytes of the address used as identifiers 
to also aid in summarization. 

### Locator Planning and Formatting  
The SRv6 locator identifies a node and its specific services. SRv6 using micro-SID should use a specific 
locator format that adheres to the micro-SID carrier format and lends itself to summarization at network boundaries. 
The following is a recommended way to define the locator format which allows for efficient summarization.  It is 
required to use a locator prefix length of /48 on all nodes when using the F3216 carrier format.   


![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-locator-example.png)

The example locator above encodes the following information: 

|Identifier|Bit Length|Usage| 
|--------|----|----|
|fccc:00|24|Base SRv6 locator prefix used network wide |
|XY|8|Together identifies the uSID block| 
|X|4|General-use identifier, NCS platforms require this byte be set to 0 |
|Y|4|In our case the X portion identifies Flexible Algorithm, 0-3F usable for global SIDs |
|ZZ|8|Identifies domain| 
|NN|8|Identifies node|

In our example network, we have a /32 micro-SID prefix allocated network wide for each Flex-Algo. This 
is recommended as it quickly allows operators to identify the FA being used and promotes more efficient summarization. However, if FA is not being used these bits could be used for a different identifier. 

The 8-bit domain identifier allows 255 domains, the 8-bit node identifier allows 255 nodes per domain. This 
is flexible however, the structure could be shifted to allow less domains and more nodes per domain. 

Using the schema above our example address is as follows: 

|Identifier|Value|Meaning| 
|--------|----|----|
|fccc:00|24|Base SRv6 locator prefix used network wide |
|X|0|General-use identifier, NCS platforms require this byte be set to 0 |
|Y|1|Flexible Algorithm 128 |
|ZZ|02|Domain 102| 
|NN|15|Node assigned identifier 15|

This allows each domain's SRv6 SIDs to be summarized per flex-algo at the /40 prefix length. 


## Router Configuration
The CST design supports using SRv6 micro-SID only. Legacy SRv6 using the 128-bit 
carrier format is not supported.  

### SRv6 Micro-SID Hardware Enablement
On NCS 540 and NCS 5500 platforms the following command enables SRv6 with 
micro-SID carrier. 

<div class="highlighter-rouge">
<pre class="highlight"> 
hw-module profile segment-routing srv6 mode micro-segment format f3216
</pre>
</div>

### Loopback Interface Configuration 
While not required, it is recommended to use an IPv6 address from the Algo 0 (base IGP)
locator address block for the Loopback interface.  

<div class="highlighter-rouge">
<pre class="highlight"> 
interface Loopback0
 ipv4 address 101.0.2.53 255.255.255.255
 ipv6 address fccc:0:214::1/128
!
</pre>
</div>


### PE Locator Configuration 
SRv6 enabled routers terminating services must have SRv6 locators configured.  In our 
Flex-Algo use case we will have a single locator configured for each Flex-Algo, although 
more locators can be configured as needed. The unode behavior is set to psp-usd which 
performs penultimate-segment-popping and ultimate-segment-decapsulation.  See RFC 8986 for 
more information on these behaviors. 

Locators are used to allocate both static and dynamic /64 SIDs used for services and link adjacency SIDs. 
The SIDs used for dynamic allocation are in the *e0000-ffff* range in bits 48-63 of the IPv6 address.  

In this case the locator value is assigned based on the following as identified in the addressing section:   

|Identifier|Value| 
|--------|----|
|Domain | Access 2 | 
|Global Base SRv6 Block | FCCC:00::/24| 
|Global FA 0 Block| FCCC:00<b>00</b>::/32| 
|Global FA 128 Block| FCCC:00<b>01</b> ::/32|
|Global FA 129 Block| FCCC:00<b>02</b>::/32| 
|Global FA 130 Block| FCCC:00<b>03</b>::/32| 
|Global FA 131 Block| FCCC:00<b>04</b>::/32| 
|Access Domain 2| FCCC:00XX:<b>02</b>::/40| 
|Unique node in Access Domain 2| FCCC:00XX:02<b>14</b>::/48| 

**PE Locator Router Configuration** 

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 srv6
  encapsulation
   source-address fccc:0:214::1
  !
  locators
   locator LocAlgo0
    micro-segment behavior unode psp-usd
    prefix fccc:0:214::/48
   !
   locator LocAlgo128
    micro-segment behavior unode psp-usd
    prefix fccc:1:214::/48
    algorithm 128
   !
   locator LocAlgo129
    micro-segment behavior unode psp-usd
    prefix fccc:2:214::/48
    algorithm 129
   !
   locator LocAlgo130
    micro-segment behavior unode psp-usd
    prefix fccc:3:214::/48
    algorithm 130
   !
   locator LocAlgo131
    micro-segment behavior unode psp-usd
    prefix fccc:4:214::/48
    algorithm 131
   !
  !
 !
!
</pre>
</div>

**Configured Locators** 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe3#show segment-routing srv6 locator
Thu Jan  5 17:27:30.127 UTC
Name                  ID       Algo  Prefix                    Status   Flags
--------------------  -------  ----  ------------------------  -------  --------
LocAlgo0              1        0     fccc:0:214::/48           Up       U
LocAlgo128            2        128   fccc:1:214::/48           Up       U
LocAlgo129            3        129   fccc:2:214::/48           Up       U
LocAlgo130            4        130   fccc:3:214::/48           Up       U
LocAlgo131            5        131   fccc:4:214::/48           Up       U
</pre>
</div>

**Dynamic Micro-SID Service SIDs based on Locator**

As shown below the primary locator for Algo 0 is identified as fccc:0:103::/48. Each 
service has one or more SIDs allocated starting at fccc:0:103:e000::/64. 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe3#show segment-routing srv6 sid
Thu Jan  5 17:30:55.250 UTC

*** Locator: 'LocAlgo0' ***

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
fccc:0:103::                uN (PSP/USD)      'default':259                     sidmgr              InUse  Y
fccc:0:103:e000::           uDT2U             4550:0                            l2vpn_srv6          InUse  Y
fccc:0:103:e001::           uDT2M             4550:0                            l2vpn_srv6          InUse  Y
fccc:0:103:e002::           uDX2              4600:600                          l2vpn_srv6          InUse  Y
fccc:0:103:e003::           uDX2              650:650                           l2vpn_srv6          InUse  Y
fccc:0:103:e004::           uDT4              'l3vpn-v4-srv6'                   bgp-100             InUse  Y
</pre>
</div>



### PE IS-IS Router Configuration 
SRv6 is the IPv6 data plane for Segment Routing and utilizes the same SID 
distribution semantics as SR-MPLS. This is achieved through IGP extensions responsible 
for distributing SID reachability within an IGP domain or even across IGP domains.  The CST 
design utilizes IS-IS. 

It is recommended to utilize IS-IS as an IPv6 IGP. OSPFv3 is not widely deployed 
in networks today and typically lags behind IS-IS in feature development. 
{: .notice--warning}

In this example we are utilizing the base Algo 0 and four additional Algos, 
128,129,130,131.  Each requires a separate Locator be applied to a Loopback interface
on the router. The locator names are those defined in the earlier SRv6 base configuration section.  


<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
 address-family ipv6 unicast
  metric-style wide
  microloop avoidance segment-routing
  router-id Loopback0
  segment-routing srv6
   locator LocAlgo0
   !
   locator LocAlgo128
   !
   locator LocAlgo129
   !
   locator LocAlgo130
   !
   locator LocAlgo131
   !
  !
  maximum-redistributed-prefixes 10000 level 2
 !
 interface Loopback0
  address-family ipv6 unicast
  !
 ! 
 interface TenGigE0/0/0/9
  bfd fast-detect ipv6
  point-to-point
  hello-password keychain ISIS-KEY
  address-family ipv6 unicast
   fast-reroute per-prefix
   fast-reroute per-prefix ti-lfa
  !
 !
!
</pre>
</div>

### Domain Boundary IS-IS Configuration 
The multiple instance IS-IS configuration is similar to the SR-MPLS CST design. 
The primary differences are the use of summarization, redistribution between instances, 
and the use of UPA. 


#### Unreachable Prefix Advertisement Configuration 
The UPA configuration is enabled by configuring the UPA parameters under prefix-unreachable and 
enabling the "adv-unreachable" for the summary prefix.  The adv-metric sets the metric of the unreachable 
prefix and the adv-lifetime sets the amount of time it should be advertised in milliseconds. Prefixes 
generated by UPA can also be limited to those with a specific IGP tag by using the *unreachable-component-tag* 
option. As an example Loopback and SRv6 Locator addresses can be tagged so they generate a UPA but infrastructure 
links are omitted.   

#### Summary Prefix Configuration 

A flex-algo algorithm can be attached to the summary prefix and used in path
computation. Using the *explicit* keyword means only SRv6 prefix SIDs with the
specified algorithm attached will be considered as contributing prefixes for the
summary. 

#### Core and Access Mutual Redistribution

![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-isis-redist.png)

Redistribution between IGP instances should always utilize route policies with 
appropriate prefix-sets or tags to restrict the prefixes advertised between domains. 
  
Link-state IGP protocols only allow prefix summarization at boundaries such 
as IS-IS areas, levels, or OSPF area boundaries. Summarization can also be 
performed on external prefixes redistributed from another protocol or IGP instance.
In our case the summary-prefix configuration for the ACCESS IGP instances 
is in the CORE instance configuration. Tagging can be used to filter multiple 
prefixes based on a single tag, such as all prefixes belonging to a specific 
domain.  

UPA prefixes must also be redistributed beyond domain boundaries. The longer UPA
component prefix is generated in the CORE IS-IS instance, and must be
redistributed into the appropriate remote instance. As an example if
fccc:0\:100\:1/128 in Access Domain 1 is unreachable, a UPA is generated in the
CORE domain for the /128 prefix, and then must be redistributed in Access domain
2 so ingress PEs can more quickly switch to an alternative path. In our example
BGP next-hop for service routes is the /128 Loopback address assigned from Algo
0, so we must leak /128 UPAs from the core into each access domain matching the
remote domain prefixes.  This can be simplified by using tagging instead of 
strict prefix-lists.     


**Access-1 to Core Boundary** 

<div class="highlighter-rouge">
<pre class="highlight">
prefix-set ACCESS1-PE-uSID
  fccc:0:100::/40 eq 48,
  fccc:1:100::/40 eq 48,
  fccc:2:100::/40 eq 48,
  fccc:3:100::/40 eq 48,
  fccc:4:100::/40 eq 48
end-set

prefix-set ACCESS2-PE-uSID-Summary
  fccc:0:200::/40,
  fccc:1:200::/40,
  fccc:2:200::/40,
  fccc:3:200::/40,
  fccc:4:200::/40
end-set

prefix-set ACCESS2-PE-uSID-UPA
  fccc:0:200::/40 eq 128
end-set

route-policy CORE-TO-ACCESS1-SRv6
  if destination in ACCESS2-PE-uSID-Summary then
    pass
  else
    drop
  endif
end-policy

route-policy ACCESS1-TO-CORE-SRv6
  if destination in ACCESS1-PE-uSID then
    pass
  else
    drop
  endif
end-policy

router isis ACCESS
 address-family ipv6 unicast
  prefix-unreachable
   adv-metric 4261412866
   adv-lifetime 1000
   rx-process-enable
  !
  summary-prefix fccc::/40 tag 100 adv-unreachable 
  redistribute isis CORE route-policy CORE-TO-ACCESS1-SRv6
</pre>
</div>

In the ACCESS instance configuration the summary prefix is equivalent to 
fccc:0000:00/40 as IOS-XR removes trailing zeroes from the address in the configuration. 
{: .notice--warning}

<div class="highlighter-rouge">
<pre class="highlight">
router isis CORE
 address-family ipv6 unicast
  prefix-unreachable
   adv-metric 4261412866
   adv-lifetime 1000
   rx-process-enable
  !
  summary-prefix fccc:0:100::/40 tag 101 adv-unreachable
  summary-prefix fccc:1:100::/40 algorithm 128 tag 101 adv-unreachable explicit
  summary-prefix fccc:2:100::/40 algorithm 129 tag 101 adv-unreachable explicit 
  summary-prefix fccc:3:100::/40 algorithm 130 tag 101 adv-unreachable explicit 
  summary-prefix fccc:4:100::/40 algorithm 131 tag 101 adv-unreachable explicit 
  redistribute isis ACCESS route-policy ACCESS1-TO-CORE-SRv6
  !
 !
!
</pre>
</div>



**Access-2 to Core Boundary** 

<div class="highlighter-rouge">
<pre class="highlight">
prefix-set ACCESS2-PE-uSID
  fccc:0:200::/40 le 48,
  fccc:1:200::/40 le 48,
  fccc:2:200::/40 le 48,
  fccc:3:200::/40 le 48,
  fccc:4:200::/40 le 48
end-set

prefix-set ACCESS1-PE-uSID-Summary
  fccc:0:100::/40,
  fccc:1:100::/40,
  fccc:2:100::/40,
  fccc:3:100::/40,
  fccc:4:100::/40
end-set

prefix-set ACCESS1-PE-uSID-UPA 
  fccc:0:100::/40 eq 128
end-set

route-policy CORE-TO-ACCESS2-SRv6
  if destination in ACCESS1-PE-uSID-Summary then
    pass
  else
    drop
  endif
end-policy

route-policy ACCESS2-TO-CORE-SRv6
  if destination in ACCESS2-PE-uSID then
    pass
  else
    drop
  endif
end-policy
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
 address-family ipv6 unicast
  prefix-unreachable
   adv-metric 4261412866
   adv-lifetime 1000
   rx-process-enable
  !
  summary-prefix fccc::/40 tag 100 adv-unreachable 
  redistribute isis CORE route-policy CORE-TO-ACCESS2-SRv6
</pre> 
</div> 

<div class="highlighter-rouge">
<pre class="highlight">
router isis CORE
 address-family ipv6 unicast
  prefix-unreachable
   adv-metric 4261412866
   adv-lifetime 1000
   rx-process-enable
  !
  summary-prefix fccc:0:200::/40 tag 102 adv-unreachable
  summary-prefix fccc:1:200::/40 algorithm 128 tag 102 adv-unreachable explicit
  summary-prefix fccc:2:200::/40 algorithm 129 tag 102 adv-unreachable explicit 
  summary-prefix fccc:3:200::/40 algorithm 130 tag 102 adv-unreachable explicit 
  summary-prefix fccc:4:200::/40 algorithm 131 tag 102 adv-unreachable explicit 
  redistribute isis ACCESS route-policy ACCESS2-TO-CORE-SRv6
  !
 !
!
</pre>
</div>



### SRv6-TE Policies 
SRv6 supports Traffic Engineering policies, using different metric types and 
additional path constraints to engineer traffic paths from head-end to endpoint 
node. SRv6 TE Policy configuration follows the same configuration as SR-MPLS 
TE policies. 

In the CST SRv6 design 1.0 dynamic path calculation is supported using SR-PCE.
Please see the CST implementation guide for PCE configuration details.
Additionally, in CST SRv6 1.0 path computation for SRv6-TE Policies requires a path 
of nodes supporting for SRv6, if the only path is via a node not supporting 
SRv6, path calculation will fail.    

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  policy srte_c_21009_ep_fccc:0:215::1
   srv6
    locator LocAlgo0 binding-sid dynamic behavior ub6-insert-reduced
   !
   source-address ipv6 fccc:0:103::1
   color 21009 end-point ipv6 fccc:0:215::1
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type igp
      !
     !
    !
   !
  !
 !
!
</pre>
</div>

**Operational Details** 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe3#show segment-routing traffic-eng policy color 21009
Thu Jan  5 23:42:24.222 UTC

SR-TE policy database

Color: 21009, End-point: fccc:0:215::1
  Name: srte_c_21009_ep_fccc:0:215::1
  Status:
    Admin: up  Operational: up for 13:36:55 (since Jan  5 10:05:28.918)
  Candidate-paths:
    Preference: 100 (configuration) (active)
      Name: srte_c_21009_ep_fccc:0:215::1
      Requested BSID: dynamic
      PCC info:
        Symbolic name: cfg_srte_c_21009_ep_fccc:0:215::1_discr_100
        PLSP-ID: 52
      Constraints:
        Protection Type: protected-preferred
        Maximum SID Depth: 7
      Dynamic (pce 101.0.1.101) (valid)
        Metric Type: IGP,   Path Accumulated Metric: 260
          SID[0]: fccc:0:108::/48 Behavior: uN (PSP/USD) (48)
                  Format: f3216
                  LBL:32 LNL:16 FL:0 AL:80
                  Address: fccc:0:108::1
          SID[1]: fccc:0:e::/48 Behavior: uN (PSP/USD) (48)
                  Format: f3216
                  LBL:32 LNL:16 FL:0 AL:80
                  Address: fccc:0:e::1
          SID[2]: fccc:0:215::/48 Behavior: uN (PSP/USD) (48)
                  Format: f3216
                  LBL:32 LNL:16 FL:0 AL:80
                  Address: fccc:0:215::1
      SRv6 Information:
        Locator: LocAlgo0
        Binding SID requested: Dynamic
        Binding SID behavior: End.B6.Insert.Red
  Attributes:
    Binding SID: fccc:0:103:e014::
    Forward Class: Not Configured
    Steering labeled-services disabled: no
    Steering BGP disabled: no
    IPv6 caps enable: yes
    Invalidation drop enabled: no
    Max Install Standby Candidate Paths: 0
</pre>
</div>

**Explicit segment list definition** 

In the case of building a policy with explicit paths, the sid-format must be defined so the appropriate uSID container can be 
populated with each node SID in the path.  

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  segment-lists
   srv6
    sid-format usid-f3216
   !
   segment-list APE7-srv6
    srv6
     index 1 sid fccc:0:109::
     index 2 sid fccc:0:20f::
     index 3 sid fccc:0:215::
    !
   !
  !
 !
!
</pre>
</div>


### On-Demand SRv6-TE Policy 
On-demand next-hop or ODN is supported for SRv6. In the 1.0 version of the CST 
SRv6 design, ODN paths must be computed using SR-PCE. 

In this case the dynamic binding SID for the policy is associated with the flex-algo Algo128 
locator and has a constraint to use algorithm 128.  

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  on-demand color 6128
   srv6
    locator LocAlgo128 binding-sid dynamic behavior ub6-insert-reduced
   !
   source-address ipv6 fccc:1:103::
   dynamic
    pcep
    !
    metric
     type igp
    !
   !
   constraints
    segments
     sid-algorithm 128
    !
   !
  !
 !
!
</pre>
</div>

## Enabling Services over SRv6  
Segment Routing with the IPv6 data plane is used to support all of the services 
supported by the MPLS data plane while also enabling advanced functionality not 
capable in MPLS based networks. In CST SRv6 1.0 L3VPN, EVPN-ELAN, and EVPN-VPWS services are 
supported using SRv6 micro-SID. 

### Services Route Reflector Design 
The Converged SDN Transport design makes use of a set of service BGP route reflectors (S-RRs) communicating 
BGP service routes between PE nodes in a scalable and resilient manner. In CST SRv6 1.0 SRv6 service routes 
are expected from an IPv6 peer.  

### SRv6 Service Forwarding 
MPLS uses a multi-label stack to carry overlay VPN services over an MPLS underlay 
network. There is always at least a 2-label stack identifying the egress node and 
specific underlay service or service entity.  

SRv6 utilizes the flexibility of IPv6 to encode the service information without 
multiple layers. Since the Locator assigned to the egress node is a /48 all traffic within 
the /48 will reach the node. This leaves the remaining 80 bits to be used for identifying 
services and service components.  IOS-XR will dynamically assign a /64 out of the /48 Locator 
for services.  As an example a L3VPN with per-VRF SRv6 SID allocation will be assigned a /64 as will 
an EVPN-VPWS service.   

In the simplest forwarding use case the ingress node simply sets the SRv6 packet destination 
address to the IPv6 service address.  It is forwarded hop by hop based on the IPv6 DA, meaning 
intermediate nodes not SRv6 aware can also forward the traffic. 

#### L3VPN Forwarding Example 

In the single-domain example below, a VPNv4 L3VPN is configured between routers 
R1 and R3. R3 advertises the VPNv4 prefix with the appropriate parameters required 
for R1 to properly build the packet to R3.  As you can see, there is no SRH involved
in this simple example, all of the information is encoded in the VPNv4 advertisement 
to allow R1 to use a single IPv6 destination address to send traffic to the appropriate
service. 

Traffic will be routed across the proper Flex-Algo path. R4 will utilize
standard LPM (longest prefix match) routing using the Algo 128 topology.     

The uDT4 behavior means "decapsulate the packet and perform an IPv4 routing lookup". The local 
SID fccc:1:215:e004::/64 is assigned to the specific L3VPN VRF.  


![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-l3vpn-forwarding.png)

### L3VPN Configuration Example 
This is an example of a 3-node L3VPN using SRv6. Each node has already been assigned 
a SRv6 Locator to be used with this L3VPN.  In this case we are using the Locator 
defined for the base algo, LocAlgo0. Service SIDs will be allocated from the 
fccc::103::/48 block.  This service is carrying IPv4 routes over SRv6 and utilizes the  
uDT4 behavior type.    

#### Egress PE Configuration 

**Locator Configuration**
<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 srv6
  locators
   locator LocAlgo0
    micro-segment behavior unode psp-usd
    prefix fccc:0:103::/48
</pre>
</div> 

**VRF Configuration** 
The VRF configuration is identical to non-SRv6 use cases. 

**BGP Configuration**
SRv6 must be explicitly enabled for services utilizing SRv6.   
In IOS-XR 7.8.1 per-vrf is the only SID allocation mode supported. 

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 vrf l3vpn-v4-srv6
  rd 100:6001
  address-family ipv4 unicast
   segment-routing srv6
    locator LocAlgo0
    alloc mode per-vrf
   !
   redistribute connected
  !
 !
!
</pre>
</div> 

**SID Allocation** 
Here we see the two SIDs allocated to each address family.  

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe3#show segment-routing srv6 sid detail fccc:0:103:e004::
Sun Jan  8 16:54:01.080 UTC

*** Locator: 'LocAlgo0' ***

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
fccc:0:103:e004::           uDT4              'l3vpn-v4-srv6'                   bgp-100             InUse  Y
  SID Function: 0xe004
  SID context: { table-id=0xe0000005 ('l3vpn-v4-srv6':IPv4/Unicast) }
  Locator: 'LocAlgo0'
  Allocation type: Dynamic
  Created: Nov 28 20:20:21.184 (5w5d ago)
</pre>
</div> 

#### L3VPN Route on Ingress Node 
Here we see the route received from the egress node. There is a new SubTLV containing 
the SRv6 encoding type and we can see the service SID has been encoded as part of the 
BGP label TLV.  

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe8#show bgp vrf l3vpn-v4-srv6 64.4.4.0/24 detail
BGP routing table entry for 64.4.4.0/24, Route Distinguisher: 100:6001
Versions:
  Process           bRIB/RIB  SendTblVer
  Speaker             2215286      2215286
    Flags: 0x20041012+0x00000000;
Last Modified: Dec 20 03:42:26.946 for 2w5d
Paths: (1 available, best #1)
  Not advertised to any peer
  Path #1: Received by speaker 0
  Flags: 0x2000000085060005+0x00, import: 0x39f
  Not advertised to any peer
  Local
    fccc:0:103::1 (metric 120) from fccc:0:216::1 (101.0.1.50), if-handle 0x00000000
      Received Label 0xe0040
      Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported
      Received Path ID 0, Local Path ID 1, version 2215286
      Extended community: RT:100:6001
      Originator: 101.0.1.50, Cluster list: 101.0.2.202, 101.0.0.200, 101.0.1.201
      PSID-Type:L3, SubTLV Count:1, R:0x00,
       SubTLV:
        T:1(Sid information), Sid:fccc:0:103::, F:0x00, R2:0x00, Behavior:63, R3:0x00, SS-TLV Count:1
         SubSubTLV:
          T:1(Sid structure):
           Length [Loc-blk,Loc-node,Func,Arg]:[32,16,16,0], Tpose-len:16, Tpose-offset:48
      Source AFI: VPNv4 Unicast, Source VRF: l3vpn-v4-srv6, Source Route Distinguisher: 100:6001

</pre>
</div> 

#### Forwarding Entry on Ingress Node  

In the forwarding entry we see the SRv6 encapsulation with a SID list of the 
service address. This address is used as a the destination address of the IPv6 
packet sent from the ingress router. The router will utilize the /40 summary 
to reach the end service address.   

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe8#show cef vrf l3vpn-v4-srv6 64.4.4.0/24
64.4.4.0/24, version 11, SRv6 Headend, internal 0x5000001 0x30 (ptr 0x8c326328) [1], 0x0 (0x0), 0x0 (0x9016d028)
 Updated Dec 20 03:42:26.875
 Prefix Len 24, traffic index 0, precedence n/a, priority 3
  gateway array (0xa4a8b360) reference count 1, flags 0x2010, source rib (7), 0 backups
                [1 type 3 flags 0x48441 (0xa5ba6658) ext 0x0 (0x0)]
  LW-LDI[type=0, refc=0, ptr=0x0, sh-ldi=0x0]
  gateway array update type-time 1 Dec 20 03:42:26.874
 LDI Update time Dec 20 03:42:26.874

  Level 1 - Load distribution: 0
  [0] via fccc:0:103::/128, recursive

   via fccc:0:103::/128, 619 dependencies, recursive [flags 0x6000]
    path-idx 0 NHID 0x0 [0x8df4b168 0x0]
    next hop VRF - 'default', table - 0xe0800000
    next hop fccc:0:103::/128 via fccc:0:100::/40
    SRv6 H.Encaps.Red SID-list {fccc:0:103:e004::}

    Load distribution: 0 1 (refcount 1)

    Hash  OK  Interface                 Address
    0     Y   TenGigE0/0/0/8            fe80::28a:96ff:fe4a:8078
    1     Y   TenGigE0/0/0/9            fe80::28a:96ff:fec7:e878


RP/0/RP0/CPU0:cst-a-pe8#show route ipv6 fccc:0:103:e004::

Routing entry for fccc:0:100::/40
  Known via "isis ACCESS", distance 115, metric 120
  Tag 1003, type level-2
  Installed Dec  1 01:53:02.153 for 5w3d
  Routing Descriptor Blocks
    fe80::28a:96ff:fe4a:8078, from fccc:0:e::1, via TenGigE0/0/0/8, Protected, ECMP-Backup (Local-LFA)
      Route metric is 120
    fe80::28a:96ff:fec7:e878, from fccc:0:e::1, via TenGigE0/0/0/9, Protected, ECMP-Backup (Local-LFA)
      Route metric is 120
  No advertising protos.
</pre>
</div> 

### L2VPN EVPN-VPWS 
EVPN-VPWS is configured similar to the MPLS use case with the exception of 
specifying the transport type as srv6.  

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe3#show run l2vpn xconnect group EVPN-VPWS-SRv6_MH
Sun Jan  8 17:22:09.140 UTC
l2vpn
 xconnect group EVPN-VPWS-SRv6_MH
  p2p EVPN-VPWS-SRv6_MH
   interface TenGigE0/0/0/5.600
   neighbor evpn evi 4600 service 600 segment-routing srv6
   !
  !
 !
!
</pre>
</div> 

#### L2VPN EVPN-VPWS State 
The SRv6 behavior of uDX2 means micro-SID behavior with L2 cross-connect. This 
is a multi-homed service on the remote side, so there are two service SIDs 
listed as remote endpoints. 

<div class="highlighter-rouge">
<pre class="highlight">
Group EVPN-VPWS-SRv6_MH, XC EVPN-VPWS-SRv6_MH, state is up; Interworking none
  AC: TenGigE0/0/0/5.600, state is up
    Type VLAN; Num Ranges: 1
    Rewrite Tags: []
    VLAN ranges: [600, 600]
    MTU 1500; XC ID 0x264; interworking none
    Statistics:
      packets: received 480066518, sent 480034205
      bytes: received 480066514256, sent 479074130038
      drops: illegal VLAN 0, illegal length 0
  EVPN: neighbor ::ffff:10.0.0.1, PW ID: evi 4600, ac-id 600, state is up ( established )
    XC ID 0xc0000031
    Encapsulation SRv6
    Encap type Ethernet
    Ignore MTU mismatch: Enabled
    Transmit MTU zero: Enabled
    Reachability: Up

      SRv6              Local                        Remote
      ----------------  ---------------------------- --------------------------
      uDX2              fccc:0:103:e002::            fccc:0:214:e002::
                                                     fccc:0:215:e002::
      AC ID             600                          600
      MTU               1514                         0
      Locator           LocAlgo0                     N/A
      Locator Resolved  Yes                          N/A
      SRv6 Headend      H.Encaps.L2.Red              N/A
    Statistics:
      packets: received 480034205, sent 480066518
      bytes: received 479074130038, sent 480066514256
</pre>
</div> 


### EVPN ELAN 
In this case we are using Algorithm 128, the low latency Flex Algo for the end 
to end path between the ingress node and egress node.  

#### Egres PE EVPN Configuration 

<div class="highlighter-rouge">
<pre class="highlight">
evpn 
  evi 4500 segment-routing srv6
    bgp
    route-target import 100:4500
    route-target export 100:4500
    !
    advertise-mac
    !
    locator LocAlgo128
  !
!
</pre>
</div> 

#### Egress PE BVI Configuration 

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 bridge group srv6_evpn_MH
  bridge-domain srv6_evpn_MH_1
   interface Bundle-Ether25.4500
   !
   evi 4500 segment-routing srv6
   !
  !
 !
!
</pre>
</div> 

#### Egress PE SRv6 SID Allocation 
The EVPN detailed output shows the two SRv6 SIDs allocated for the service.
These are allocated from the Algo128 Locator block. Unicast and BUM are handled
by different labels in MPLS based EVPN, and with SRv6 use two 
separate SIDs one for the uDT2U unicast behavior and one for the uDT2M multicast 
behavior.    

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:cst-a-pe7#show evpn evi vpn-id 4500 detail

VPN-ID     Encap      Bridge Domain                Type
---------- ---------- ---------------------------- -------------------
4500       SRv6       srv6_evpn_MH_1               EVPN
   Stitching: Regular
   Unicast SID:  fccc:1:215:e000::
   Multicast SID:  fccc:1:215:e001::
   E-Tree: Root
   Forward-class: 0
   Advertise MACs: Yes
   Advertise BVI MACs: No
   Aliasing: Enabled
   UUF: Enabled
   Re-origination: Enabled
   Multicast:
     Source connected   : No
     IGMP-Snooping Proxy: No
     MLD-Snooping Proxy : No
   BGP Implicit Import: Enabled
   VRF Name:
   SRv6 Locator Name: LocAlgo128
   Preferred Nexthop Mode: Off
   BVI Coupled Mode: No
   BVI Subnet Withheld: ipv4 No, ipv6 No
   RD Config: none
   RD Auto  : (auto) 101.0.2.52:4500
   RT Auto  : 100:4500
   Route Targets in Use           Type
   ------------------------------ ---------------------
   100:4500                       Import
   100:4500                       Export

*** Locator: 'LocAlgo128' ***

SID                         Behavior          Context                           Owner               State  RW
--------------------------  ----------------  --------------------------------  ------------------  -----  --
fccc:1:215:e000::           uDT2U             4500:0                            l2vpn_srv6          InUse  Y
fccc:1:215:e001::           uDT2M             4500:0                            l2vpn_srv6          InUse  Y
</pre>
</div> 


## MPLS and SRv6 Migration 
In 7.8.1 IOS-XR supports two transition technologies used to interwork
between traditional MPLS and SRv6 domains. Interworking between different data plane  
types can take place using transport layer interworking or service layer interworking. 
The transition methods supported in CST SRv6 1.0 and IOS-XR 7.8.1 utilize service 
interworking and support IPv4 and IPv6 L3VPN services.   

### SRv6 and MPLS Service Interworking Gateway 
IETF draft draft-agrawal-spring-srv6-mpls-interworking-10 covers the semantics of 
the SRv6 and MPLS gateway functionality. A gateway node translates L3VPN service routes 
and their associated data plane forwarding information between SR-MPLS and SRv6 endpoints.  
In CST SRv6 1.0 the ASR 9000 and NCS 5500 series support this functionality using per-VRF MPLS label and 
SRv6 SID allocation.   

![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-service-interworking.png)


<div class="highlighter-rouge">
<pre class="highlight">
vrf gw-l3vpn-v4-srv6
 address-family ipv4 unicast
  enable label-mode
  segment-routing srv6
  import route-target
   100:6200 stitching
   100:6210
  !
  export route-target
   100:6200 stitching
   100:6210
  !
 !
!
router bgp 100
 nsr
 bgp router-id 101.0.0.3
 bgp redistribute-internal
 bgp graceful-restart
 segment-routing srv6
  locator LocAlgo0
 !
neighbor 101.0.2.202
  use neighbor-group SvRR
  address-family vpnv4 unicast
   <b>import reoriginate stitching-rt</b>
   route-reflector-client
   <b>advertise vpnv4 unicast re-originated</b>
   next-hop-self
  !
  address-family vpnv6 unicast
   <b>import reoriginate stitching-rt</b> 
   route-reflector-client
   <b>advertise vpnv6 unicast re-originated</b> 
  !
!
 neighbor fccc:0:10::1
  use neighbor-group SvRR-Client-srv6
  address-family vpnv4 unicast
   <b>import stitching-rt reoriginate</b> 
   route-reflector-client
   <b>encapsulation-type srv6</b> 
   <b>advertise vpnv4 unicast re-originated stitching-rt</b> 
   next-hop-self
  !
  address-family vpnv6 unicast
   <b>import stitching-rt reoriginate</b>
   route-reflector-client
   <b>encapsulation-type srv6</b>
   <b>advertise vpnv6 unicast re-originated stitching-rt</b> 
   next-hop-self
  !
 !
</pre>
</div>

The interworking gateway uses the concept of a stitching Route Target to identify 
prefixes requiring re-origination using the opposite data plane. L3VPN prefixes are 
advertised to the IPv4 BGP neighbor using the MPLS data plane and prefixes advertised 
to the SRv6 BGP neighbor using the SRv6 data plane. The gateway node is always in the 
data path for service prefixes being translated.  

**The folowing shows the data plane in the MPLS to SRv6 direction** 

![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-interworking-dataplane.jpeg)

**In-Depth SRv6 to MPLS Service Interworking Documentation** 

<https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r7-8/segment-routing/configuration/guide/b-segment-routing-cg-asr9000-78x/configure-srv6-micro-sid.html#id_133508> 

### Dual-Connected PE 
Dual Connected PE allows the seamless migration of L3VPN PE routers from the
MPLS to SRv6 data plane. In the dual-connected PE scenario the PE can
communicate with MPLS and SRv6 neighbors within the same VRF. By default, IOS-XR
will advertise prefixes using an MPLS label, advertising with an SRv6 SID
requires the *encapsulation-type srv6* keyword. In the case below, routes
advertised to the 101.0.2.202 neighbor will use the default MPLS label, routes
advertised to fccc:0:10::1 will use the SRv6 encapsulation. The PE node will
properly process incoming packets using the MPLS label or SRv6 SID.   

<div class="highlighter-rouge">
<pre class="highlight">
vrf dual-l3vpn-v4-srv6
 address-family ipv4 unicast
  enable label-mode
  segment-routing srv6
  import route-target
   100:7200 
   100:7210
  !
  export route-target
   100:7200 
   100:7210
  !
 !
!
router bgp 100
 segment-routing srv6
  locator LocAlgo0
 !
neighbor 101.0.2.202
  use neighbor-group SvRR-MPLS-ONLY
  address-family vpnv4 unicast
   route-reflector-client
   next-hop-self
  !
  address-family vpnv6 unicast
   route-reflector-client
   next-hop-self
  !
!
 neighbor fccc:0:10::1
  use neighbor-group SvRR-SRV6-ONLY
  address-family vpnv4 unicast
   route-reflector-client
   <b>encapsulation-type srv6</b> 
   next-hop-self
  !
  address-family vpnv6 unicast
   route-reflector-client
   <b>encapsulation-type srv6</b>
   next-hop-self
  !
 !
</pre>
</div>


## SRv6 Automation 

### Crosswork Network Controller 4.1 
Crosswork Network Controller 4.1 supports the provisioning and visualization of 
SRv6 domains and SRv6-TE Policies. CNC also supports provisioning L2VPN EVPN-VPWS 
and L3VPN services utilizing an SRv6 data plane.  

**The Traffic Engineering dashboard gives summary information for all TE path types** 

![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-cnc-dashboard.png)

**Selecting an SRv6-TE Policy will highlight the end to end path across domains**

![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-cnc-srv6-policy-selection.png)

**Selecting the ellipses and "details" will show details about the policy**

We can now see the full details of the SRv6-TE policy incuding the uSID list
which has been created to build the end-to-end path across IGP instances. Since
we are crossing two domains we have two additional micro-SIDs at cst-pa1 and
cst-pe3. The last SID is the egress node, a-pe7.  

![](http://xrdocs.io/design/images/cst-srv6/cst-srv6-cnc-srv6-policy-details.png)


# Additional Resources 

**Converged SDN Transport Design** 

High Level Design Guide: <https://xrdocs.io/design/blogs/latest-converged-sdn-transport-hld> 
Implementation Guide: <https://xrdocs.io/design/blogs/latest-converged-sdn-transport-ig> 

**Cisco Segment Routing Home** 

<https://segment-routing.net> contains many blogs, demo videos, and configuration 
guides for SRv6 and SRv6 Micro-SID.  

**SRv6 CCO Documentation** 

SRv6 Micro-SID Configuration Guide for ASR 9000: <https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r7-8/segment-routing/configuration/guide/b-segment-routing-cg-asr9000-78x/configure-srv6-micro-sid.html><br>
SRv6 Traffic Engineering Configuration Guide for ASR 9000: <https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r7-8/segment-routing/configuration/guide/b-segment-routing-cg-asr9000-78x/configure-srv6-traffic-engineering.html><br> 


SRv6 Micro-SID Configuration Guide for NCS 5500: <https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/78x/b-segment-routing-cg-ncs5500-78x/configure-srv6-micro-sid.html><br> 
SRv6 Traffic Engineering Configuration Guide for NCS 5500: <https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/78x/b-segment-routing-cg-ncs5500-78x/configure-srv6-traffic-engineering.html><br> 

