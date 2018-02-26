---
published: true 
date: '2018-08-01 11:00-0400'
title: Internet Traffic Trends 
excerpt: Internet Video Fuels Bandwidth Growth 
author: Phil Bedard
tags:
  - iosxr
  - Peering
  - Design
position: hidden 
---

{% include toc %}

**Introduction**
===============

Over the last few years a Internet traffic has been driven by one dominant source, unicast video. The rise in unicast video has its roots in how users began consuming video content versus traditional broadcast video. These new delivery mechanisms require rethinking networks to make the most efficient use of resources at all layers. This blog will first cover howe we arrived where we are today, and then cover network architecture and technology to help improve network efficiency to deal with the rising tide of unicast video.  

**Unicast Video Growth**
==================

Broadcast Video History
-----------------------
Television video delivery for many years followed the same path blazed by radio before it, broadcasting a single program over the air at a specific time to anyone within range of the signal. Cable networks were built in the 70s and 80s, with the promise of delivering a wider variety of content to subscribers not subject to the same impairments as over the air (OTA) broadcasts without the use of antennas. The networks however were still analog and built for broadcast delivery of all video to every user. Satellite video delivery worked in much the same way, simply broadcasting all signals and requiring the end device tune to the channel at a specific time. While broadcast video has limitations on flexibility for users, it has the ultimate efficiency when it comes to network resources as the signal is broadcast once to all users at the origin.   

Video on Demand
-----------
Video on Demand, or VoD, originated in the late 1980s and rose to prominence in the 1990s as a way to deliver video content to users on their own timeframe. Users could now select what they wanted to view and have it be delivered to them immediately. It took the reduction in cost of the base infrastructure components, mainly storage and compute, to make a service like VoD a reality. VoD was also seen as a way to further monetize the cable network and challenge the huge video rental business that existed during those times with Pay Per View (PPV) content. 

VoD delivery used two different methods for delivery back in its original form, push or pull. In the push method, content was delivered via broadcast to a single or all subscribers and stored locally on a device such as a set-top box, for viewing later. The pull method streamed the content to the subscriber device from a remote server on user demand. In the end, the pull method easily won out due to the much larger variety of content available and less costly end user device. In order to support a single user viewing content destined for only their device, it required dedicating analog spectrum for the channel. It was still broadcast to a number of users, but encrypted such that only the paying subscriber could view it. Even though it was still broadcast at the lowest level, the content was unicast to a specific subscriber by the stream consuming resources for a single video stream. VoD fundamentally changed users viewing habits and also introduced the first unicast video delivery.    

Video over IP 
--------------
Service providers who built out wireline networks using DSL and Ethernet technology, data-link protocols not having a native analog video delivery method, looked at IP as the higher layer protocol to deliver video content to users. These networks were deployed to take advantage of multicast, a subset of the broadcast capability inherent in Ethernet, and standardized for IP in RFC 1112. IP multicast improves network efficiency by implementing frame replication in the network devices, combined with a set of control-plane protocols to create optimized distribution trees. In its simplest form IP multicast replicates a broadcast network, sending all channels to all users (dense mode), and some providers used this method. However, to improve network efficiency it is now most common for end devices to use protocols like IGMP (v4) and MLD (v6) so optimized multicast trees are built. This type of multicast IP delivery is known as IPTV and is implemented in NA by networks such as AT&T UVerse and Google Fiber.  

Supporting VoD on these networks requires delivering video over IP. Similar mechanisms can be used as analog networks, using a specific multicast address for the subscriber stream. However, instead of simulating a unicast stream using a more complex multicast process, streaming the content as a to a unicast IP address assigned to a device is much simpler and supported a wider range of devices, even across networks that do not support native multicast delivery. Today more and more content on wireline networks is delivered using unicast IP, even on traditional cable networks, due to its flexibility and the ability to serve content to a variety of end user devices from a single content source. The flexibility and ease of delivery using unicast IP has superceded the inefficiencies of delivering duplicate content over the same network resources.   

Over the Top IP Video
--------------------- 
The unicast video content described above has typically been contained within a service provider network. As the Internet has grown and bandwidth to end users increased, video content from alternative sources emerged. Broadband Internet became more widely available in the mid 2000s and with it came user-generated video providers like YouTube along with traditional media rental companies like Netflix embracing streaming video for rental delivery. These Internet content providers deliver video "over the top" (OTT) of service provider networks since the origin and destination are applications controlled by the content provider. The growth of OTT Internet video has continued to climb rapidly over the last decade along with IP video in general. IP video accounted for 73% of all Internet traffic in 2016, and by 2021 will account for 82% of all Internet traffic.  (cite Cisco VNI).

It is not only on-demand content driving OTT growth, streaming of traditional broadcast video like sports to mobile devices, 3rd party devices like tablets, smart TVs, and additional endpoints is increasing in popularity. The last few years have seen a number of new services delivering traditional linear TV using OTT IP delivery. Over the top video is by nature unicast, as each stream is simply sent on demand when a user clicks "play." Since there is little efficiency in sending a single stream to each user, it causes tremendous strain on network resources. It is however a trend that is unlikely change, so new methods need to be employed to improve network efficiency.   


