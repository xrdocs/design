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

