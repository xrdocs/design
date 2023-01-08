---
published: true
date: '2022-01-07 15:22 -0600'
title: Cisco Converged SDN Transport SRv6 Transport High Level Design 
author: Phil Bedard 
excerpt: CST SRv6 HLD  
tags:
  - iosxr
  - design
  - sr 
  - 5g 
  - transport 
  - cst 
  - srv6 
  - routing 
position: hidden 
---
{% include toc %}

# Revision History

| Version          |Date                    |Comments| 
| ---------------- | ---------------------- |-----|
| 1.0       | 01/10/2023| Initial version  


# Solution Component Software Versions  

| Element          |Version                    |
| ---------------- | --------------------------|
| Router IOS-XR      | 7.8.1 | 
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
SRvv6-TE pahs is available in Crosswork Network Controller 4.0.  

 

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


# SRv6 OAM 

## SRv6 Path Tracing 

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
|WX|16|Together identifies the uSID block| 
|W|8|General-use identifier, NCS platforms require this byte be set to 0 |
|X|8|In our case the X portion identifies Flexible Algorithm, 0-3F usable for global SIDs |
|ZZ|16|Identifies domain| 
|NN|16|Identifies node|

In our example network, we have a /32 micro-SID prefix allocated network wide for each Flex-Algo. This 
is recommended as it quickly allows operators to identify the FA being used and promotes more efficient summarization. However, if FA is not being used these bits could be used for a different identifier. 

The 16-bit domain identifier allows 255 domains, the 16-bit node identifier allows 255 nodes per domain. This 
is flexible however, the structure could be shifted to allow less domains and more nodes per domain. 

Using the schema above our example address is as follows: 

|Identifier|Value|Meaning| 
|--------|----|----|
|fccc:00|24|Base SRv6 locator prefix used network wide |
|W|0|General-use identifier, NCS platforms require this byte be set to 0 |
|X|1|Flexible Algorithm 128 |
|ZZ|02|Domain 102| 
|NN|15|Node assigned identifier 15|

This allows each domain's SRv6 SIDs to be summarized per flex-algo at the /40 prefix length. 


## Router Configuration
The CST design supports using SRv6 micro-SID only. Legacy SRv6 using the 128-bit 
carrier format is not supported.  

### SRv6 Micro-SID Hardware Enablement
On NCS 540, 5500, and 5700 platforms the following commands enable SRv6 with 
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
|Global FA 131 Block| FCCC:00i<b>04</b>::/32| 
|Access Domain 2| FCCC:00XX:<b>02</b>::/40| 
|Unique node in Access Domain 2| FCCC:00XX:02<b>14</b>::/48| 


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

The configuration below is from the PE3 boundary node between the core and access 2 domains. 


#### Unreachable Prefix Advertisement Confguration 
The UPA configuration is enabled by configuring the UPA parameters under prefix-unreachable and 
enabling the "adv-unreachable" for the summary prefix.  The adv-metric sets the metric of the unreachable 
prefix and the adv-lifetime sets the amount of time it should be advertised in milliseconds.  

#### Summary Prefix Configuration 

A flex-algo algorithm can be attached to the summary prefix and used in path
computation. Using the *explicit* keyword means only SRv6 prefix SIDs with the
specified algorithm attached will be considered as contributing prefixes for the
summary. 

#### Core and Access Mutual Redistribution 
Redistribution between IGP instances should always utilize route policies with 
appropriate prefix-sets or tags to restrict the prefixes advertised between domains. 
  
Link-state IGP protocols only allow prefix summarization at boundaries such 
as IS-IS areas, levels, or OSPF area boundaries. Summarization can also be 
performed on external prefixes redistributed from another protocol or IGP instance.
In our case the summary-prefix configuration for the ACCESS IGP instances 
is in the CORE instance configuration. Tagging can be used to filter multiple 
prefixes based on a single tag, such as all prefixes belonging to a specific 
domain.   

**Boundary Router 1** 

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

**Boundary Router 2** 

<div class="highlighter-rouge">
<pre class="highlight">
prefix-set ACCESS2-PE-uSID
  fccc:0:200::/40 eq 48,
  fccc:1:200::/40 eq 48,
  fccc:2:200::/40 eq 48,
  fccc:3:200::/40 eq 48,
  fccc:4:200::/40 eq 48
end-set

prefix-set ACCESS1-PE-uSID-Summary
  fccc:0:100::/40,
  fccc:1:100::/40,
  fccc:2:100::/40,
  fccc:3:100::/40,
  fccc:4:100::/40
end-set

router isis ACCESS
 address-family ipv6 unicast
  prefix-unreachable
   adv-metric 4261412866
   adv-lifetime 1000
  !
  summary-prefix fccc::/40 tag 100 adv-unreachable
  redistribute isis CORE route-policy CORE-TO-ACCESS2-SRv6
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
TE policies. In the CST SRv6 design 1.0 dynamic path calculation is supported using SR-PCE.   

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

<pre>
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

## MPLS to SRv6 Migration 


## Enabling Services over SRv6  
Segment Routing with the IPv6 data plane is used to support all of the services 
supported by the MPLS data plane while also enabling advanced functionality not 
capable in MPLS based networks. In CST SRv6 1.0 L3VPN, EVPN-ELAN, and EVPN-VPWS services are 
supported using SRv6 micro-SID.   

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




## Pluggable Digital Coherent Optics 

Simple networks are easier to build and easier to operate. As networks scale to
handle traffic growth, the level of network complexity must decline or at least
remain flat. 

IPoDWDM has attempted to move the transponder function into the router to remove
the transponder and add efficiency to networks. In lower bandwidth applications,
it has been a very successful approach. CWDM, DWDM SFP/SFP+, and CFP2-DCO
pluggable transceivers have been used for many years now to build access,
aggregation, and lower speed core networks. The evolution to 400G and
advances in technology created an opportunity to unlock this potential
in higher speed networks.   

Transponder or muxponders have typically been used to aggregate multiple 10G or
100G signals into a single wavelength. However, with reach limitations, and the
fact transponders are still operating at 400G wavelength speeds, the transponder
becomes a 1:1 input to output stage in the network, adding no benefit. 

The Routed Optical Networking architecture unlocks this efficiency for networks
of all sizes, due to advancements in coherent plugable technology. 

## QSFP-DD and 400ZR and OpenZR+ Standards   

As mentioned, the industry saw a point to improve network efficiency by shifting
coherent DWDM functions to router pluggables. Technology advancements have
shrunk the DCO components into the standard QSFP-DD form factor, meaning no
specialized hardware and the ability to use the highest capacity routers
available today.  ZR/OpenZR+ QSFP-DD optics can be used in the same ports as the
highest speed 400G non-DCO transceivers. 