**Producers and Consumers** 
==========================

Content Providers 
-----------------
Content providers are those who originate video streams. A content source could also be an eyeball network providing video content to its own subscriber base, or an OTT Internet video source. A content provider may not be the original origin of the content, but is simply a means to deliver the content to the end user.  

Caching and Content Delivery Networks
-------------------------
Caching is the process of keeping local copies of content to serve to local users instead of utilizing network resources to retrieve the content each time the content is accessed. Caching of Internet content became popular with the rise of the Internet in the late 1990s with open-source software such as Squid and commercial products like Cacheflow and Cisco WAAS. Called "transparent" caches, they intercept content without the source or destination having knowledge of the caching. The content in those days was mainly primitive static content, but with the high cost of bandwidth, it was still sometimes beneficial to cache content. Transparent caches have evolved into systems today targeted at OTT providers, and use more sophisticated techniques to cache video content from any source. However, with the rise of end to end encryption use, transparent caches are no longer a realistic option for serving content closer to users.  
Content Delivery Networks (CDN) have been around for many years now, with the first major CDN Akamai going live in 1999. The aim of a CDN is to place content closer to end users by distributing servers closer to end users. CDNs such as Akamai, Limelight, and Fastly host and deliver a variety of content from their customers. In addition to more generic CDNs, content owners have built their own CDNs to deliver their own content. Examples of dedicated CDNs include the Netflix OpenConnect network and Google Global Cache network. Most new streaming video providers utilize a distributed CDN to deliver content.   

Eyeball Networks
----------------
Wireless and wireline service providers providing the last mile Internet connection to end users are commonly known as "Eyeball" networks, because the all content must pass through those networks for end users to view it. Around the world, and especially in North America, consolidation of service providers have left relatively few Eyeball networks serving a large number of subscribers.   

**Efficient Unicast Video Delivery**
==================================== 

What is Network Efficiency? 
---------------------------
Network efficiency in this context refers to minimizing the cost and consumption of network resources such as physical fiber, wavelengths,and IP interfaces to deliver unicast video content to end users. The equation to delivering video traffic efficiently is to create a network model reducing the distance, network hops, and network layer transitions between the content provider and content consumer while maintaining statistical multiplexing through aggregation where beneficial.  

Role of Internet Peering 
------------------------
Internet Peering is the exchange of traffic between two providers. Peering originated at third party carrier-neutral facilities known as Internet Exchange Points (IXP), with the exchange providing a public fabric to interconnect service providers. Due to consolidation and the dominant traffic type being video, the Internet has evolved from most content flowing through a Tier-1 Internet provider via transit connections to one of direct traffic exchange between content providers and eyeball networks. (Quote death of transit NANOG presentation). The majority of Internet video traffic today is exchanged via private network interconnection (PNI) between content providers and eyeball networks. The traditional large IXPs still act as meet-me points for many providers, facilitating both public and private interconnection, but improving network efficiency demands traffic take shorter paths.    

Localized Peering
-----------------
 Reducing the distance and network hops between where unicast video packets enter your network and exit to the consumer is a key priority for service providers to reduce network cost. Each pass through an optical transponder or router interface adds additional cost to the transit path, especially on long-haul paths from traditional IXPs to subscriber regions. The aforementioned rise in video traffic demands peering move closer to the edges of the network to serve wireline broadband subscribers along with high-bandwidth 5G mobile users. Content providers have invested heavily in their own networks as well as distributed caches serving content from any network location with Internet access. Third party co-location providers have begun building more regional locations supporting PNI between content distributors and the end subscribers on the SP network. This leads to a localized peering option for SPs and content providers, greatly reducing the distance and hops across the network. As more traffic shifts to becoming locally delivered building additional regional or metro peering locations becomes important to ensure less reliance on longer paths during failures. 

 Service Provider Unicast Delivery Headend 
------------------------------------------
As mentioned, service providers are seeing growth not only in OTT unicast video delivery, but also delivery for their own video services. Most service providers have deployed their own internal CDNs to provide unicast video content to their subscribers and migrate VoD off legacy analog systems onto an all-IP infrastructure. The same efficiency tools for deling with off-net content from peers applies to on-net video services. There may be efficiencies gained in placing SP content servers in the same facilities as other content peers, aggregating all content traffic in a single location for efficient delivery to end users.   


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
In some instances, providers have built linear extensions from regional peering locations to core aggregation sites since all connectivity went between the peering routers to the metro core aggregation routers. In order to eliminate redundant hops, the peering locations must be connected to upstream ROADMs to directly reach subscriber locations. There is generally very little cost incurred with adding additional multi-degree ROADMs today, and their use greatly increases network flexibility. While it's most beneficial to have the peering location connected to diverse sites via a fiber ring, even a linear route connected via ROADM will pay dividends in greater efficiency. 

