---
published: false 
date: '2019-07-29 11:00-0400'
title: Advanced L2/L3 Services using Segment Routing and EVPN  
excerpt: In this blog we look at advanced Segment Routing transport and EVPN service use cases. These include multi-plane networks, constraint-based traffic engineering, EVPN multi-homing, and Anycast IRB for L2VPN/L3VPN integration.   
author: Phil Bedard
tags:
  - iosxr
  - evpn 
  - segment routing 
  - design  
  - backhaul 
  - mobile  
position: hidden 
---


# Traffic Engineered Services  
In the simple examples, the ingress PE will simply use any available SR-MPLS forwarding path to the egress PE, the BGP next-hop of the EVPN or L3VPN service prefix. SR-TE gives us the ability to create engineered paths across the network to the egress PE, using a number of potential constraints. There are also different ways  
### Low-Latency P2P L2 interconnect 
In this example we will configure the P2P EVPN-VPWS service to use a specific low latency SR-TE policy across the network.   
<b>Methods to configure low latency path</b> 
#### Static defined SR Policy on head-end node 
#### On-demand next-hop to dynamically create SR Policy on demand 
### Diverse L2 P2P services using SR Flex-Algo 


## L3 Services using EVPN and L3VPN

### "L3 IXP" Design 
Traditional IXPs are designed using a L2 fabric, native or emulated. Traditional L2 IXPs use either native or emulated L2 fabrics, exposing the fabric and participants to the unwanted characteristics associated with L2 networks. The bevy of security features required are due to these negative characteristics. We can build a L3 IXP today by using P2P interfaces to each participant and simply route between them. EVPN using IRB with proxy-arp provides a fabric where I do not need to add default routes to the CE host, they can simply rely on the same ARP mechanism as previously done when all multi-point hosts are in the same subnet, but provides the isolation of a L3 interface.  

### First-hop Redundancy Protocol (FHRP)using EVPN multi-homing and Anycast IRB  
One simple use case for EVPN is to provide simplified L3 multi-homing by eliminating the scale and L2 switching requirements of VRRP or HSRP. We utilize the concept of Anycast Integrated Routing and Bridging to allow a redundant L3 interface be created within an EVPN instance. This IRB can be located within a L3VPN or in the global routing table. In a simple L3 IXP connectivity example the intra-subnet and inter-subnet routing is done using EVPN's built-in route types.  It is recommended to carry all L3 services within a VPN so the base infrastructure does not share the same routing and forwarding plane as services. This enhances security of the infrastructure layer.    