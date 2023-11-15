---
published: false
date: '2023-11-15 12:24 -0700'
title: Routed Access for Rural Broadband
author: Shelly Cadora
excerpt: A Cisco routed access design for rural broadband
---

## Key Drivers

### New Sources of Funding
Substantial and ongoing federal investment in US broadband has created a historic opportunity to bring high speed access to the unserved and underserved across the country, primarily in rural areas.  The following federal programs are some of the major funding initiatives that have brought $100 billion to solve the challenges of rural broadband access:
- The NTIA Broadband Equity, Access, and Deployment (BEAD) Program, provides $42.45 billion to expand high-speed internet access by funding planning, infrastructure deployment and adoption programs in all 50 states
- FCC Rural Digital Opportunity Fund (RDOF) Phase 1 & 2 will disburse up to $20.4 billion over 10 years in two phase to bring fixed broadband and voice service to millions of unserved homes and small businesses in rural America.
- The American Rescue Plan (ARPA)  has already spent or committed more than $25 billion to invest in affordable high-speed internet and connectivity.
- The NTIA Tribal Broadband Connectivity Program (TBCP) is a $3 billion program, from President Biden’s Bipartisan Infrastructure Law and the Consolidated Appropriations Act, to support Tribal governments bringing high-speed Internet to Tribal lands

With an estimated $40 in additional private equity funds being deployed in tandem with federal grants, 57 million additional homes will be served with fiber to the home (FTTH) in the next 5 years.

### New Providers
While traditional, large-scale service providers will be able to expand their services with these new sources of funds, the goals and funding model of federal grants also open the door for new types of providers to enter the market.  Smaller scale and non-traditional providers (such as utilities, tribal organizations, non-profit consortiums and public sector organizations) are taking advantage of federal grants to bring broadband to their constituents.  For example, the BEAD program grants money to individual states that then distribute the funds through state-level broadband offices.  These offices can give preference to smaller, community-based, and non-traditional providers that strive to serve the mission of providing broadband access to local underserved populations.

### Focus on Fiber
Although many types of technology can enable high-speed broadband access, a clear preference for fiber to the home has emerged.  The majority of BEAD funding ($41.6 billion) is to be used to deploy last-mile fiber-optic networks.   Only “extremely high-cost” areas will be permitted to deploy non-fiber technologies under the terms of the program. 

There are several reasons for the focus on fiber for rural broadband deployments. Network design for residential broadband services is constrained by the availability of power for electronic components that make up the network.  So called “active” electronic components need power (and sometimes cooling) to function.  A passive optical network (PON) running over fiber does not need to be powered by electricity (even splitters function without added power).  Because PON is composed of passive optical components that don’t need power, higher distances can be achieved between a customer’s home or business and the central office or hub. For rural deployments with low population densities and long distances between subscribers, PON is an obvious choice.

In addition to being able to transmit high-bandwidth data over long distances with minimal electro-magnetic interference, fiber deployments can last for decades with the same fiber supporting higher and higher data rates as technologies evolves over time. One such transition is happening now.  For more than a decade, Gigabit-capable Pon (GPON) has been the gold standard for fiber deployments, providing 2.5Gpbs downstream and 1.25 Gbps upstream.  As the demand for bandwidth has grown, providers have started to shift to 10-Gigabit symmetric PON (XGS-PON).  Because GPON and XGS-PON run on different wavelengths, both can run on the same fiber.  Providers can easily migrate from GPON to XGS-PON by swapping out the equipment on either end of the fiber.

With all the advantages of fiber and 10 Gigabits of symmetric bandwidth, XGS-PON is a compelling choice for rural broadband over the next 5 years.

## Value Proposition