### Coherent Optics 
Another key to the transport design is the use of coherent transponders or coherent integrated IPoDWDM ports. High-density 100G is typically done through 100G muxponders and transponders, while integrated IPoDWDM can provide 200G per port for sites which may not support a transport shelf deployment. The NCS2000 and its family of integrated muxponders can support 96 channels at up to 250G per wavelength. The 2RU NCS10002 muxponder provides a flexible 2Tbps of capacity in an external shelf in a platform running IOS-XR, supporting rich telemetry and automation capabilities. In IPoDWDM use cases, the Cisco NCS5500 supports 1.2Tbps per slot using its 6x200G IPoDWDM line card.   

Network Modeling 
----------------
Network modeling must be performed to determine which sites are candidates for direct connectivity to content locations. The modeling is based on factors such as statmux gain, component cost, and resource cost such as DWDM wavelengths. A simplified traffic demand matrix needs to be computed from the ingress traffic location to the egress customer sites. Netflow can be used as a tool to determine how much traffic is being sent to customer prefixes at each site. Alternative to Netflow, networks using MPLS can derive the stats to each egress router using either MPLS FEC or TE Tunnel statistics. Once the traffic matrix has been computed, a network model can be created with and without bypass links to calculate the total number of router interfaces and transport links needed. There will be an optimal traffic percentage where connecting a bypass link aids efficiency. In some cases however, traffic growth may be projected to be high enough over time to connect all sites day one.  

Control Plane Design 
--------------------
In most cases the peering or content location routers will be connected to both an end location as well as the metro core aggregation network. Care must be taken to make sure the end site locations do not act as transit paths between content location and the core. In order to create an isolated domain, use carefully selected metrics to ensure traffic does not flow through the wrong links. Another option is to use a separate IGP process entirely for the express network, ensuring the end site nodes cannot become transit nodes from the content location to the core aggregation nodes. Using multiple loopback addresses is recommended in that instance to create additional separation between networks. More advanced techniques may also be used such as using Segment Routing Policies to define an express routing plane across the regional network.  

**Should I build an Express Fabric?**
=====================================
There are several factors that go into whether or not building an Express Peering fabric is the right approach for your network. Most important is to analyze the traffic coming into your network from external peers and determine the true network cost of the traffic path from ingress to egress. Building a detailed network cost model incoporating physical fiber, optical transport, and IP networks will allow you to gain insight into how much each hop of the network path costs at each layer and combined. An advanced network modeling tool such as Cisco WAE Planning can help build a network model and simulate the current network as well as potential Express Fabric designs to determine if building an Express Network is an efficient solution. However, if you have taken the steps to build a local peering location, an Express Fabric is the next logical step in reducing cost from ingress peer to customer endpoints.  

 
**Additional Efficiency Options**
--------------------------------- 

Local Caching Nodes 
-------------------
Placing CDN cache nodes directly into service provider aggregation or end subscriber location can also be a way to reduce cost and netowrk complexity in delivery unicast video content. The CDN nodes can be 3rd party cache nodes supplied by a content provider, such as the Netflix OpenConnect appliance, or internal CDN nodes deliverying SP video content. The main benefits to using local cache nodes are reduction in network resources and improved QoE for subscribers. The cache hit rate or efficiency of the nodes varies but in general they are very good and for very high bandwidth flash events, like the release of a new season of a TV series, are extremely high. Using distributed caches which also serve as origins for downstream caches can help emulate a multicast delivery network without the operational headaches of multicast. 

Work has been done to standardize caching infrastructure through the Streaming Video Alliance, found at https://www.streamingvideoalliance.org. The Streaming Video Alliance is a consortium of service providers, network hardware and software vendors, and content networks. The Open Cache initiative is meant to create a caching server capable of caching any content, owned and operated by the service provider. Work has been done by the IETF CDNI working group to define a framework of how caching nodes interconnect and route requests between providers, and the Open Cache WG in the SVA has adopted most of that architecture. There are however many challenges to open caching such as content encryption, quality of experience metrics, and efficient request routing.    

There are also a few downsides to placing cache nodes in SP facilities. The space and power required for servers may be relatively high compared to their delivery throughput. As storage, compute, and server networking become more efficient this hurdle becomes less over time. In many cases it may also require another device to aggregate those connections.    

ICN 
---
Information Centric Networking has gained much research exposure over the last several years, with two primary archtectures being Concent Centric Networking and Named Data Networking. The premise behind ICN is the Internet is almost completely content-driven today, so request routing and delivery should be based off content names and not IP addresses. It tackles the concept of location vs. content identifier. Caching is ubiquitous in the ICN architecture to aid in efficient content delivery. Typically every ICN router has one or more cache nodes to serve local content from when additional requests are made. ICN currently is mostly a research effort, with work being led by the IETF ICNNG working group. CCN and NDN networks can be created as overlays over IP using Linux software as a way to explore the architecture and routing constructs of ICN.   
