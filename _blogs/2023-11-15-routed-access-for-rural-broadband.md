---
published: true
date: '2023-11-15 12:24 -0700'
title: Routed Access for Rural Broadband
author: Shelly Cadora
excerpt: A Cisco routed access design for rural broadband
position: top
tags:
  - cisco
---

{% include toc %}
# High-Level Design

Routed Access for Rural Broadband introduces best-practice network design for small operators looking to deploy residential broadband services.

## Key Drivers

### New Sources of Funding
Substantial and ongoing federal investment in US broadband has created a historic opportunity to bring high speed access to the unserved and underserved across the country, primarily in rural areas.  The following federal programs are some of the major funding initiatives that have brought $100 billion to solve the challenges of rural broadband access:
- The NTIA Broadband Equity, Access, and Deployment (BEAD) Program, provides $42.45 billion to expand high-speed internet access by funding planning, infrastructure deployment and adoption programs in all 50 states
- FCC Rural Digital Opportunity Fund (RDOF) Phase 1 & 2 will disburse up to $20.4 billion over 10 years in two phase to bring fixed broadband and voice service to millions of unserved homes and small businesses in rural America.
- The American Rescue Plan (ARPA)  has already spent or committed more than $25 billion to invest in affordable high-speed internet and connectivity.
- The NTIA Tribal Broadband Connectivity Program (TBCP) is a $3 billion program, from President Biden’s Bipartisan Infrastructure Law and the Consolidated Appropriations Act, to support Tribal governments bringing high-speed Internet to Tribal lands

With an estimated $40 billion in additional private equity funds being deployed in tandem with federal grants, 57 million additional homes will be served with fiber to the home (FTTH) in the next 5 years.

### New Providers
While traditional, large-scale service providers will be able to expand their services with these new sources of funds, the goals and funding model of federal grants also open the door for new types of providers to enter the market.  Smaller scale and non-traditional providers (such as utilities, tribal organizations, non-profit consortiums and public sector organizations) are taking advantage of federal grants to bring broadband to their constituents.  For example, the BEAD program grants money to individual states that then distribute the funds through state-level broadband offices.  These offices can give preference to smaller, community-based, and non-traditional providers that strive to serve the mission of providing broadband access to local underserved populations.

### Focus on Fiber
Although many types of technology can enable high-speed broadband access, a clear preference for fiber to the home has emerged.  The majority of BEAD funding ($41.6 billion) is to be used to deploy last-mile fiber-optic networks.   Only “extremely high-cost” areas will be permitted to deploy non-fiber technologies under the terms of the program. 

There are several reasons for the focus on fiber for rural broadband deployments. Network design for residential broadband services is constrained by the availability of power for electronic components that make up the network. A passive optical network (PON) running over fiber does not need to be powered by electricity (even splitters function without added power) which means that higher distances can be achieved between a customer’s home or business and the central office or hub. For rural deployments with low population densities and long distances between subscribers, PON is an obvious choice.

In addition to being able to transmit high-bandwidth data over long distances with minimal electro-magnetic interference, fiber deployments can last for decades with the same fiber supporting higher and higher data rates as technologies evolves over time. One such transition is happening now.  For more than a decade, Gigabit-capable Pon (GPON) has been the gold standard for fiber deployments, providing 2.5Gpbs downstream and 1.25 Gbps upstream.  As the demand for bandwidth has grown, providers have started to shift to 10-Gigabit symmetric PON (XGS-PON).  Because GPON and XGS-PON run on different wavelengths, both can run on the same fiber.  Providers can easily migrate from GPON to XGS-PON by swapping out the equipment on either end of the fiber.

With all the advantages of fiber and 10 Gigabits of symmetric bandwidth, XGS-PON is a compelling choice for rural broadband over the next 5 years.

