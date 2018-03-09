---
published: true 
date: '2018-08-01 11:00-0400'
title: Express Peering Fabrics 
excerpt: In Part 2 of this blog series on transforming regional and metro networks for efficient OTT video delivery, we'll explore how an Express Peering Fabric can optimize networks for unicast video delivery.  
author: Phil Bedard
tags:
  - iosxr
  - Peering
  - Design
position: hidden 
---

{% include toc %}

**Express Peering Fabric**
============================

Overview 
---------
Regional SP networks serving residential subscribers are typically deployed in an aggregation/access hierarchy using logical Ethernet connections over a regional optical transport network, as shown in Figure 1 below.  The aggregation nodes serve as an aggregation point for connections to regional sites along with acting as the ingress point for traffic coming from the SP backbone. SPs can drive even greater efficiency by selecting specific high bandwidth regional sites for core bypass. This is simply connecting the regional hub routers directly to a localized peering facility or facilities, bypassing the regional core aggregation nodes which are simply acting as a pass through for the traffic. This is called an **Express Peering Fabric.** Due to the growth in Internet video traffic, this secondary express peering network in time will likely be higher capacity than the original SP converged network. The same express peering network can also be used to serve content originated by the SP, leaving the converged regional network to serve other higher priority traffic needs.

Not only is the express delivery network design a more efficient logical design, it can also use a simplified control-plane as the network does not need to support more complex network services or multicast video delivery. The RIB and FIB resources to carry video delivery routes are also reduced, requiring less power and memory resources than a device capable of carrying a full Internet routing table. Service providers are advised to look for hardware supporting flexible FIB options delivering the greatest environmental efficiency.    

## Figure 1: Traditional Peering and Content Delivery
![peering-express-bypass1.jpg]({{site.baseurl}}/images/peering-express-bypass1.jpg)

## Figure 2: Optimized Regional Express Peering Fabric  
![peering-express-bypass2.jpg]({{site.baseurl}}/images/peering-express-bypass2.jpg)

Regional Transport Design 
-------------------------
### Flexible Photonic Network 
One of the key building blocks to an express delivery networks is flexibility in placement of circuits between ingress peering endpoints and end user locations. The regional transport network must allow DWDM wavelengths direct reach between peeering and content locations to subscriber locations without additional router hops. The lowest layer block is a flexible photonic layer providing any-to-any wavelength connectivity through multi-degree colorless and contentionless ROADMs and add-drop complexes. The Cisco NCS2000 with its intelligent high-density multi-degree ROADMs and GMPLS control-plane give providers the flexible photonic layer they need to build a more efficient express traffic delivery network. 

### On-net Content Source Facilities 
In some instances, providers have built linear extensions from regional peering locations to core aggregation sites since all connectivity went between the peering routers to the metro core aggregation routers. In order to eliminate redundant hops, the peering locations must be connected to upstream ROADMs to directly reach subscriber locations. There is generally very little cost incurred with adding additional multi-degree ROADMs today, and their use greatly increases network flexibility. While it's most beneficial to have the peering location connected to diverse sites via a fiber ring, even a linear route connected via ROADM will pay dividends in network agility and efficient connectivity. 

### Coherent Optics 
Another key to the transport design is the use of coherent transponders or coherent integrated IPoDWDM ports. Coherent optics give transport networks longer reach without regeneration and flexibility through tuning across 80+ channels. This tuning flexibility allows the ability to connect router interfaces to any wavelength across an optical transport network. High-density 100G is typically done through 100G muxponders and transponders, while integrated IPoDWDM coherent router interfaces support 100 or 200G per port for sites which may not support a transport shelf deployment. 

Network Modeling 
----------------
Network modeling must be performed to determine which sites are candidates for direct connectivity to content locations. The modeling is based on factors such as statmux gain, component cost, and resource cost such as DWDM wavelengths. A simplified traffic demand matrix needs to be computed from the ingress traffic location to the egress customer sites. Netflow can be used as a tool to determine how much traffic is being sent to customer prefixes at each site. Alternative to Netflow, networks using MPLS can derive the stats to each egress router using either MPLS FEC or TE Tunnel statistics. Once the traffic matrix has been computed, a network model can be created with and without bypass links to calculate the total number of router interfaces and transport links needed. There will be an optimal traffic percentage where connecting a bypass link aids efficiency. In some cases however, traffic growth may be projected to be high enough over time to connect all sites day one.  