### Cisco OpenZR+ Transceiver (QDD-400G-ZRP-S)
![](http://xrdocs.io/design/images/ron-hld/zrp.png)

### Cisco OIF 400ZR Transceiver (QDD-400G-ZR-S)
![](http://xrdocs.io/design/images/ron-hld/zr.png)

Two industry optical standards have emerged to cover a variety of use cases. The
OIF created the 400ZR specification,
<https://www.oiforum.com/technical-work/hot-topics/400zr-2> as a 400G interopable
standard for metro reach coherent optics. The industry saw the benefit of the
approach, but wanted to cover longer distances and have flexibility in
wavelength rates, so the OpenZR+ MSA was created, https://www.openzrplus.org.
The following table outlines the specs of each standard. ZR400 and OpenZR+ transceivers 
are tunable across the ITU C-Band, 196.1 To 191.3 THz.  

![](http://xrdocs.io/design/images/ron-hld/zr_zrp_specs.png)

The following part numbers are used for Cisco's ZR400 and OpenZR+ MSA transceivers 

|Standard|Part| 
|--------|----|
400ZR| QDD-400G-ZR-S| 
|OpenZR+| QDD-400G-ZRP-S|

Cisco datasheet for these transceivers can be found at <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/datasheet-c78-744377.html> 

## Cisco Routers

We are at a point in NPU development where the pace of NPU bandwidth growth has
outpaced network traffic growth. Single NPUs such as Cisco's Silicon One have a
capacity exceeding 12.8Tbps in a single NPU package without sacrificing
flexibility and rich feature support. This growth of NPU capacity also brings
reduction in cost, meaning forwarding traffic at the IP layer is more
advantageous vs. a network where layer transitions happen often.  

Cisco supports 400ZR and OpenZR+ optics across the NCS 540, NCS 5500, NCS 5700,
ASR 9000, and Cisco 8000 series routers. This enabled providers to utilize the architecture 
across their end to end infrastructure in a variety of router roles. See   

![](http://xrdocs.io/design/images/ron-hld/npu_bandwidth.png)


## Cisco Private Line Emulation 

Starting in Routed Optical Networking 2.0, Cisco now supports Private Line
Emulation (PLE) hardware and IOS-XR support to provide bit-transparent private
line services over the converged packet network. Private Line Emulation supports
the transport of Ethernet, SONET/SDH, OTN, and Fiber Channel services. See the
PLE section of the document for in-depth information on PLE.   
## Circuit Style Segment Routing 
Circuit Style Segment Routing (CS-SR) is another Cisco advancement bringing 
TDM circuit like behavior to SR-TE Policies. These policies use deterministic 
hop by hop routing, co-routed bi-directional paths, hot standby protect paths, 
and end to end liveness detection. Standard Ethernet services not requiring bit 
transparency can be transported over a Segment Routing network similar to OTN 
networks without the additional cost, complexity, and inefficiency of an OTN 
network layer.   
## Cisco DWDM Network Hardware  

Routed Optical Networking shifts an expensive and now often redundant
transponder function into a pluggable transceiver. However, to make the most
efficient use of a valuable resource, the underlying fiber optic network, we
still need a DWDM layer. Routed Optical Networking is flexible enough to work
across point to point, ROADM based optical networks, or a mix of both. Cisco
multiplexers, amplifiers, and ROADMs can satisfy any network need.  

*Cisco NCS 1010* 

Routed Optical Networking 2.0 introduces the new Cisco NCS 1010 open optical line system. 
The NCS 1010 represents an evolution in open optical line systems, utilizing the same 
IOS-XR software as Cisco routers and NCS 1004 series transponders. This enables the rich 
XR automation and telemetry support to extend to the DWDM photonic line system. The NCS 1010 
also simplifies how operators build DWDM networks with advanced integrated functions and a flexible 
twin 1x33 WSS.  

See the validated design hardware section for more information.  


<br>


# Routed Optical Networking Network Use Cases

Cisco is embracing Routed Optical Networking in every SP router role. Access,
aggregation, core, peering, DCI, and even PE routers can be enabled with high
speed DCO optics. Routed Optical Networking is also not limited to SP networks,
there are applications across enterprise, government, and education networks.  

![](http://xrdocs.io/design/images/ron-hld/network-use-cases.png)

## Where to use 400ZR and where to use OpenZR+

The OIF 400ZR and OpenZR+ MSA standards have important differences. 

400ZR supports 400G rates only, and targets metro distance point to point
connections up to 120km. 400ZR mandates a strict power consumption of 15W as
well. Networks requiring only 400G over distances less than 120km may benefit
from using 400ZR optics. DCI and 3rd party peering interconnection are good use
cases for 400ZR.  

If a provider needs flexibility in rates and distances and wants to standardize
on a single optics type, OpenZR+ can fulfill the need. In areas of the network
where 400G may not be needed, OpenZR+ optics can be run at 100G or 200G.
Additionally, hardware with QSFP-DD 100G ports can utilize OpenZR+ optics in
100G mode.  This can be ideal for high density access and aggregation networks.

## Supported DWDM Optical Topologies

For those unfamiliar with DWDM hardware, please see the overview of DWDM network
hardware in [Appendix A](#appendix-a) 
{: .notice--warning}

The future of networks may be a flat L3 network with simple point to point
interconnection, but it will take time to migrate to this type of architecture.
Routed Optical Network supports an evolution to the architecture by working over
most modern photonic DWDM networks.  Below gives just a few of the supported
optical topologies including both point to point and ROADM based DWDM networks.

### NCS 2000 64 Channel FOADM P2P Deployment 
This example provides up to 25.6Tb on a single network span, and highlights the
simplicity of the Routed Optical Networking solution.  The "optical" portion of
the network including the ZR/ZR+ configuration can be completed in a matter of
minutes from start to finish.   

![](http://xrdocs.io/design/images/ron-hld/optical-zr-p2p.png)

### NCS 1010 64 Channel FOADM P2P Deployment 
The NCS 1010 includes two add/drop ports with embedded bi-directional EDFA
amplifiers, ideal for connecting the new MD-32-E/O 32 channel, 150Ghz spaced
passive multiplexer. Connecting both even and odd multiplexers allows the use of 
64 total channels. 

![](http://xrdocs.io/design/images/ron-hld/optical-zr-p2p-1010.png)
### NCS 2000 Colorless Add/Drop Deployment 
Using the NCS2K-MF-6AD-CFS colorless NCS2K-MF-LC module along with the LC16 LC
aggregation module, and SMR20-FS ROADM module, a scalable colorless add/drop
complex can be deployed to support 400ZR and OpenZR+.   

![](http://xrdocs.io/design/images/ron-hld/optical-zrp-colorless.png)

### NCS 2000 Multi-Degree ROADM Deployment 
In this example a 3 degree ROADM node is shown with a local add/drop degree. The
Routed Optical Networking solution fully supports ROADM based networks with
optical bypass. The traffic demands of the network will dictate the most
efficient network build. In cases where an existing or new build requires DWDM
switching capability, ZR and ZR+ wavelengths are easily provisioned over the
infrastructure.  

![](http://xrdocs.io/design/images/ron-hld/optical-zrp-roadm.png)


### NCS 1010 Multi-Degree Deployment
A multi-degree NCS 1010 site utilizes a separate NCS 1010 OLT device for each 
degree. The degree may be an add/drop or bypass degree. In our example Site 3 
can support the add/drop of wavelengths via its A/D ports on the upper node, or
express those wavelengths through the interconnect to site 4 via the additional 
1010 OLT unit connected to site 4. In our example the wavelength originating at 
sites 1 and 4 using ZR+ optics is expressed through site 3.  

![](http://xrdocs.io/design/images/ron-hld/optical-zr-bypass-1010.png)


### Long-Haul Deployment 
Cisco has demonstrated in a physical lab 400G OpenZR+ services provisioned
across 1200km using NCS 2000 and NCS 1010 optical line systems. 300G, 200G,
and 100G signals can achieve even greater distances. OpenZR+ is not just for
shorter reach applications, it fulfills an ideal sweet spot in most provider
networks in terms of bandwidth and reach.  

## Core Networks
Long-haul core networks also benefit from the CapEx and OpEx savings of moving
to Routed Optical Networking. Moving to a simpler IP enabled converged
infrastructure makes networks easier to manage and operate vs. networks with
complex underlying optical infrastructure.  The easiest place to start in the
journey is replacing external transponders with OpenZR+ QSFP-DD transceivers. At
400G connecting a 400G gray Ethernet port to a transponder with a 400G or 600G
line side is not cost or environmentally efficient. Cisco can assist in modeling 
your core network to determine the TCO of Routed Optical Networking compared to 
traditional approaches. 

## Metro Aggregation 
Tiered regional or metro networks connecting hub locations to larger aggregation 
site or datacenters can also benefit from Routed Optical Networking. Whether 
deployed in a hub and spoke topology or hop by hop IP ring, Routed Optical Networking 
satisfied provider's growth demands at a lower cost than traditional approaches.  

## Access 
Access deployments in a ring or point-to-point topology are ideal for Routed 
Optical Networking. Shorter distances over dark fiber may not require 
active optical equipment, and with up to 400G per span may provide the bandwidth
necessary for growth over a number of years without the use of additional 
multiplexers.  

## DCI and 3rd Party Location Interconnect 
In this use case, Routed Optical Networking simplifies deployments by eliminating 
active transponders, reducing power, space, and cabling requirements between end 
locations. 25.6Tbps of bandwidth is available over a single fiber using 64 400G 
wavelengths and simple optical amplifiers and multiplexers requiring no additional 
configuration after initial turn-up.   

# Routed Optical Networking Private Line Services 
Release 2.0 introduces Circuit Style Segment Routing TE Policies and Private Line 
Emulation hardware to enable traditional TDM-like private line services over the 
converged Segment Routing packet network. The following provides an overview of 
the hardware and software involved in supporting PL services. The figure below 
gives an overview of PLE service signaling and transport.  

![](http://xrdocs.io/design/images/ron-hld/ron-ple-overview.png)


## Circuit Style Segment Routing 
CS-SR provides the underlying TDM-like transport to support traditional private line 
Ethernet services without additional hardware and emulated bit-transparent services 
using Private Line Emulation hardware. 

### CS SR-TE paths characteristics 

* Co-routed Bidirectional - Meaning the paths between two client ports are symmetric
* Deterministic without ECMP - Meaning the path does not vary based on any load balancing criteria
* Persistent - Paths are routed on a hop by hop basis, so they are not subject to path changes induced by network changes 
* End-to-end path protection - Entire paths are switched from working to protect with the protect path in a hot standby state for fast transition   

SR CS-TE policies are built using link adjacency SIDs without protection to ensure the paths do not take a TI-LFA path during 
path failover and instead fail over to the pre-determined protect path.  

### CS SR-TE path liveness detection 
Paths can be configured with end to end liveness detection. Liveness detection uses TWAMP-lite probes which are looped at the far end to determine 
if the end to end path is up bi-directionally. If more than the set number of probes is missed (set by the multiplier) the path will be considered down. Once liveness detection is enabled probes will be sent on all candidate paths. Either the default liveness probe profile can be used or if 
you want to modify the default parameters a customized one can be created. 
### CS SR-TE path failover behavior 
CS SR-TE policies contain multiple candidate paths.  The highest preference candidate path is considered the working path, the second highest preference path is the protect path, and if a third lower preference path is configured would be a dynamic restoration path.  This provides 1:1+R 
protection for CS SR-TE policies.  The following below shows the configuration of a CS SR-TE Policy with a working, protect, and restoration path. T  

![](http://xrdocs.io/design/images/ron-hld/ron-srcs-paths.png)


<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  policy to-55a2-1
   color 1001 end-point ipv4 100.0.0.44
   path-protection
   !
   candidate-paths
    preference 25
     dynamic
       metric-type igp 
     !
    !
    preference 50
     explicit segment-list protect-forward-path
      reverse-path segment-list protect-reverse-path
     !
    !
    preference 100
     explicit segment-list working-forward-path
      reverse-path segment-list working-reverse-path
     !
    !
   !
   performance-measurement
    liveness-detection
     liveness-profile name liveness-check
</pre>
</div>

### CS SR-TE Policy operational details 
<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs55a2-1#show segment-routing traffic-eng policy color 1001
Sat Dec  3 13:32:38.356 PST

SR-TE policy database
---------------------

Color: 1001, End-point: 100.0.0.42
  Name: srte_c_1001_ep_100.0.0.42
  Status:
    adjmin: up  Operational: up for 2d09h (since Dec  1 04:08:12.648)
  Candidate-paths:
    Preference: 100 (configuration) (active)
      Name: to-100.0.0.42
      Requested BSID: dynamic
      PCC info:
        Symbolic name: cfg_to-100.0.0.42_discr_100
        PLSP-ID: 1
      Constraints:
        Protection Type: protected-preferred
        Maximum SID Depth: 12
      Explicit: segment-list forward-adj-path-working (valid)
        Reverse: segment-list reverse-adj-path-working
        Weight: 1, Metric Type: TE
          SID[0]: 15101 [adjacency-SID, 100.1.1.21 - 100.1.1.20]
          SID[1]: 15102
          SID[2]: 15103
          SID[3]: 15104
        Reverse path:
          SID[0]: 15001
          SID[1]: 15002
          SID[2]: 15003
          SID[3]: 15004
      Protection Information:
        Role: WORKING
        Path Lock: Timed
        Lock Duration: 300(s)
        State: ACTIVE
    Preference: 50 (configuration) (protect)
      Name: to-100.0.0.42
      Requested BSID: dynamic
      PCC info:
        Symbolic name: cfg_to-100.0.0.42_discr_50
        PLSP-ID: 2
      Constraints:
        Protection Type: protected-preferred
        Maximum SID Depth: 12
      Explicit: segment-list forward-adj-path-protect(valid)
        Reverse: segment-list reverse-adj-path-protect
        Weight: 1, Metric Type: TE
          SID[0]: 15119 [adjacency-SID, 100.1.42.1 - 100.1.42.0]
        Reverse path:
          SID[0]: 15191
      Protection Information:
        Role: PROTECT
        Path Lock: Timed
        Lock Duration: 300(s)
        State: STANDBY
  Attributes:
    Binding SID: 24017
    Forward Class: Not Configured
    Steering labeled-services disabled: no
    Steering BGP disabled: no
    IPv6 caps enable: yes
    Invalidation drop enabled: no
    Max Install Standby Candidate Paths: 0
</pre>
</div>

## Private Line Emulation Hardware 
Starting in IOS-XR 7.7.1 the NC55-OIP-02 Modular Port Adapter (MPA) is supported
on the NCS-55A2-MOD and NCS-57C3-MOD platforms. The NC55-OIP-02 has 8 SFP+ ports 

![](http://xrdocs.io/design/images/ron-hld/ron-hardware-ple-mpa-2.png){:height="100%" width="100%"}

Each port on the PLE MPA can be configured independently. The PLE MPA is responsible 
for receiving data frames from the native PLE client and packaging those into fixed 
frames for transport over the packet network.  

More information on the NC55-OIP-02 can be found in its datasheet located at 
<https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/network-con-5500-series-ds.pdf>. A full 
detailed to end to end configuration for PLE can be found in the Routed Optical Networking 2.0 Solution Guide found at <https://www.cisco.com/c/en/us/td/docs/optical/ron/2-0/solution/guide/b-ron-solution-20/m-ron.pdf>

### Supported Client Transceivers 

|Transport Type| Supported Transceivers| 
|--------|--------|
|Ethernet| SFP-10G-SR/LR/ER, GLC-LH/EX/ZX-SMD, 1G/10G CWDM |  
|OTN (OTU2e)| SFP-10G-LR-X, SFP-10G-ER-I, SFP-10G-Z |  
|SONET/SDH| ONS-SC+-10G-LR/ER/SR (OC-192/STM-64), ONS-SI-2G-L1/L2/S1 (OC-48/STM-16) | 
|Fiber Channel|DS-SFP-FCGE, DS-SFP-FC8G, DS-SFP-FC16G, DS-SFP-FC32G, 1/2/4/8G FC CWDM|

Note FC32G transceivers are supported in the even ports only and will disable the adjacent odd 
SFP+ port.  
{: .notice--warning}

## Private Line Emulation Pseudowire Signaling 
PLE utilizes IETF SAToP pseudowire encoding carried over dynamically signalled EVPN-VPWS 
circuits. Enhancements to the EVPN VPWS service type have been introduced to the IETF via 
<https://datatracker.ietf.org/doc/draft-schmutzer-bess-ple>. 

PLE services use Differential Clock Recovery (DCR) to ensure proper frame timing between 
the two PLE clients. In order to mmaintain accuracy of the clock each PLE endpoint router 
must have its frequency source traceable to a common primary reference clock (PRC).   

## Private Line Emulation EVPN-VPWS Configuration 
PLE services can be configured to utilize a CS SR-TE Policy or use dynamic MPLS 
protocols. The example belows shows the use of CS SR-TE Policy as transport for the 
PLE EVPN-VPWS service. Note the name of the sr-te policy in the preferred path command 
is the persistent generated name and not the name used in the CLI configuration. This can 
be determined using the "show segment-routing traffic-engineering policies" command.   

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 pw-class circuit-style-srte
  encapsulation mpls
   preferred-path sr-te policy srte_c_1001_ep_100.0.0.42
  !
 !
 xconnect group ple
  p2p ple-cs-1
   interface CEM0/0/2/1
   neighbor evpn evi 100 target 4201 source 4401
    pw-class circuit-style-srte
   !
  !
</pre>
</div>

## PLE Monitoring and Telemetry 

The following "show" command can be used to monitor the state of PLE ports and services. 

### Client Optics Port State 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs55a2-1#show controllers optics 0/0/2/1
Sat Dec  3 14:00:10.873 PST

 Controller State: Up

 Transport Admin State: In Service

 Laser State: On

 LED State: Not Applicable

 Optics Status

         Optics Type:  SFP+ 10G SR
         Wavelength = 850.00 nm

         Alarm Status:
         -------------
         Detected Alarms: None


         LOS/LOL/Fault Status:

         Laser Bias Current = 8.8 mA
         Actual TX Power = -2.60 dBm
         RX Power = -2.33 dBm

         Performance Monitoring: Disable

         THRESHOLD VALUES
         ----------------

         Parameter                 High Alarm  Low Alarm  High Warning  Low Warning
         ------------------------  ----------  ---------  ------------  -----------
         Rx Power Threshold(dBm)          2.0      -13.9          -1.0         -9.9
         Tx Power Threshold(dBm)          1.6      -11.3          -1.3         -7.3
         LBC Threshold(mA)              13.00       4.00         12.50         5.00
         Temp. Threshold(celsius)       75.00      -5.00         70.00         0.00
         Voltage Threshold(volt)         3.63       2.97          3.46         3.13

         Polarization parameters not supported by optics

         Temperature = 33.00 Celsius
         Voltage = 3.30 V

 Transceiver Vendor Details

         Form Factor            : SFP+
         Optics type            : SFP+ 10G SR
         Name                   : CISCO-FINISAR
         OUI Number             : 00.90.65
         Part Number            : FTLX8574D3BCL-CS
         Rev Number             : A
         Serial Number          : FNS23300J42
         PID                    : SFP-10G-SR
         VID                    : V03
         Date Code(yy/mm/dd)    : 19/07/25
</pre>
</div>


### PLE CEM Controller Stats 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs57c3-1#show controllers CEM 0/0/3/1
Sat Sep 24 11:34:22.533 PDT
Interface                          : CEM0/0/3/1
Admin state                        : Up
Oper  state                        : Up
Port bandwidth                     : 10312500 kbps
Dejitter buffer (cfg/oper/in-use)  : 0/813/3432 usec
Payload size (cfg/oper)            : 1280/1024 bytes
PDV (min/max/avg)                  : 980/2710/1845 usec
Dummy mode                         : last-frame
Dummy pattern                      : 0xaa
Idle pattern                       : 0xff
Signalling                         : No CAS
RTP                                : Enabled
Clock type                         : Differential
Detected Alarms                    : None

Statistics Info
---------------
Ingress packets          : 517617426962, Ingress packets drop     : 0
Egress packets           : 517277124278, Egress packets drop      : 0
Total error              : 0
        Missing packets          : 0, Malformed packets        : 0
        Jitter buffer underrun   : 0, Jitter buffer overrun    : 0
        Misorder drops           : 0
Reordered packets        : 0, Frames fragmented        : 0
Error seconds            : 0, Severely error seconds   : 0
Unavailable seconds      : 0, Failure counts           : 0

Generated L bits         : 0, Received  L bits         : 0
Generated R bits         : 339885178, Received  R bits         : 17

Endpoint Info
-------------
Passthrough     : No
</pre>
</div> 

### PLE CEM PM Statistics 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs57c3-1#show controllers CEM 0/0/3/1 pm current 30-sec cem
Sat Sep 24 11:37:02.374 PDT

CEM in the current interval [11:37:00 - 11:37:02 Sat Sep 24 2022]

CEM current bucket type : Valid
INGRESS-PKTS                : 2521591              Threshold : 0            TCA(enable) : NO
EGRESS-PKTS                 : 2521595              Threshold : 0            TCA(enable) : NO
INGRESS-PKTS-DROPPED        : 0                    Threshold : 0            TCA(enable) : NO
EGRESS-PKTS-DROPPED         : 0                    Threshold : 0            TCA(enable) : NO
INPUT-ERRORS                : 0                    Threshold : 0            TCA(enable) : NO
OUTPUT-ERRORS               : 0                    Threshold : 0            TCA(enable) : NO
MISSING-PKTS                : 0                    Threshold : 0            TCA(enable) : NO
PKTS-REORDER                : 0                    Threshold : 0            TCA(enable) : NO
JTR-BFR-UNDERRUNS           : 0                    Threshold : 0            TCA(enable) : NO
JTR-BFR-OVERRUNS            : 0                    Threshold : 0            TCA(enable) : NO
MIS-ORDER-DROPPED           : 0                    Threshold : 0            TCA(enable) : NO
MALFORMED-PKT               : 0                    Threshold : 0            TCA(enable) : NO
ES                          : 0                    Threshold : 0            TCA(enable) : NO
SES                         : 0                    Threshold : 0            TCA(enable) : NO
UAS                         : 0                    Threshold : 0            TCA(enable) : NO
FC                          : 0                    Threshold : 0            TCA(enable) : NO
TX-LBITS                    : 0                    Threshold : 0            TCA(enable) : NO
TX-RBITS                    : 0                    Threshold : 0            TCA(enable) : NO
RX-LBITS                    : 0                    Threshold : 0            TCA(enable) : NO
RX-RBITS                    : 0                    Threshold : 0            TCA(enable) : NO

</pre>
</div> 

### PLE Client PM Statistics  

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs57c3-1#show controllers EightGigFibreChanCtrlr0/0/3/4 pm current 30-sec fc
Sat Sep 24 11:51:55.168 PDT

FC in the current interval [11:51:30 - 11:51:55 Sat Sep 24 2022]

FC current bucket type : Valid
 IFIN-OCTETS                           : 16527749196          Threshold : 0            TCA(enable) : NO
 RX-PKT                                : 196758919            Threshold : 0            TCA(enable) : NO
 IFIN-ERRORS                           : 0                    Threshold : 0            TCA(enable) : NO
 RX-BAD-FCS                            : 0                    Threshold : 0            TCA(enable) : NO
 IFOUT-OCTETS                          : 0                    Threshold : 0            TCA(enable) : NO
 TX-PKT                                : 0                    Threshold : 0            TCA(enable) : NO
 TX-BAD-FCS                            : 0                    Threshold : 0            TCA(enable) : NO
 RX-FRAMES-TOO-LONG                    : 0                    Threshold : 0            TCA(enable) : NO
 RX-FRAMES-TRUNC                       : 0                    Threshold : 0            TCA(enable) : NO
 TX-FRAMES-TOO-LONG                    : 0                    Threshold : 0            TCA(enable) : NO
 TX-FRAMES-TRUNC                       : 0                    Threshold : 0            TCA(enable) : NO
</pre>
</div> 




# Routed Optical Networking Architecture Hardware 
All Routed Optical Networking solution routers are powered by Cisco IOS-XR.   
## Routed Optical Networking Validated Routers 

Below is a non-exhaustive snapshot of platforms validated for use with ZR and
OpenZR+ transceivers. Cisco supports Routed Optical Networking in the NCS 540,
NCS 5500/5700, ASR 9000, and Cisco 8000 router families. The breadth of coverage
enabled the solution across all areas of the network.   

![](http://xrdocs.io/design/images/ron-hld/ron-validated-hardware.png)
### Cisco 8000 Series  

The Cisco 8000 and its Silicone One NPU represents the next generation in
routers, unprecedented capacity at the lowest power consumption while supporting
a rich feature set applicable for a number of network roles. 

See more information on Cisco 8000 at <https://www.cisco.com/c/en/us/products/collateral/routers/8000-series-routers/datasheet-c78-742571.html>  

Specific information on ZR/ZR+ support can be found at <https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000/Interfaces/73x/configuration/guide/b-interfaces-config-guide-cisco8k-r73x/m-zr-zrp-cisco-8000.html> 

### Cisco 5700 Systems and NCS 5500 Line Cards 

The Cisco 5700 family of fixed and modular systems and line cards are flexible
enough to use at any location in the networks. The platform has seen widespread
use in peering, core, and aggregation networks.  

See more information on Cisco NCS 5500 and 5700 at <https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-736270.html> and 
<https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-744698.html>

Specific information on ZR/ZR+ support can be found at <https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/73x/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-73x/m-zr-zrp.html> 

### ASR 9000 Series 

The ASR 9000 is the most widely deployed SP router in the industry.  It has a
rich heritage dating back almost 20 years, but Cisco continues to innovate on
the ASR 9000 platform. The ASR 9000 series now supports 400G QSFP-DD on a
variety of line cards and the ASR 9903 2.4Tbps 3RU platform.  

See more information on Cisco ASR 9000 at <https://www.cisco.com/c/en/us/products/collateral/routers/asr-9000-series-aggregation-services-routers/data_sheet_c78-501767.html> 

Specific information on ZR/ZR+ support can be found at <https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r7-3/interfaces/configuration/guide/b-interfaces-hardware-component-cg-asr9000-73x/m-zr-zrp.html#Cisco_Concept.dita_59215d6f-1614-4633-a137-161ebe794673> 

### NCS 500 Series 

The 1Tbps N540-24QL16DD-SYS high density router brings QSFP-DD and Routed Optical Networking
ZR/OpenZR+ optics to a flexible access and aggregation platform. Using OpenZR+ optics it allows a 
migration path from 100G to 400G access rings or uplinks when used in an aggregation role.   

See more information on Cisco NCS 540 at <https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-500-series-routers/ncs-540-large-density-router-ds.html>
## Routed Optical Networking Optical Hardware

![](http://xrdocs.io/design/images/ron-hld/ron-optical-hardware.png)


### Network Convergence System 1010 

The NCS 1010 Open Optical Line System (O-OLS) is a next-generation DWDM platform 
available in fixed variants to satisfy building a modern flexible DWDM photonic 
network.  

The NCS 1010 Optical Line Terminal (OLT) uses a twin 33-port WSS architecture
allowing higher scale for either add/drop or express wavelengths. The OLT also
has two LC add/drop ports with integrated fixed gain EDFA to support the
add/drop of lower power optical signals. OLTs are available in models with or
without RAMAN amplification.  NCS 1010 Inline Amplifier nodes are available as
bi-directional EDFA, EDFA with RAMAN in one direction, or bi-directional RAMAN.
Each model of NCS 1010 is also available to support both C and L bands. In Routed Optical Networking 
2.0 ZR and ZR+ optics utilize the C band, but may be used on the same fiber with
L band signals using the NCS 1010 C+L combiner. 

The NCS 1010 utilizes IOS-XR, inheriting the advanced automation and telemetry
features similar to IOS-XR routers.   

**NCS 1010 OLT with RAMAN** 
![](http://xrdocs.io/design/images/ron-hld/ron-1010-olt-raman.png)

**NCS 1010 ILA with RAMAN** 
![](http://xrdocs.io/design/images/ron-hld/ron-1010-ila-raman.png)

The NCS1K-MD32-E/O-C 32-port 150Ghz spaced passive multiplexer is used with the NCS 1010, supporting the 75Ghz 
ZR/ZR+ signals and future higher baud rate signals. The MD-32 contains photodiodes to monitor RX power levels 
on each add/drop port.   

**NCS 1010 MD-32 Passive Filter** 
![](http://xrdocs.io/design/images/ron-hld/ron-1010-md32.png)

The NCS 1010 supports point to point and express DWDM optical topologies in Routed Optical Networking 
2.0. All NCS 1010 services in Routed Optical Networking are managed using Cisco Optical Network
Controller.  

See more information on the NCS 1010 series at <https://www.cisco.com/c/en/us/products/collateral/optical-networking/network-convergence-system-1000-series/network-conver-system-1010-ds.html>

### Network Convergence System 2000 

The NCS 2000 Optical Line System is a flexible platform supporting all modern
optical topologies and deployment use cases. Simple point to point to
multi-degree CDC deployments are all supported as part of Routed Optical
Networking.   

See more information on the NCS 2000 series at <https://www.cisco.com/c/en/us/products/optical-networking/network-convergence-system-2000-series/index.html>  
### Network Convergence System 1000 Multiplexer

The NCS1K-MD-64-C is a new fixed multiplexer designed specifically
for the 400G 75Ghz 400ZR and OpenZR+ wavelengths, allowing up to 25.6Tbps on a
single fiber. 
### Network Convergence System 1001 

The NCS 1001 is utiized in point to point network spans as an amplifier and
optionally protection switch. The NCS 1001 now has specific support for 75Ghz
spaced 400ZR and OpenZR+ wavelengths, with the ability to monitor incoming
wavelengths for power. The 1001 features the ability to determine the proper
amplifier gain setpoints based on the desired user power levels.  

See more information on the NCS 1001 at <https://www.cisco.com/c/en/us/products/collateral/optical-networking/network-convergence-system-1000-series/datasheet-c78-738782.html>

### NCS 2000 and NCS 1001 Hardware 
The picture below does not represent all available hardware on the NCS 2000, 
however does capture the modules typically used in Routed Optical Networking 
deployments.  

![](http://xrdocs.io/design/images/ron-hld/ron-optical-hardware.png)

# Routed Optical Networking Automation 
## Overview 
Routed Optical Networking by definition is a disaggregated optical solution,
creating efficiency by moving coherent endpoints in the router. The solution
requires a new way of managing the network, one which unifies the IP and Optical
layers, replacing the traditional siloed tools used in the past. Real
transformation in operations comes from unifying teams and workflows, rather
than trying to make an existing tool fit a role it was not originally designed
for. Cisco's standards based hierarchical SDN solution allows providers to
manage a multi-vendor Routed Optical Networking solution using standard
interfaces and YANG models.   


## IETF ACTN SDN Framework 
The IETF Action and Control of Traffic Engineered Networks group (ACTN) has
defined a hierarchical controller framework to allow vendors to plug components
into the framework as needed.  The lowest level controller, the Provisioning
Network Controller (PNC), is responsible for managing physical devices. These
controller expose their resources through standard models and interface to a
Hierarchical Controller (HCO), called a Multi-Domain Service Controller (MDSC)
in the ACTN framework.  

Note that while Cisco is adhering to the IETF framework proposed in 
[RFC8453](https://datatracker.ietf.org/doc/html/rfc8453/) , Cisco is supporting the most
widely supported industry standards for controller to controller communication
and service definition. In optical the de facto standard is Transport API from
the ONF for the management of optical line system networks and optical services.
In packet we are leveraging Openconfig device models where possible and IETF
models for packet topology (RFC8345) and xVPN services (L2NM and L3NM) 

![](http://xrdocs.io/design/images/ron-hld/actn-framework.png){:height="90%" width="90%"}
## Cisco's SDN Controller Automation Stack
Aligning to the ACTN framework, Cisco's automation stack includes a
multi-vendor IP domain controller (PNC), optical domain controller (PNC), and
multi-vendor hierarchical controller (HCO/MDSC).    

![](http://xrdocs.io/design/images/ron-hld/cisco-automation-stack.png)

## Cisco Open Automation 
Cisco believes not all providers consume automation in the same way, so we are
dedicated to make sure we have open interfaces at each layer of the network
stack. At the device level, we utilize standard NETCONF, gRPC, and gNMI
interfaces along with native, standard, and public consortium YANG models. There
is no aspect of a Cisco IOS-XR router today not covered by YANG models. At the
domain level we have Cisco's network controllers, which use the same standard
interfaces to communicate with devices and expose standards based NBIs. Our
multi-layer/multi-domain controller likewise uses the same standard interfaces.


![](http://xrdocs.io/design/images/ron-hld/open-automation.png){:height="90%" width="90%"}

## Crosswork Hierarchical Controller 
Responsible for Multi-Layer Automation is the Crosswork Hierarchical Controller. Crosswork Hierarchical Controller is responsible for the following network functions: 

* CW HCO unifies data from the IP and optical networks into a single network
  model. HCO utilizes industry standard IETF topology models for IP and TAPI for
  optical topology and service information. HCO can also leverage legacy EMS/NMS
  systems or device interrogation.   
* Responsible for managing multi-layer Routed Optical Networking links using a
  single UI.    
*  Providing assurance at the IP and optical layers in a single tool. The
   network model allows users to quickly correlate faults and identify at which
   layer faults have occurred.
* Additional HCO applications include the following  
  - Root Cause Analysis: Quickly correlate upper layer faults to an underlying cause.   
  - Layer Relations: Quickly identify the lower layer resources supporting higher layer network resource or all network resources reliant on a selected 
  lower layer network resource.  
  - Network Inventory: View IP and optical node hardware inventory along with with network resources such as logical links, optical services, and traffic engineering tunnels  
  - Network History: View state changes across all network resources at any point in time  
  - Performance: View historical link utilization  

Please see the following resources for more information on Crosswork HCO. <https://www.cisco.com/c/en/us/products/collateral/cloud-systems-management/crosswork-network-automation/solution-overview-c22-744695.html> 

![](http://xrdocs.io/design/images/ron-hld/hco-multi-layer-circuit-2.png){:height="100%" width="100%"}

## Crosswork Network Controller 
Crosswork Network Controller is a multi-vendor IP domain controller. Crosswork
Network Controller is responsible for the following IP network functions. 

* Collecting Ethernet, IP, RSVP-TE, and SR network information for internal
  applications and exposing northbound via IETF RFC 8345 topology models 
* Collecting traffic information from the network for use with CNC's traffic
  optimization application, Crosswork Optimization Engine 
* Perform provisioning of SR-TE, RSVP-TE, L2VPN, and L3VPN using standard
  industry models (IETF TEAS-TE, L2NM, L3NM) via UI or northbound API 
* Visualization and assurance of SR-TE, RSVP-TE, and xVPN services 
* Use additional Crosswork applications to perform telemetry collection/alerting,
  zero-touch provisioning, and automated and assurance network changes  

More information on Crosswork and Crosswork Network Controller can be found at <https://www.cisco.com/c/en/us/products/collateral/cloud-systems-management/crosswork-network-automation/datasheet-c78-743456.html>

## Cisco Optical Network Controller 
Cisco Optical Network Controller (Cisco ONC) is responsible for managing Cisco 
optical line systems and circuit services. Cisco ONC exposes a ONF TAPI northbound 
interface, the de facto industry standard for optical network management. Cisco ONC 
runs as an application on the same Crosswork Infrastructure as CNC.  

More information on Cisco ONC can be found at <https://www.cisco.com/c/en/us/support/optical-networking/optical-network-controller/series.html> 

## Cisco Network Services Orchestrator and Routed Optical Networking ML Core Function Pack 
Cisco NSO is the industry standard for service orchestration and device
configuration management. The RON-ML CFP can be used to fully configure an IP
link between routers utilizing 400ZR/OpenZR+ optics over a Cisco optical line
system using Cisco ONC. This includes IP addressing and adding links to an
existing Ethernet LAG. The CFP can also support optical-only provisioning on the
router to fit into existing optical provisioning workflows.  

# Routed Optical Networking Service Management

## Supported Provisioning Methods
We support multiple ways to provision Routed Optical Networking services based 
on existing provider workflows.  

* [Unified IP and Optical using Crosswork Hierarchical Controller](#crosswork-hco-ui-provisioning)
* [Unified IP and Optical using Cisco NSO Routed Optical Networking Multi-Layer Function Pack](#nso-ron-ml-cfp-provisioning) 
* [ZR/ZR+ Optics using IOS-XR CLI](#ios-xr-cli-configuration) 
* [Model-driven ZR/ZR+ Optics configuration using Netconf or gNMI](#ios-xr-netconf-configuration)
* [OpenConig ZR/ZR+ Optics configuration using Netconf or gNMI](#ios-xr-oc-configuration)

## OpenZR+ and 400ZR Properties 
### ZR/ZR+ Supported Frequencies 

The frequency on Cisco ZR/ZR+ transceivers may be set between 191.275Thz and
196.125Thz in increments of 6.25Ghz, supporting flex spectrum applications. To
maximize the available C-Band spectrum, these are the recommended 64
75Ghz-spaced channels, also aligning to the NCS1K-MD-64-C fixed channel add/drop
multiplexer. 


|     |    |    |      |    |    |    |   |
|-----|----|----|------|----|----|----|----|
|196.100|196.025|195.950|195.875|195.800|195.725|195.650|195.575|
|195.500|195.425|195.350|195.275|195.200|195.125|195.050|194.975| 
|194.900|194.825|194.75|194.675|194.600|194.525|194.450|194.375|
|194.300|194.225|194.150|194.075|194.000|193.925|193.850|193.775| 
|193.700|193.625|193.550|193.475|193.400|193.325|193.250|193.175|
|193.100|193.025|192.950|192.875|192.800|192.725|192.650|192.575| 
|192.500|192.425|192.350|192.275|192.200|192.125|192.050|191.975|
|191.900|191.825|191.750|191.675|191.600|191.525|191.450|191.375| 

### Supported Line Side Rate and Modulation  

OIF 400ZR transceivers support 400G only per the OIF specification. OpenZR+
transceivers can support 100G, 200G, 300G, or 400G line side rate. See router
platform documentation for supported rates. The modulation is determined by the
line side rate.  400G will utilize 16QAM, 300G 8QAM, and 200G/100G rates will
utilize QPSK.  

## Crosswork Hierarchical Controller UI Provisioning 

End-to-End IP+Optical provisioning can be done using Crosswork Hierarchical Controller's GUI IP Link
provisioning. Those familiar with traditional GUI EMS/NMS systems for service
management will have a very familiar experience. Crosswork Hierarchical Controller provisioning will provision
both the router optics as well as the underlying optical network to support the
ZR/ZR+ wavelength.   

### Inter-Layer Link Definition 

End to end provisioning requires first defining the Inter-Layer link between the
router ZR/ZR+ optics and the optical line system add/drop ports. This is done
using a GUI based NMC (Network Media Channel) Cross-Link application in Crosswork HCO.
The below screenshot shows defined NMC cross-links.   

![](http://xrdocs.io/design/images/ron-hld/ron-hco-nmc-xconnects.png)
### IP Link Provisioning 
Once the inter-layer links are created, the user can then proceed in
provisioning an end to end circuit.  The provisioning UI takes as input the two
router endpoints, the associated ZR/ZR+ ports, and the IP addressing or bundle
membership of the link. The optical line system provisioning is abstracted from
the user, simplifying the end to end workflow. The frequency and power is
automatically derived by Cisco Optical Network Controller based on the add/drop
port and returned as a parameter to be used in router optics provisioning.  

![](http://xrdocs.io/design/images/ron-hld/ron-hco-ip-link-provisioning-2.png)

### Operational Discovery 
The Crosswork Hierarchical Controller provisioning process also performs a discovery phase to ensure the
service is operational before considering the provisioning complete. If
operational discovery fails, the end to end service will be rolled back.  
## NSO RON-ML CFP Provisioning
Providers familiar with using Cisco Network Service Orchestrator have an option
to utilize NSO to perform IP+Optical provisioning of Routed Optical Networking
services. Cisco has created the Routed Optical Network Multi-Layer Core Function
Pack, RON-ML CFP to perform end to end provisioning of services. The
aforementioned Crosswork HCO provisioning utilizes the RON-ML CFP to perform end device
provisioning.  

Please see the Cisco Routed Optical Networking RON-ML CFP documentation located at 

### Routed Optical Networking Inter-Layer Links 
Similar to the use case with CW HCO provisioning, before end to end provisioning
can be performed, inter-layer links must be provisioned between the optical
ZR/ZR+ port and the optical line system add/drop port.  This is done using the
"inter-layer-link" NSO service. The optical end point can be defined as either a
TAPI SIP or by the TAPI equipment inventory identifier. Inter-layer links are 
not required for router-only provisioning.   

### RON-ML End to End Service 
The RON-ML service is responsible for end to end IP+optical provisioning. RON-ML
supports full end to end provisioning, router-only provisioning, or optical-only
provisioning where only the router ZR/ZR+ configuration is performed. The
frequency and transmit power can be manually defined or optionally provided by
Cisco ONC when end to end provisioning is performed.   

### RON-ML API Provisioning
Use the following URL for NSO provisioning: ```http://<nso host>/restconf/data``` 

**Inter-Layer Link Service** 

```json
{
  "data": {
    "cisco-ron-cfp:ron": {
      "inter-layer-link": [
        {
          "end-point-device": "ron-8201-1",
          "line-port": "0/0/0/20",
          "ols-domain": {
            "network-element": "ron-ols-1",
            "optical-add-drop": "1/2008/1/13,14",
            "optical-controller": "onc-real-new"
          }
        }
      ]
    }
  }
}
```

**Provisioning ZR+ optics and adding interface to Bundle-Ether 100 interface**  

```json
{
    "cisco-ron-cfp:ron": {
      "ron-ml": [
        {
          "name": "E2E_Bundle_ZRP_ONC57_2",
          "mode": "transponder",
          "bandwidth": "400",
          "circuit-id": "E2E Bundle ONC-57 S9|chan11 - S10|chan11",
          "grid-type": "100mhz-grid",
          "ols-domain": {
            "service-state": "UNLOCKED"
          },
          "end-point": [
            {
              "end-point-device": "ron-8201-1",
              "terminal-device-optical": {
                "line-port": "0/0/0/11",
                "transmit-power": -100
              },
              "ols-domain": {
                "end-point-state": "UNLOCKED"
              },
              "terminal-device-packet": {
                "bundle": [
                  {
                    "id": 100
                  }
                ],
                "interface": [
                  {
                    "index": 0,
                    "membership": {
                      "bundle-id": 100,
                      "mode": "active"
                    }
                  }
                ]
              }
            },
            {
              "end-point-device": "ron-8201-2",
              "terminal-device-optical": {
                "line-port": "0/0/0/11",
                "transmit-power": -100
              },
              "ols-domain": {
                "end-point-state": "UNLOCKED"
              },
              "terminal-device-packet": {
                "bundle": [
                  {
                    "id": 100
                  }
                ],
                "interface": [
                  {
                    "index": 0,
                    "membership": {
                      "bundle-id": 100,
                      "mode": "active"
                    }
                  }
                ]
              }
            }
          ]
        }
      ]
    }
  }
```
## IOS-XR CLI Configuration
Configuring the router portion of the Routed Optical Networking link is very
simple.  All optical configuration related to the ZR/ZR+ optics configuration is
located under the optics controller relevent to the faceplate port. Default
configuration the optics will be in an up/up state using a frequency of
193.10Thz.

The basic configuration with a specific frequency of 195.65 Thz is located below,   
the only required component is the bolded channel frequency setting.  

**ZR/ZR+ Optics Configuration** 

<div class="highlighter-rouge">
<pre class="highlight">
controller Optics0/0/0/20
 transmit-power -100
 <b>dwdm-carrier 100MHz-grid frequency 1956500</b>
 logging events link-status
</pre>
</div>

## Model-Driven Configuration using IOS-XR Native Models using NETCONF or gNMI  
All configuration performed in IOS-XR today can also be done using NETCONF/YANG. The following payload exhibits the models 
and configuration used to perform router optics provisioning. This is a more complete example showing the FEC, power, and 
frequency configuration. .  

**Note in Release 2.0 using IOS-XR 7.7.1 the newer IOS-XR Unified Models are utilized for provisioning**

```xml
<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<controllers xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-interface-cfg">
    <controller>
        <controller-name>Optics0/0/0/0</controller-name>
        <transmit-power xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-cont-optics-cfg">-115</transmit-power>
        <fec xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-cont-optics-cfg">OFEC</fec>
        <dwdm-carrier xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-cont-optics-cfg">
          <grid-100mhz>
            <frequency>1913625</frequency>
          </grid-100mhz>
        </dwdm-carrier>
        <dac-rate xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-dac-rate-cfg">1x1.25</dac-rate>
      </controller>
</controllers> 

</data>
```

## Model-Driven Configuration using OpenConfig Models
Starting on Release 2.0 all IOS-XR 7.7.1+ routers supporting ZR/ZR+ optics can be 
configured using OpenConfig models. Provisioning utilizes the openconfig-terminal-device 
model and its extensions to the openconfig-platform model to support DWDM configuration 
parameters. 

Below is an example of an OpenConfig payload to configure ZR/ZR+ optics port 0/0/0/20 with a 
300G trunk rate with frequency 195.20 THz.  

Please visit the blog at <https://xrdocs.io/design/blogs/zr-openconfig-mgmt> for in depth information 
about configuring and monitoring ZR/ZR+ optics using OpenConfig models.  

```xml
<config>
        <terminal-device xmlns="http://openconfig.net/yang/terminal-device">
            <logical-channels>
                <channel>
                    <index>100</index>
                    <config>
                        <index>200</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
                <channel>
                    <index>101</index>
                    <config>
                        <index>101</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
                <channel>
                    <index>102</index>
                    <config>
                        <index>102</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
                <channel>
                    <index>200</index>
                    <config>
                        <index>200</index>
                        <admin-state>ENABLED</admin-state>
                        <description>Coherent Logical Channel</description>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_OTN</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>300</allocation>
                                <assignment-type>OPTICAL_CHANNEL</assignment-type>
                                <description>Coherent to optical assignment</description>
                                <optical-channel>0/0-OpticalChannel0/0/0/20</optical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
            </logical-channels>
        </terminal-device>
        <components xmlns="http://openconfig.net/yang/platform">
            <component>
                <name>0/0-OpticalChannel0/0/0/20</name>
                <optical-channel xmlns="http://openconfig.net/yang/terminal-device">
                    <config>
                        <operational-mode>5007</operational-mode>
                        <frequency>195200000</frequency>
                    </config>
                </optical-channel>
            </component>
        </components>
    </config>
```

# Routed Optical Networking Assurance 

## Crosswork Hierarchical Controller
### Multi-Layer Path Trace 
Using topology and service data from both the IP and Optical network CW HCO can
display the full service from IP services layer to the physical fiber.  Below is
an example of the "waterfall" trace view from the OTS (Fiber) layer to the
Segment Routing TE layer across all layers. CW HCO identifies specific Routed
Optical Networking links using ZR/ZR+ optics as seen by the ZRC (ZR Channel) and
ZRM (ZR Media) layers from the 400ZR specification.  

![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-trace-2.png){:height="100%" width="100%"}

When faults occur at a specific layer, faults will be highlighted in red,
quickly identifying the layer a fault has occurred. In this case we can see the
fault has occurred at an optical layer, but is not a fiber fault. Having the
ability to pinpoint the fault layer even within a specific domain is a powerful
way to quickly determine the root cause of the fault.  

![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-trace-fault-2.png){:height="100%" width="100%"}

### Routed Optical Networking Link Assurance 
The Link Assurance application allows users to view a network link and all of 
its dependent layers. This includes Routed Optical Networking multi-layer 
services. In addition to viewing layer information, fault and telemetry information 
is also available by simply selecting a link or port.   

#### ZRM Layer TX/RX Power 
![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-2.png){:height="100%" width="100%"}

#### ZRC Layer BER and Q-Factor / Q-Margin  
![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-3.png){:height="100%" width="100%"}

Optionally the user can see graphs of collected telemetry data to quickly identify trends or changes in specific operational 
data. Graphs of collected performance data is accessed using the "Performance" tab when a link or port is selected.   

#### OTS Layer RX/TX Power Graph
![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-graph-2.png){:height="100%" width="100%"}

#### Event Monitoring 
Crosswork HCO records any transition of a network resource between up/down operational states. This is reflected in the 
Link Assurance tool under the "Events" tab. 

![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-events.png){:height="100%" width="100%"}
## IOS-XR CLI Monitoring of ZR400/OpenZR+ Optics

### Optics Controller 
The optics controller represents the physical layer of the optics.  In the case
of ZR/ZR+ optics this includes the frequency information, RX/TX power, OSNR, and
other associated physical layer information.   

```
RP/0/RP0/CPU0:ron-8201-1#show controllers optics 0/0/0/20
Thu Jun  3 15:34:44.098 PDT

 Controller State: Up
 Transport Admin State: In Service
 Laser State: On
 LED State: Green
 FEC State: FEC ENABLED

 Optics Status

         Optics Type:  QSFPDD 400G ZR
         DWDM carrier Info: C BAND, MSA ITU Channel=10, Frequency=195.65THz,
         Wavelength=1532.290nm

         Alarm Status:
         -------------
         Detected Alarms: None

         LOS/LOL/Fault Status:

         Alarm Statistics:

         -------------
         HIGH-RX-PWR = 0            LOW-RX-PWR = 0
         HIGH-TX-PWR = 0            LOW-TX-PWR = 4
         HIGH-LBC = 0               HIGH-DGD = 1
         OOR-CD = 0                 OSNR = 10
         WVL-OOL = 0                MEA  = 0
         IMPROPER-REM = 0
         TX-POWER-PROV-MISMATCH = 0
         Actual TX Power = -7.17 dBm
         RX Power = -9.83 dBm
         RX Signal Power = -9.18 dBm
         Frequency Offset = 9 MHz
         Baud Rate =  59.8437500000 GBd
         Modulation Type: 16QAM
         Chromatic Dispersion 6 ps/nm
         Configured CD-MIN -2400 ps/nm  CD-MAX 2400 ps/nm
         Second Order Polarization Mode Dispersion = 34.00 ps^2
         Optical Signal to Noise Ratio = 35.50 dB
         Polarization Dependent Loss = 1.20 dB
         Polarization Change Rate = 0.00 rad/s
         Differential Group Delay = 2.00 ps

```
Performance Measurement Data 

```
RP/0/RP0/CPU0:ron-8201-1#show controllers optics 0/0/0/20 pm current 30-sec optics 1
Thu Jun  3 15:39:40.428 PDT

Optics in the current interval [15:39:30 - 15:39:40 Thu Jun 3 2021]

Optics current bucket type : Valid
             MIN       AVG       MAX      Operational      Configured      TCA   Operational      Configured     TCA
                                          Threshold(min)   Threshold(min) (min) Threshold(max)   Threshold(max) (max)
LBC[% ]      : 0.0       0.0       0.0      0.0               NA              NO   100.0            NA              NO
OPT[dBm]     : -7.17     -7.17     -7.17    -15.09            NA              NO   0.00             NA              NO
OPR[dBm]     : -9.86     -9.86     -9.85    -30.00            NA              NO   8.00             NA              NO
CD[ps/nm]    : -489      -488      -488     -80000            NA              NO   80000            NA              NO
DGD[ps ]     : 1.00      1.50      2.00     0.00              NA              NO   80.00            NA              NO
SOPMD[ps^2]  : 28.00     38.80     49.00    0.00              NA              NO   2000.00          NA              NO
OSNR[dB]     : 34.90     35.12     35.40    0.00              NA              NO   40.00            NA              NO
PDL[dB]      : 0.70      0.71      0.80     0.00              NA              NO   7.00             NA              NO
PCR[rad/s]   : 0.00      0.00      0.00     0.00              NA              NO   2500000.00       NA              NO
RX_SIG[dBm]  : -9.23     -9.22     -9.21    -30.00            NA              NO   1.00             NA              NO
FREQ_OFF[Mhz]: -2        -1        4        -3600             NA              NO   3600             NA              NO
SNR[dB]      : 16.80     16.99     17.20    7.00              NA              NO   100.00           NA              NO
```

### Coherent DSP Controller
The coherent DSP controller represents the framing layer of the optics. It 
includes Bit Error Rate, Q-Factor, and Q-Margin information. 

```
RP/0/RP0/CPU0:ron-8201-1#show controllers coherentDSP 0/0/0/20
Sat Dec  4 17:24:38.245 PST

Port                                            : CoherentDSP 0/0/0/20
Controller State                                : Up
Inherited Secondary State                       : Normal
Configured Secondary State                      : Normal
Derived State                                   : In Service
Loopback mode                                   : None
BER Thresholds                                  : SF = 1.0E-5  SD = 1.0E-7
Performance Monitoring                          : Enable
Bandwidth                                       : 400.0Gb/s

Alarm Information:
LOS = 10        LOF = 0 LOM = 0
OOF = 0 OOM = 0 AIS = 0
IAE = 0 BIAE = 0        SF_BER = 0
SD_BER = 0      BDI = 0 TIM = 0
FECMISMATCH = 0 FEC-UNC = 0     FLEXO_GIDM = 0
FLEXO-MM = 0    FLEXO-LOM = 3   FLEXO-RDI = 0
FLEXO-LOF = 5
Detected Alarms                                 : None

Bit Error Rate Information
PREFEC  BER                                     : 1.7E-03
POSTFEC BER                                     : 0.0E+00
Q-Factor                                        : 9.30 dB
Q-Margin                                        : 2.10dB


FEC mode                                        : C_FEC
```

Performance Measurement Data 
```
RP/0/RP0/CPU0:ron-8201-1#show controllers coherentDSP 0/0/0/20 pm current 30-sec fec
Thu Jun  3 15:42:28.510 PDT

g709 FEC in the current interval [15:42:00 - 15:42:28 Thu Jun 3 2021]

FEC current bucket type : Valid
    EC-BITS   : 20221314973             Threshold : 83203400000            TCA(enable)  : YES
    UC-WORDS  : 0                       Threshold : 5                      TCA(enable)  : YES

                                      MIN       AVG        MAX      Threshold      TCA     Threshold     TCA
                                                                       (min)     (enable)    (max)     (enable)
PreFEC BER                     :   1.5E-03   1.5E-03   1.6E-03         0E-15        NO       0E-15        NO
PostFEC BER                    :     0E-15     0E-15     0E-15         0E-15        NO       0E-15        NO
Q[dB]                          :      9.40      9.40      9.40          0.00        NO        0.00        NO
Q_Margin[dB]                   :      2.20      2.20      2.20          0.00        NO        0.00        NO
```

## EPNM Monitoring of Routed Optical Networking 
Evolved Programmable Network Manager, or EPNM, can also be used to monitor router ZR/ZR+ performance measurement data 
and display device level alarms when faults occur. EPNM stores PM and alarm data for historical analysis.  

### EPNM Chassis View of DCO Transceivers 
The following shows a chassis view of a Cisco 8201 router. The default view is to show all active alarms on the 
device and its components. Clicking on a specific component will give information on the component and narrow 
the scope of alarms and data.   

#### Chassis View 

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-chassis-view.png){:height="100%" width="100%"}

#### Interface/Port View 

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-interface-view.png){:height="100%" width="100%"}

### EPNM DCO Performance Measurement 
EPNM continuously monitors and stores PM data for DCO optics for important KPIs such as TX/RX power, BER, and Q values.  The screenshots 
below highlight monitoring. While EPNM stores historical data, clicking on a speciic KPI will enable realtime monitoring by polling for 
data every 20 seconds.  

#### DCO Physical Layer PM KPIs  

The following shows common physical layer KPIs such as OSNR and RX/TX power.  This is exposed by monitoring the Optics layer of the interface.   
DCO.    

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-optics-phy-pm.png){:height="100%" width="100%"}

The following shows common framing layer KPIs such as number of corrected words per interval and  (BIEC) Bit Error Rate. This is exposed by monitoring the CoherentDSP 
layer of the interface.   

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-optics-dsp-pm.png){:height="100%" width="100%"}

# Cisco IOS-XR Model-Driven Telemetry for Routed Optical Networking Monitoring 


All operational data on IOS-XR routers and optical line systems can be monitored using streaming telemetry 
based on YANG models. Routed Optical Networking is no different, so a wealth of 
information can be streamed from the routers in intervals as low as 5s.  

## ZR/ZR+ DCO Telemetry 

The following represents a list of validated sensor paths useful for monitoring
the DCO optics in IOS-XR and the data fields available within these
sensor paths.  Note PM fields also support 15m and 24h paths in addition to the 
30s paths shown in the table below.  

| Sensor Path | Fields |
|-------------|--------| 
|Cisco-IOS-XR-controller-optics-oper:optics-oper/optics-ports/optics-port/optics-info | alarm-detected, baud-rate, dwdm-carrier-frequency, controller-state, laser-state, optical-signal-to-noise-ratio, temperature, voltage |
|Cisco-IOS-XR-controller-optics-oper:optics-oper/optics-ports/optics-port/optics-lanes/optics-lane|receive-power, receive-signal-power, transmit-power|
|Cisco-IOS-XR-controller-otu-oper:otu/controllers/controller/info|bandwidth, ec-value, post-fec-ber, pre-fec-ber, qfactor, qmargin, uc | 
|Cisco-IOS-XR-pmengine-oper:performance-management/optics/optics-ports/optics-port/optics-current/optics-second30/optics-second30-optics/optics-second30-optic|dd__average, dgd__average, opr__average, opt__average, osnr__average, pcr__average, pmd__average, rx-sig-pow__average, snr__average, sopmd__average| 
|Cisco-IOS-XR-pmengine-oper:performance-management/otu/otu-ports/otu-port/otu-current/otu-second30/otu-second30fecs/otu-second30fec|ec-bits__data, post-fec-ber__average, pre-fec-ber__average, q__average, qmargin__average, uc-words__data |  


## NCS 1010 Optical Line System Monitoring  

The following represents a list of validated sensor paths useful for monitoring
the different optical resources on the NCS 1010 OLS. The OTS controller represents 
the lowest layer port interconnecting optical elements. The NCS 1010 supports 
per-channel monitoring, exposed as the OTS-OCH  

| Sensor Path | Fields |
|-------------|--------| 
|Cisco-IOS-XR-controller-ots-oper:ots-oper/ots-ports/ots-port/ots-info | total-tx-power, total-rx-power, transmit-signal-power, receive-signal-power, agress-ampi-gain, ingress-ampli-gain, controller-state |
|Cisco-IOS-XR-controller-ots-och-oper:ots-och-oper/ots-och-ports/ots-och-port/ots-och-info | total-tx-power, total-rx-power, transport-admin-state, line-channel, add-drop-channel | 
|Cisco-IOS-XR-controller-oms-oper | rx-power, tx-power, controller-state, led-state | 
|Cisco-IOS-XR-controller-och-oper:och-oper/och-ports/och-port/och-info |channel-frequency, channel-wavelength, controller-state, rx-power, tx-power, channel-width, led-state| 
|Cisco-IOS-XR-pmengine-oper:performance-management/optics/optics-ports | opr, opt, opr-s, opt-s| 
|Cisco-IOS-XR-olc-oper:olc/span-loss-ctrlr-tables/span-loss-ctrlr-table | neighbor-rid, rx-span-loss, tx-span-loss, name | 


## Open-source  Monitoring 
Cisco model-driven telemetry along with the open source collector Telegraf and the open source dashboard software 
Grafana can be used to quickly build powerful dashboards to monitor ZR/ZR+ and NCS 1010 OLS performance.  

![](http://xrdocs.io/design/images/ron-hld/ron-telemetry-grafana.png)

![](http://xrdocs.io/design/images/ron-hld/ron-telemetry-grafana-1010.png)

<br>

---
# Additional Resources
## Cisco Routed Optical Networking 2.0 Solution Guide

<https://www.cisco.com/content/en/us/td/docs/optical/ron/2-0/solution/guide/b-ron-solution-20.html>
## Cisco Routed Optical Networking Home 
* <https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking.html> 
## Cisco Routed Optical Networking Tech Field Day 
* Solution Overview: <https://techfieldday.com/video/build-your-network-with-cisco-routed-optical-networking-solution/> 
* Automation Demo: <https://techfieldday.com/video/cisco-routed-optical-networking-solution-demo/> 

## Cisco Champion Podcasts 
* Cisco Routed Optical Networking Solution for the Next Decade <https://smarturl.it/CCRS8E24> 
* Simplify Network Operations with Crosswork Hierarchical Controller: <https://smarturl.it/CCRS8E48 >

# Appendix A

## Acronyms 

|     |     | 
|-----|-----| 
| DWDM | Dense Waveform Division Multiplexing | 
| OADM | Optical Add Drop Multiplexer |
| FOADM | Fixed Optical Add Drop Multiplexer |
| ROADM | Reconfigurable Optical Add Drop Multiplexer |
| DCO | Digital Coherent Optics | 
| FEC | Forward Error Correction | 
| OSNR| Optical Signal to Noise Ratio | 
| BER | Bit Error Rate | 



## DWDM Network Hardware Overview  

![](http://xrdocs.io/design/images/ron-hld/ron-design-optical-components.png)
### Optical Transmitters and Receivers 
Optical transmitters provide the source signals carried across the DWDM network.
They convert digital electrical signals into a photonic light stream on a
specific wavelength. Optical receivers detect pulses of light and and convert
signals back to electrical signals. In Routed Optical Networking, digital 
coherent QSFP-DD OpenZR+ and 400ZR transceivers in routers are used as optical 
transmitters and receivers.  

### Multiplexers/Demultiplexers  
Multiplexers take multiple wavelengths on separate fibers and combine them into
a single fiber. The output of a multiplexer is a composite signal.
Demultiplexers take composite signals that compatible multiplexers generate and
separate the individual wavelengths into individual fibers.

### Optical Amplifiers 
Optical amplifiers amplify an optical signal. Optical amplifiers increase the
total power of the optical signal to enable the signal transmission across
longer distances. Without amplifiers, the signal attenuation over longer
distances makes it impossible to coherently receive signals. We use different
types of optical amplifiers in optical networks. For example: preamplifiers,
booster amplifiers, inline amplifiers, and optical line amplifiers.

### Optical add/drop multiplexers (OADMs)
OADMs are devices capable of adding one or more DWDM channels into or dropping
them from a fiber carrying multiple channels. 

### Reconfigurable optical add/drop multiplexers (ROADMs)
ROADMs are programmable versions of OADMs. With ROADMs, you can change the
wavelengths that are added or dropped. ROADMs make optical networks flexible and
easily modifiable.