[Converged SDN Transport](https://xrdocs.io/design/blogs/latest-converged-sdn-transport-hld) is the gold standard for modern service provider networking architecture, delivering a high-scale, programmable network capable of delivering all services (Residential, Business, 4G/5G Mobile Backhaul, Video, IoT).   While many of the high-level design principles of Converged SDN Transport can be extended to rural broadband, some deployments will benefit from some modifications.  In particular,  rural broadband deployments may emphasize:
- **Lower densities**: rural deployments will have fewer people over greater distances than urban or suburban deployments
- **Lower scale**: unlike urban or suburban deployments, rural deployments may have much smaller total subscriber counts.  The RBB Design is targeted at deployments from 5,000 to 100,000 subscribers.
- **More flexibility**:  rural deployments may require the flexibility to build as they go, starting simple and adding functionality as requirements evolve and funding allows.  The RBB design starts with basic connectivity with the option to add redundancy and subscriber-specific features later on.
- **Residential PON**: while large providers may need to support a wide variety of services, rural broadband will often focus on providing PON to residential subscribers with a simple high-speed data (with or without voice) service.  The RBB design focuses on providing an access network design for PON networks.
- **Simplicity**: smaller, non-traditional providers need simple, easy-to-manage networks.  The RBB design streamlines the Converged SDN Transport design, eliminating complex features where possible.  These features can be added later if the scale and service requirements of the network change.

Routed Access for Rural Broadband PON introduces best-practice network design for small operators looking to deploy residential broadband services.

## Solution Overview
The Routed Access for Rural Broadband is made of the following main building blocks:
- IOS-XR as a common Operating System proven in Service Provider Networks
- Routed access with transport based on Segment Routing
- Redundancy and traffic isolation provided by Layer 2 (EVPN) and Layer 3 VPN services based on BGP
- BNG for optional per-subscriber traffic management


### Routed Access
Access domains have traditionally been built using flat Layer 2 native ethernet technologies.  But traditions sometimes persist even when the reasons for them no longer exist.  In the past, many people defaulted to Layer 2 networks because switching was less expensive and Layer 2 seemed simpler than IP/MPLS networks.  But big shifts in the economics of routing silicon have made routers more affordable and innovations like Segment Routing have made MPLS much simpler to deploy.  

There are many benefits to bringing IP and MPLS to the access network:

- Arbitrary Topologies: To prevent broadcast storms and duplicate packets, traditional Layer2 topologies must not contain loops.  G.8032 is a common loop prevention technique that requires an underlying ring topology.  In other topologies, Spanning Tree Protocol is required to block redundant ports.  Because IP networks use TTL to prevent loops, arbitrary topologies can be supported without disabling redundant paths.
- Reconvergence: In Layer 2 networks, MAC-learning occurs in the data plane.  Changes in topology (i.e. link failures) require flushing all MAC tables and subsequent flooding of traffic.  This can caused extended reconvergence times. L3 networks, on the other hand, can reconverge very quickly
- Troubleshooting: It can be difficult to know the actual path in complex Layer 2 networks.
- Efficiency:  Because redundant paths must be blocked in Layer 2 topologies, there is no notion of Equal Cost Multi-Pathing (ECMP).  IP networks support equal-cost multi-path and active-active redundancy, so that all links can be used at the same time. 
- Resilience: Layer 2 networks achieve multi-homing only with active-standby MC-LAG, which is vulnerable to traffic storms.  Layer 3 networks can achieve multi-homing with EVPN-based LAG which supports active-active redundancy.


### Hardware Components

#### NCS 540 Series
The NCS 540 family of routers are powerful and versatile access routers capable of aggregating 10G uplinks from XGS-PON devices.
More information on the NCS 540 router line can be found at:
https://www.cisco.com/c/en/us/products/routers/network-convergence-system-540-series-routers/index.html

#### NCS 5500 / 5700 Fixed Chassis
The NCS 5500 / 5700 fixed series devices are validated in access and aggregation roles.  The NCS-55A1-24Q6H-S and NCS-55A1-24Q6H-SS have 24x1GE/10GE, 24x1GE/10GE/25GE, and 6x40GE/100GE interfaces. The 24Q6H-SS provides MACSEC support on all interfaces. The NCS-55A1-24Q6H series also supports 10GE/25GE DWDM optics on all relevant ports.

#### ASR 9000
The ASR 9000 is the router of choice for edge services. The Routed Access for Rural Broadband utilizes the ASR 9000 in a PE function role, performing Pseudowire headend termination for per-subscriber policy with BNG.

#### Cisco 8000
The Routed Access design includes the Cisco 8000 family. Cisco 8000 routers provide the lowest power consumption in the industry, all while supporting systems over 200 Tbps and features service providers require. The Cisco 8000 can fulfills the role of aggregation router in the design. Service termination is not supported on the 8000.

### Design Components

#### Topology Options
Unlike the tidy racks of data center networks, the physical design of broadband networks tends to be messy, dominated by the geography and population density of the area to be served.  Homes connect back to an Optical Line Terminal (OLT) located at a point-of-presence, central office or outside plant in a point-to-multipoint topology.  An XGS-PON OLT can serve homes in a 20 to 100 km radius.  A small, remote OLT (hardened for outside conditions) with 4 ports and a 64-way split could serve up to 256 homes.  A 16-port PON shelf with the same 64-way split might serve 1024 homes within a 40 km radius.  For denser, less rural areas, larger PON shelves can be stacked to serve tens or hundreds of thousands of homes within the supported radius.

Once the user’s traffic reaches the OLT, it gets handed off to the access network which takes that traffic to the Internet (or other service).  The design of access networks is also determined by geography and the availability of fiber.   Two typical designs are ring and point-to-point.

In a routed ring access topology, routers are connected in a ring.  Because traffic can flow in either direction, the ring topology provides an automatic backup path if a link or router fails. 

[insert picture]

In a point to point access architecture, the access routers can be single or dual-homed to the central office .  Dual-homing enables redundancy should either of the uplinks fail.  

[insert picture]

Regardless of how the routers connect back to the Internet (ring or point-to-point), each router serves as an aggregation point for individual OLTs, OLT rings or OLT trees.  Depending on the availability requirements, those OLTs can be single or dual-homed to the routers.  

### Routing and Forwarding

The foundation technology used in this design is Segment Routing (SR) with a MPLS based Data Plane.  Because rural broadband designs are typically small enough to run a single IGP domain, the focus of the design is on intra-domain forwarding.  For networks that exceed the scale limits of a single domain, refer to the Inter-Domain Operation guidelines of the Converged SDN Transport design.
Segment Routing dramatically reduces the amount of protocols needed in a Service Provider Network. Simple extensions to traditional IGP protocols like ISIS or OSPF provide full Intra-Domain Routing and Forwarding Information over a label switched infrastructure, along with High Availability (HA) and Fast Re-Route (FRR) capabilities.
Segment Routing introduces the idea of a Prefix Segment Identifier or Prefix-SID.  A prefix-SID identifies the router and must be unique for every router in the IGP Domain. Prefix-SID is statically allocated by the network operator in the IGP configuration process. The Prefix-SID is advertised by the IGP protocol which eliminates the need to use LDP or RSVP protocol to exchange MPLS labels.
The Routed Access for Rural Broadband design uses IS-IS as the IGP protocol.
Fast Re-Route using TI-LFA
Segment-Routing embeds a simple Fast Re-Route (FRR) mechanism known as Topology Independent Loop Free Alternate (TI-LFA).
TI-LFA provides sub 50ms convergence for link and node protection. TI-LFA is completely stateless and does not require any additional signaling mechanism as each node in the IGP Domain calculates a primary and a backup path automatically and independently based on the IGP topology. After the TI-LFA feature is enabled, no further care is expected from the network operator to ensure fast network recovery from failures. This is in stark contrast with traditional MPLS-FRR, which requires RSVP and RSVP-TE and therefore adds complexity in the transport design.

### MPLS VPN Services
One of the advantages of an IP/MPLS network is that it enables the deployment of VPN services.  Many people associate L3VPN and L2VPN with expensive business services.  But even small providers can benefit from judicious use of MPLS VPNs in residential deployments.

In the simplest design, subscriber traffic arrives at the access device, receives an IP address via DHCP and gets access to the Internet via the global routing table. Many larger, modern networks are being built to isolate the global routing table from the underlying infrastructure. In this case, the Internet global table is carried as an L3VPN service just like any other VPN service, leaving the infrastructure layer protected from both the global Internet.  

The other use case for MPLS VPN services is when Broadband Network Gateway (BNG)  is deployed for per-subscriber traffic management.  Because BNG requires more sophisticated treatment of the user traffic and, hence, more expensive forwarding hardware, the BNG device is often deployed in a centralized location to take advantage of economies of scale.  Access devices can be configured to backhaul Layer 2 subscriber traffic to the BNG device using EVPN pseudowires.

### QOS
Quality of Service (QoS) refers to the ability of a network to provide better service to various types of network traffic.  To achieve an end to end QoS objective (i.e. between the subscriber and the Internet), it’s necessary to consider the different mechanisms in the PON network and routed access network.

QoS can be applied in two directions: upstream (from the subscriber to the Internet) or downstream (from the internet to the subscriber).  In PON networks, upstream traffic is controlled by the OLT, which implements a flow-control mechanism that lends itself to a simpler implementation of QoS.  The ITU-T G.983.4 recommendation coupled with other QoS mechanisms enable the OLT to effectively schedule, shape and prioritize upstream traffic.

While upstream traffic management in the OLT is well-defined and largely consistent across vendors, downstream traffic management can often be more effectively carried out deeper in the access and core network.  

The platforms in the design have sophisticated and rich QoS feature sets that are widely deployed in large-scale, converged, multi-service provider networks.  Even on the smallest platforms, QoS has many options and can be endlessly customized.  Rural broadband networks, however, tend to be relatively simple and well-supplied with bandwidth.  Therefore, the goal of this design is to get as much value as possible from the simplest deployment of QoS. Simple policies with basic marking and egress queuing are often sufficient and require less monitoring and maintenance than more complex policies.

QoS policies applied to network interfaces act on the aggregate subscriber traffic on that interface. This, combined with OLT QoS capabilities, may be sufficient for simple deployments of sparsely populated areas, may be sufficient for some RBB providers.  Other providers may need to leverage a BNG solution for finer-grained QoS policies.  

## Use Cases
### High Speed Residential Data Services
The primary use case for rural broadband deployments leveraging federal grants such as BEAD is a high speed (100M minimum) data service.  Other offerings, such as telephone
and video, may be offered over a funded network that meets the unicast data requirements but those services are subject to additional regulation.

Depending on the configuration of the PON network, unicast subscriber traffic will arrive at the access router from the OLT in one of two ways: VLAN per service (N:1 VLAN) or VLAN per subscriber (1:1 VLAN).  In the N:1 VLAN model, a service (e.g. high-speed data) is represented by a VLAN.  Many (“N”) subscribers access that service via a single (“1”) VLAN.  Other services, like voice and video, use other VLANs.  This is the simplest solution to deploy and maintain.

In the 1:1 VLAN model, each subscriber is assigned their own VLAN.  Typically, the 1:1 VLAN model is deployed using  Q-in-Q double tagging (C-VLAN + S-VLAN) to overcome the scale limitations of 4096 VLANs. However, managing thousands of customer VLANs can become administratively heavy, especially for provisioning, troubleshooting and documentation.

The routed access design for rural broadband can support either VLAN model.  

### High Availability for PON

Whether you’re deploying a single OLT shelf, a tree of OLTs or a ring of OLT, the connection from the OLT to the access router represents a potential point of failure.  To protect against the failure of a single port on either the router or the OLT, multiple links are commonly be bundled together for a redundant connection.  The router, however, remains a single point of failure.

To protect against a router failure, the OLT can be dual-homed to two different routers using EVPN.  EVPN enables “all-active” redundant links to two or more routers.  The  OLT believes that it is connected to a single device using a normal bundle interface.  

The routed access design for rural broadband supports a single, non-redundant uplink, a bundled interface to the same router, and a bundled interface to two redundant routers with EVPN.

### Per-subsciber Policies with BNG
To manage individual subscriber sessions, providers can implement Broadband Network Gateway (BNG).  BNG acts as a gatekeeper, verifying that only approved subscribers can get access to the network and implementing per-subscriber policies (such as QoS or access-control lists) to improve security and the quality of subscriber experience.

BNG manages all aspects of subscriber access including:
- Authentication, authorization and accounting of subscriber sessions
- Address assignment
- Security
- Policy management
- Quality of Service (QoS)

Cisco’s implementation of BNG is a mature, feature-rich technology that supports many use cases of varying degrees of complexity.  The RBB design focuses on a simple IP over Ethernet (IPoE) deployment that authenticates subscribers, assigns addresses using DHCP, and applies per-subscriber QoS policies.  