Control Plane Design 
--------------------
In most cases the peering or content location routers will be connected to both an end location as well as the metro core aggregation network. Care must be taken to make sure the end site locations do not act as transit paths between content location and the core. In order to create an isolated domain, use carefully selected metrics to ensure traffic does not flow through the wrong links. Another option is to use a separate IGP process entirely for the express network, ensuring the end site nodes cannot become transit nodes from the content location to the core aggregation nodes. Using multiple loopback addresses is recommended in that instance to create additional separation between networks. More advanced techniques may also be used such as using Segment Routing Policies to define an express routing plane across the regional network.  

**Should I build an Express Fabric?**
=====================================
There are several factors that go into whether or not building an Express Peering fabric is the right approach for your network. Most important is to analyze the traffic coming into your network from external peers and determine the true network cost of the traffic path from ingress to egress. Building a detailed network cost model incoporating physical fiber, optical transport, and IP networks will allow you to gain insight into how much each hop of the network path costs at each layer and combined. An advanced network modeling tool such as Cisco WAE Planning can help build a network model and simulate the current network as well as potential Express Fabric designs to determine if building an Express Network is an efficient solution. However, if you have taken the steps to build a local peering location, an Express Fabric is the next logical step in reducing cost from ingress peer to customer endpoints.  

**Cisco Express Peering Fabric Components**
===========================================
Cisco optical transport and routing platforms allow providers to build the most efficient transport delivery. The NCS2000 and its family of integrated muxponders can support 96 channels at up to 250G per wavelength. The 2RU NCS10002 muxponder provides a flexible 2Tbps of capacity in an external shelf in a platform running IOS-XR, supporting rich telemetry and automation capabilities. The NCS5500 routing platform has flexible fixed and modular chassis options. The 1RU NCS-55A1-36H-SE has 36 100G interfaces with a 4M IPv4 FIB capacity. The modular NCS-5504 and NCS-5508 support the same scale in each line card slot. A 6x200G IPoDWDM line card can be used to extend connections over passive optical muxes or dark fiber at up to 200G per interface.    

 
**Additional Efficiency Options**
--------------------------------- 

Local Caching Nodes 
-------------------
Placing CDN cache nodes directly into service provider aggregation or end subscriber locations can also reduce cost and netowrk complexity. The CDN nodes can be 3rd party cache nodes supplied by a content provider, such as the Netflix OpenConnect appliance, or internal CDN nodes delivering service provider video content. The main benefits to using local cache nodes are reduction in network resources and improved QoE for subscribers. The cache hit rate or efficiency of the nodes varies but in general they are very good and for very high bandwidth flash events, like the release of a new season of a TV series, the majority of content can be delivered locally. Using distributed caches which also serve as origins for downstream caches can help emulate a multicast delivery network without the operational headaches of multicast.

Aggregating caching nodes requires high speed routers with adequate buffering capacity due to the bursty traffic profile of video traffic. The NCS-5500 has deep buffers along with the 10G, 25G, and 100G density required to satisfy cache node aggregation needs. Scale-out network design allows providers to build delivery fabrics in the hundreds of Tbps.   

Work has been done to standardize caching infrastructure through the Streaming Video Alliance, found at https://www.streamingvideoalliance.org. The Streaming Video Alliance is a consortium of service providers, network hardware and software vendors, and content networks. The Open Cache initiative is meant to create a caching server capable of caching any content, owned and operated by the service provider. Work has been done by the IETF CDNI working group to define a framework of how caching nodes interconnect and route requests between providers, and the Open Cache WG in the SVA has adopted most of that architecture. There are however many challenges to open caching such as content encryption, quality of experience metrics, and efficient request routing.    
ICN 
---
Information Centric Networking has gained much research exposure over the last several years, with two primary archtectures being Concent Centric Networking and Named Data Networking. The premise behind ICN is the Internet is almost completely content-driven today, so request routing and delivery should be based off content names and not IP addresses. It tackles the concept of location vs. content identifier. Caching is ubiquitous in the ICN architecture to aid in efficient content delivery. Typically every ICN router has one or more cache nodes to serve local content from when additional requests are made. ICN currently is mostly a research effort, with work being led by the IETF ICNNG working group. CCN and NDN networks can be created as overlays over IP using Linux software as a way to explore the architecture and routing constructs of ICN.   