## Value Proposition
[Converged SDN Transport](https://xrdocs.io/design/blogs/latest-converged-sdn-transport-hld) is the gold standard for modern service provider networking architecture, delivering a high-scale, programmable network capable of delivering all services (Residential, Business, 4G/5G Mobile Backhaul, Video, IoT).   While many of the high-level design principles of Converged SDN Transport can be extended to rural broadband, many deployments will benefit from some modifications.  In particular,  rural broadband deployments may emphasize:
- **Lower densities**: rural deployments will have fewer people over greater distances than urban or suburban deployments
- **Lower scale**: unlike urban or suburban deployments, rural deployments may have much smaller total subscriber counts.  The RBB Design is targeted at deployments from 5,000 to 100,000 subscribers.
- **More flexibility**:  rural deployments may require the flexibility to build as they go, starting simple and adding functionality as requirements evolve and funding allows.  The RBB design starts with basic connectivity with the option to add redundancy and subscriber-specific features later on.
- **Residential PON**: while large providers may need to support a wide variety of services, rural broadband will often focus on providing PON to residential subscribers with a simple high-speed data (with or without voice) service.  The RBB design focuses on providing an access network design for PON networks.
- **Simplicity**: smaller, non-traditional providers need simple, easy-to-manage networks.  The RBB design streamlines the Converged SDN Transport design, eliminating complex features where possible.  These features can be added later if the scale and service requirements of the network change.


## Solution Overview
Routed Access for Rural Broadband is made of the following main building blocks:

- PON-Agnostic 
- IOS-XR as a common operating system proven in Service Provider Networks
- Routed access with transport based on Segment Routing
- Redundancy and traffic isolation provided by Layer 2 (EVPN) and Layer 3 VPN services based on BGP
- BNG for optional per-subscriber traffic management

### PON Agnostic
One of the first decisions a new rural broadband provider makes is how to deploy PON.  The optical line terminal (OLT) is the starting point of the optical network and serves as the demarkation point of the access network. Many PON vendors offer OLTs in a variety of form-factors, from massive 48-port PON shelves to 4-port temperature-hardened remote OLTs, to pluggable SFPs that provide full OLT functionality when plugged into an ethernet port.

The sparser densities and longer distances in rural broadband will typically favor smaller form factor OLTs.  In any case, the Routed Access for Rural Broadband design supports any vendor's OLT that connects to the access network via common ethernet technologies.

### Routed Access
Access domains have traditionally been built using flat Layer 2 native ethernet technologies.  But traditions sometimes persist even when the reasons for them no longer exist.  In the past, many people defaulted to Layer 2 networks because switching was less expensive and Layer 2 seemed simpler than IP networks.  But big shifts in the economics of routing silicon have made routers more affordable and innovations in routing protocols have made it simpler to deploy.  

There are many benefits to bringing IP to the access network:

- Arbitrary Topologies: To prevent broadcast storms and duplicate packets, traditional Layer2 topologies must not contain loops.  G.8032 is a common loop prevention technique that requires an underlying ring topology.  In other topologies, Spanning Tree Protocol is required to block redundant ports.  Because IP networks use TTL to prevent loops, arbitrary topologies can be supported without disabling redundant paths.
- Reconvergence: In Layer 2 networks, MAC-learning occurs in the data plane.  Changes in topology (i.e. link failures) require flushing all MAC tables and subsequent flooding of traffic.  This can caused extended reconvergence times. L3 networks, on the other hand, can reconverge very quickly
- Troubleshooting: It can be difficult to know the actual path in complex Layer 2 networks.
- Efficiency:  Because redundant paths must be blocked in Layer 2 topologies, there is no notion of Equal Cost Multi-Pathing (ECMP).  IP networks support equal-cost multi-path and active-active redundancy, so that all links can be used at the same time. 
- Resilience: Layer 2 networks achieve multi-homing only with active-standby MC-LAG, which is vulnerable to traffic storms.  Layer 3 networks can achieve multi-homing with EVPN-based LAG which supports active-active redundancy.


### Hardware Components

#### NCS 540 Series
The [NCS 540 family](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-540-series-routers/index.html) of routers are powerful and versatile access routers capable of aggregating uplinks from XGS-PON devices.  With a wide range of speeds (10G/25G/40G/100G) and port densities, the NCS 540 series can support many variations of access topologies.

#### NCS 5500 / 5700 Fixed Chassis
The [NCS 5500 / 5700 fixed series](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-5500-5700-platform-ar-wp.html) devices are scalable, low-power, and cost-optimized 100G routing platforms for aggregation and high-scale access roles in the Routed Access for Rural Broadband design. The platform family offers industry-leading density of 1/10/25/40/50/100/400G ports with efficient forwarding performance, low jitter, and the lowest power consumption per gigabits/sec at a very cost-effective price point.

#### ASR 9900
The [ASR 9900](https://www.cisco.com/c/en/us/products/routers/asr-9000-series-aggregation-services-routers/index.html#~models) is the router family of choice for edge services. The Routed Access for Rural Broadband utilizes the ASR 9000 in a PE function role, performing Pseudowire headend termination for per-subscriber policy with BNG.

### Design Components

#### Topology Options
Unlike the tidy racks of data center networks, the physical design of broadband networks tends to be messy, dominated by the geography and population density of the area to be served.  Homes connect back to OLT located at a point-of-presence, central office or outside plant in a point-to-multipoint topology.  An XGS-PON OLT can serve homes in a 20 to 100 km radius.  A small, remote OLT (hardened for outside conditions) with 4 ports and a 64-way split could serve up to 256 homes.  A 16-port PON shelf with the same 64-way split might serve 1024 homes within a 40 km radius.  For denser, less rural areas, larger PON shelves can be stacked to serve tens or hundreds of thousands of homes within the supported radius.

Once the user’s traffic reaches the OLT, it gets handed off to the access network which takes that traffic to the Internet (or other service).  The design of access networks is also determined by geography and the availability of fiber.   Two typical designs are ring and point-to-point.

In a routed ring access topology, routers are connected in a ring.  Because traffic can flow in either direction, the ring topology provides an automatic backup path if a link or router fails with sub 50-millisecond convergence times. 

In the following example topology, generic XGS-PON OLTs with 10G uplinks connect to a 100G routed access ring of NCS 540s.  The 55A1-24Hs, with 24 100G ports, can aggregate multiple 100G NCS 540 rings and, if needed, provide uplinks for 100G OLT shelves.

![RBB_Ring_Topo.jpg]({{site.baseurl}}/images/RBB_Ring_Topo.jpg)

Depending on the density and distribution of subscribers, the uplink capabilities of the your PON solution, the availability requirements, and the acceptable oversubscription rate, different platforms in the NCS540 and NCS5500/5700 families can be used to construct the access ring and aggregation nodes.

In a point to point access architecture, the access routers can be single or dual-homed to the central office.  Dual-homing enables redundancy should either of the uplinks fail, as well as providing extra capacity during normal operation.  

![RBB_Hub_And_Spoke.jpg]({{site.baseurl}}/images/RBB_Hub_And_Spoke.jpg)

Regardless of how the routers connect back to the Internet (ring or point-to-point), each router serves as an aggregation point for individual OLTs, OLT rings or OLT trees.  Depending on the availability requirements, those OLTs can be single or dual-homed to the routers.  

#### Routing and Forwarding

Once the user traffic reaches the access router, it enters the layer 3 domain of the network. The ethernet frame header is discarded and the packet is routed over the IP network to the final destination.

The routing protocol or IGP (Interior Gateway Protocol) allows routers in the same domain to exchange routing information so that packets can be routed according to the destination IP address. This design uses IS-IS as the IGP protocol.  Because rural broadband designs are typically small enough to run a single IGP domain, the focus of the design is on intra-domain forwarding.  For networks that exceed the scale limits of a single domain, refer to the [Inter-Domain Operation](https://xrdocs.io/design/blogs/latest-converged-sdn-transport-hld#inter-domain-operation) guidelines of the Converged SDN Transport design.

To improve forwarding performance and enable advanced features, this design uses Multiprotocol Label Switching (MPLS) for the data plane.  MPLS is a "layer 2.5" technology that enables routers to transfer packets through the network using labels. Instead of needing expensive route table lookups at every hop, MPLS traffic can be efficiently forwarded through label-switching.  Historically, a separate protocol like LDP was required to advertise labels throughout the network.  Today, Segment Routing eliminates the need to configure and maintain LDP to advertise labels.  

Segment Routing reduces the amount of protocols needed in a Service Provider Network. Simple extensions to traditional IGP protocols like ISIS provide full Intra-Domain Routing and Forwarding Information over a label switched infrastructure, along with High Availability (HA) and Fast Re-Route (FRR) capabilities that enable fast recovery should a link or node fail.

Segment Routing introduces the idea of a Prefix Segment Identifier or Prefix-SID.  A prefix-SID identifies the router and must be unique for every router in the IGP Domain. Prefix-SID is statically allocated by the network operator in the IGP configuration process. The Prefix-SID is advertised by the IGP protocol which eliminates the need to use LDP or RSVP protocol to exchange MPLS labels.  For these reasons, Segment Routing is a foundational technology in this design.

#### Fast Re-Route using TI-LFA
Segment-Routing embeds a simple Fast Re-Route (FRR) mechanism known as Topology Independent Loop Free Alternate (TI-LFA).

TI-LFA provides sub 50ms convergence for link and node protection. TI-LFA is completely stateless and does not require any additional signaling mechanism as each node in the IGP Domain calculates a primary and a backup path automatically and independently based on the IGP topology. After the TI-LFA feature is enabled, no further care is expected from the network operator to ensure fast network recovery from failures. This is in stark contrast with traditional MPLS-FRR, which requires RSVP and RSVP-TE and therefore adds complexity in the transport design.

#### MPLS VPN Services
One of the advantages of an IP/MPLS network is that it enables the deployment of VPN services.  Many people associate L3VPN and L2VPN with expensive business services.  But even small providers can benefit from judicious use of MPLS VPNs in residential deployments.

In the simplest design, subscriber traffic arrives at the access device, receives an IP address via DHCP and gets access to the Internet via the global routing table. Many large, modern networks are being built to isolate the global routing table from the underlying infrastructure. In this case, the Internet global table is carried as an L3VPN service, leaving the infrastructure layer protected from both the global Internet.  

The other use case for MPLS VPN services is when Broadband Network Gateway (BNG) is deployed for per-subscriber traffic management.  Because BNG requires more sophisticated treatment of the user traffic and, hence, more expensive forwarding hardware, the BNG device is often deployed in a centralized location to take advantage of economies of scale.  Access devices can be configured to backhaul Layer 2 subscriber traffic to the BNG device using EVPN pseudowires.

#### IPv6 vs IPv4
To address the inevitable exhaustion of IPv4 addresses, the IETF standardized IPv6 decades ago. Even today, however, IPv6 adoption remains incomplete.  Some content on the internet is only available via IPv4; some network infrastructure features are still optimized for IPv4.

For rural broadband networks, this design recommends using IPv4 for the underlay transport infrastructure.  SR-MPLS is very mature on IPv4 networks and private IP addressing can be used for the routers in the network, thus avoiding address exhaustion concerns for these devices.  At the same time, the design supports "dual-stack" (IPv4 and IPv6) deployments to ensure that the network can transition to IPv6-based transport should the need arise.  The underlay transport architecture for IPv6 network is called "SRv6."  SRv6 enables next-generation IPv6 based networks to support complex user and infrastructure services at very high scale.  Rural broadband networks typically do not have the scale or complexity to make SRv6 as compelling.  But if, for any reason, IPv4 and SR-MPLS are not suitable for your deployment, reference the [Cisco Converged SDN Transport SRv6 Transport High Level Design](https://xrdocs.io/design/blogs/latest-converged-sdn-transport-srv6) for details on SRv6-based transport.

For end customers and customer premise equipment, addressing schemes can be IPv4-only, IPv6-only or dual-stack.  Deploying IPv4 addresses will require either 1) acquiring globally routable IPv4 addresses; or 2) translating private IPv4 addresses using a Carrier Grade Nat (CGN) solution.  Each option has drawbacks.  If you have not already been allocated a block, globally routable IPv4 addresses are expensive.  In 2022, IPv4 addresses were selling for around $60 each.  In addition to being expensive, purchased addresses can come with baggage.  Depending on the actions of the previous owner, the address may have acquired a bad reputation that resulted in it being blacklisted from certain networks. This can be a difficult problem to detect, troubleshoot and clean up. If you chose a CGN solution instead, you will need far fewer globally routed addresses but you will have to purchase, configure and maintain another device in the dataplane.

Over [40% of the world's top 1000 websites](https://pulse.internetsociety.org/technologies) are accessible via native IPv6, which means that IPv6-only clients can quickly access that content.  However, for the rest of the content on the network, some translation or tunneling mechanism must be implemented to allow IPv6-only end users to connect to IPv4-only content which adds complexity to the network design.

Given the state of the internet today, a dual-stack deployment with CGN may represent the best tradeoff for rural broadband deployments.  Native IPv6 traffic (which includes popular, bandwidth heavy services like Netflix and YouTube) can go natively between clients and content providers.  IPv6 traffic will bypass the CGN function, reducing the load on that device.  IPv4-only content will continue to leverage CGN until the day that IPv6 has been fully adopted world-wide.

#### QOS
Quality of Service (QoS) refers to the ability of a network to provide better service to various types of network traffic.  To achieve an end to end QoS objective (i.e. between the subscriber and the Internet), it’s necessary to consider the different mechanisms in the PON network and routed access network.

QoS can be applied in two directions: upstream (from the subscriber to the Internet) or downstream (from the internet to the subscriber).  In PON networks, upstream traffic is controlled by the OLT, which implements a flow-control mechanism that lends itself to a simpler implementation of QoS.  The ITU-T G.983.4 recommendation coupled with other QoS mechanisms enable the OLT to effectively schedule, shape and prioritize upstream traffic.

While upstream traffic management in the OLT is well-defined and largely consistent across vendors, downstream traffic management can often be more effectively carried out deeper in the access and core network.  

The platforms in the design have sophisticated and rich QoS feature sets that are widely deployed in large-scale, converged, multi-service provider networks.  Even on the smallest platforms, QoS has many options and can be endlessly customized.  Rural broadband networks, however, tend to be relatively simple and well-supplied with bandwidth.  Therefore, the goal of this design is to get as much value as possible from the simplest deployment of QoS. Simple policies with basic marking and egress queuing are often sufficient and require less monitoring and maintenance than more complex policies.

QoS policies applied to network interfaces act on the aggregate subscriber traffic on that interface. This, combined with OLT QoS capabilities, may be sufficient for simple deployments of sparsely populated areas.  Other providers may need to leverage a BNG solution for finer-grained QoS policies.  

## Use Cases

### High Speed Residential Data Services
The primary use case for rural broadband deployments leveraging federal grants such as BEAD is a high speed (100M minimum) data service.  Other offerings, such as telephone and video, may be offered over a funded network that meets the unicast data requirements but those services are subject to additional regulation.

Depending on the configuration of the PON network, unicast subscriber traffic will arrive at the access router from the OLT in one of two ways: VLAN per service (N:1 VLAN) or VLAN per subscriber (1:1 VLAN).  In the N:1 VLAN model, a service (e.g. high-speed data) is represented by a VLAN.  Many (“N”) subscribers access that service via a single (“1”) VLAN.  Other services, like voice and video, use other VLANs.  This is the simplest solution to deploy and maintain.

In the 1:1 VLAN model, each subscriber is assigned their own VLAN.  Typically, the 1:1 VLAN model is deployed using  Q-in-Q double tagging (C-VLAN + S-VLAN) to overcome the scale limitations of 4096 VLANs. However, managing thousands of customer VLANs can become administratively heavy, especially for provisioning, troubleshooting and documentation.

The routed access design for rural broadband can support either VLAN model.  

### High Availability for PON
Whether you’re deploying a single OLT shelf, a tree of OLTs or a ring of OLTs, the connection from the OLT to the access router represents a potential point of failure.  To protect against the failure of a single port on either the router or the OLT, multiple links are commonly bundled together for a redundant connection.  To protect against a router failure, the OLT can be dual-homed to two different routers using EVPN.  EVPN supports “all-active” redundant links to two or more routers.  The  OLT believes that it is connected to a single device using a normal bundle interface.  

The routed access design for rural broadband supports a single, non-redundant uplink, a bundled interface to the same router, and a bundled interface to two redundant routers with EVPN.

### Per-subscriber Policies with BNG
To manage individual subscriber sessions, providers can implement Broadband Network Gateway (BNG).  BNG acts as a gatekeeper, verifying that only approved subscribers can get access to the network and implementing per-subscriber policies (such as QoS or access-control lists) to improve security and the quality of subscriber experience.

BNG manages all aspects of subscriber access including:
- Authentication, authorization and accounting of subscriber sessions
- Address assignment
- Security
- Policy management
- Quality of Service (QoS)

Cisco’s implementation of BNG is a mature, feature-rich technology that supports many use cases of varying degrees of complexity.  The RBB design focuses on a simple IP over Ethernet (IPoE) deployment that authenticates subscribers, assigns addresses using DHCP, and applies per-subscriber policy.  

# Implementation Details

## Targets

  - Hardware:
    
      - NCS540 and NCS5500 as Access Router 
      - ASR9903 as Provider Edge (PE) node
      - NCS5500 as Aggregation Node

  - Software:
    
      - IOS-XR 7.9.2 on NCS540, NCS 5500
      - IOS-XR 7.8.1 on ASR9000 

  - Key technologies
    
      - Transport: End-To-End Segment-Routing
      - Services: BGP-based L2 and L3 Virtual Private Network services
        (EVPN and L3VPN)


## Testbed Overview

![RBB_CVD_testbed.jpg]({{site.baseurl}}/images/RBB_CVD_testbed.jpg)

_Figure 1: Routed Access For Rural Broadband High Level Topology_


## Devices

**Access Routers**

  - Cisco N540-24Z8Q2C-M (IOS-XR) – PE101, PE102, PE104, PE105
  
  - Cisco N540X-16Z4G8Q2C-A (IOS-XR) - PE103

**Area Border Routers (ABRs) and Provider Edge Routers:**

  - Cisco ASR9000 (IOS-XR) – PE3, PE4

## Key Resources to Allocate  
- IP Addressing 
  - IPv4 address plan
  - IPv6 address plan  
- IS-IS unique instance identifiers


## Role-Based Configuration
    
### Transport IOS-XR – All IOS-XR nodes
        
#### IGP Protocol (ISIS) and Segment Routing MPLS configuration

**Router isis configuration**

```
key chain ISIS-KEY
 key 1
 accept-lifetime 00:00:00 january 01 2018 infinite
 key-string password 00071A150754
 send-lifetime 00:00:00 january 01 2018 infinite
 cryptographic-algorithm HMAC-MD5

```

All Access Routers are part of one IGP domain (ISIS ACCESS).

```
router isis ISIS-ACCESS
 set-overload-bit on-startup 360
 is-type level-2-only
 net 49.0001.0102.0000.0065.00
 nsr
 nsf cisco
 log adjacency changes
 lsp-gen-interval maximum-wait 5000 initial-wait 50 secondary-wait 200
 lsp-refresh-interval 65000
 max-lsp-lifetime 65535
 lsp-password keychain ISIS-KEY
 lsp-password keychain ISIS-KEY level 1
 address-family ipv4 unicast
  metric-style wide
  spf-interval maximum-wait 5000 initial-wait 50 secondary-wait 200
  segment-routing mpls
  spf prefix-priority critical tag 5000
  spf prefix-priority high tag 1000
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid index 65
  !
 !
```

**TI-LFA FRR configuration**

```
 interface HundredGigE0/0/1/0
  point-to-point
  hello-password keychain ISIS-KEY
  address-family ipv4 unicast
   fast-reroute per-prefix
   fast-reroute per-prefix ti-lfa
   metric 100
  !
 ! 
!

interface Loopback0
 ipv4 address 101.0.2.65 255.255.255.255
!
```

**MPLS Interface configuration**

```
interface HundredGigE0/0/1/0
  mtu 9216
 ipv4 address 10.23.65.1 255.255.255.254
 ipv4 unreachables disable
 ipv6 address 2405:23:65::/127
 load-interval 30
 dampening
!
```

**QoS Policy-Map Configuration**

The following represent simple policies to classify on ingress (core facing interfaces only) and queue on egress.  For simplicity, only two classes and queues are used: high-priority (dscp 46) and default (everything else).

```
class-map match-any match-ef
 description High priority, EF
 match dscp 46
 end-class-map
!
class-map match-any match-traffic-class-1
 description "Match highest priority traffic-class 1"
 match traffic-class 1
 end-class-map
!
policy-map rbb-egress-queuing
 class match-traffic-class-1
  priority level 1
  queue-limit 500 us
 !
 class class-default
  queue-limit 250 ms
 !
 end-policy-map
!
policy-map core-egress-queuing
 class match-traffic-class-1
  priority level 1
  queue-limit 500 us
 !
 class class-default
  queue-limit 250 ms
 !
 end-policy-map
!
policy-map rbb-ingress-classifier
 class match-ef
  set traffic-class 1
  set qos-group 1
 !
 class class-default
  set traffic-class 0
  set qos-group 0
 !
 end-policy-map
!
policy-map core-ingress-classifier
 class match-ef
  set traffic-class 1
 !
 class class-default
 !
 end-policy-map
```
 
**Core-facing Interface Qos Config**

```
interface HundredGigE0/0/1/0
 service-policy input core-ingress-classifier
 service-policy output core-egress-queuing
 ```

**OLT-facing Interface QoS Config**

```
 interface Bundle-Ether1
 service-policy input rbb-ingress-classifier
 service-policy output rbb-egress-queuing
 ```
 
## BGP – Access
Enable BGP when your deployment needs any of the following:

- Multi-homed, redundant connectivity to the OLT using EVPN-enabled LAG
- Isolation of user traffic into a non-default VRF
- Per-subscriber policy with BNG

Very small networks can use fully meshed BGP peers.  Otherwise, a route reflector is recommended.
    
### IOS-XR configuration

```
router bgp 100
 nsr
 bgp router-id 101.0.2.65
 bgp graceful-restart
 bgp graceful-restart graceful-reset
 ibgp policy out enforce-modifications
 address-family vpnv4 unicast
 !
 address-family vpnv6 unicast
 !
 address-family l2vpn evpn
 !
 neighbor-group SvRR
  remote-as 100
  update-source Loopback0
  address-family vpnv4 unicast
  !
  address-family vpnv6 unicast
  !
  address-family l2vpn evpn
  !
 !
 neighbor 100.0.1.201
  use neighbor-group SvRR
 !
```


# Services
    
## EVPN-enabled LAG for Redundant OLT Connections
![EVPN_Enabled_LAG.jpg]({{site.baseurl}}/images/EVPN_Enabled_LAG.jpg)


When applied to both pe101 and pe102, the following configuration creates a bridge group for VLAN 300 on both routers which share a default anycast gateway represented by the BVI interface.  The BVI interface can be assigned a DHCP relay profile, included in the global routing table or put in a separate VRF.  

VRF (Virtual Routing and Forwarding) is a feature that allows the router to maintain multiple, independent routing tables.  Creating a separate VRF for user and Internet traffic isolates the infrastructure layer and protects it from the Internet. Read more about the uses and benefits of VRFs in network design in the [Peering Fabric Design]{https://xrdocs.io/design/blogs/2018-10-01-peering-fabric-hld/#internet-and-peering-in-a-vrf}. {: .notice--info}

Note that this configuration can be applied even if you only have a single link to a single access router.  By configuring a bundle interface from the beginning, you can easily add another to a second router without changing the configuration of the first router.

### Access Router Service Provisioning:

**PON-facing Access Interface Configuration**

```
interface TenGigE0/0/0/0
 bundle id 1 mode active
 !
interface Bundle-Ether1
 lacp system mac 0000.0000.0001
 lacp system priority 1
!
interface Bundle-Ether1.300 l2transport
 encapsulation dot1q 300
 rewrite ingress tag pop 1 symmetric
 !
interface BVI1
 host-routing
 ipv4 address 10.10.65.1 255.255.255.0
 local-proxy-arp
 ipv6 address 3ffe:501:ffff:101::8/64
 mac-address 0.0.3
```

**EVPN Ethernet-Segment Configuration**

```
evpn
 interface Bundle-Ether1
  ethernet-segment
   identifier type 0 00.00.00.00.00.00.00.00.01
```

**L2VPN Bridge-Group Configuration**

```
l2vpn
 bridge group RBB_BRIDGE_GROUP
  bridge-domain RBB_BRIDGE_DOMAIN
   interface Bundle-Ether1.300
   !
   routed interface BVI1
   !
   evi 3
```
  
## Per-subscriber policies with BNG

The following configuration is used for a deployment of IPoE subscriber sessions. The configuration of some external elements such as the RADIUS authentication server are outside the scope of this document. For more information about the subscriber features and policy (including QoS, security ACLs, Lawful Intercept and more), see the [Broadband Network Gateway Configuration Guide for Cisco ASR 9000 Series Routers](https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r7-8/bng/configuration/guide/b-bng-cg-asr9000-78x.html).

![EVPN PW 3.jpg]({{site.baseurl}}/images/EVPN PW 3.jpg)

When applied to both pe101 and pe102, the following configuration creates a bridge group for VLAN 311 on both routers that backhauls Layer 2 traffic to PE3 which can authenticate the subscriber, apply per-subscriber policies, assign IP addresses and route traffic to the internet.

### Access Router Service Provisioning:

**Interface Configuration**

```
interface TenGigE0/0/0/0
 bundle id 1 mode active
 !
interface Bundle-Ether1
 lacp system mac 0000.0000.0001
 lacp system priority 1
 
interface Bundle-Ether1.311 l2transport
 encapsulation dot1q 311
 rewrite ingress tag pop 1 symmetric
 ```
 
 **EVPN Ethernet-Segment Configuration**

```
evpn
 interface Bundle-Ether1
  ethernet-segment
   identifier type 0 00.00.00.00.00.00.00.00.01
```

**L2VPN Pseudowire Configuration**

 ```
 l2vpn
 xconnect group RBB-PW-BNG
  p2p RBB-PW-BNG1
   interface Bundle-Ether1.311
   neighbor evpn evi 10101 service 101
 ```

### Service Router Provisioning:

The pseudowire initiated on pe101 and pe102 is terminated on the BNG router, pe3, on a pseudowire headend endpoint (PW-Ether).  When the first dhcp request from the subscriber arrives on the pseudowire headend on pe3, pe3 will authenticate the source mac-address and apply the per-subscriber policy.

**Interface configuration**

```
interface PW-Ether2101
 mtu 1518
 ipv4 address 182.168.101.1 255.255.255.0
 ipv6 address 2000:101:1::1:1/64
 ipv6 enable
 service-policy type control subscriber RBB_IPoE_PWHE
 attach generic-interface-list PWHE
 ipsubscriber ipv4 l2-connected
  initiator dhcp
 !
 ipsubscriber ipv6 l2-connected
  initiator dhcp
!
generic-interface-list PWHE
 interface HundredGigE0/0/0/15
```

**DHCP configuration**

```
dhcp ipv4
 profile RBB_GROUP proxy
  helper-address vrf default 10.0.65.2 giaddr 101.0.0.3
  relay information option allow-untrusted
 !
 interface PW-Ether2101 proxy profile RBB_GROUP
 ```
 
**L2VPN Pseudowire configuration**
```
l2vpn
 xconnect group rbb-pwhe-bng
  p2p rbb-pwhe-bng1
   interface PW-Ether2101
   neighbor evpn evi 10101 service 101
```

**Policy-Map and dynamic policy for BNG**

```
policy-map type control subscriber RBB_IPoE_PWHE
 event session-start match-first
  class type control subscriber DHCP46 do-until-failure
   1 authorize aaa list RBB_AUTH1_LIST identifier source-address-mac password cisco
   10 activate dynamic-template PWHE_PBNG1
  !
 !
 end-policy-map
 
 dynamic-template
 type ipsubscriber PWHE_PBNG1
  ipv4 verify unicast source reachable-via rx
  ipv4 unnumbered Loopback1
  ipv4 unreachables disable
  ipv6 enable
 !
!
```

## Summary: Routed Access for Rural Broadband

The Routed Access design brings the benefits of Converged SDN Transport to rural broadband networks with simple, flexible, smaller-scale approach to residential broadband. No matter what PON solution you deploy, Routed Access reduces the complexity associated with Layer 2 networks by introducing the many benefits of IP and Segment Routing as close to the PON network as possible.
