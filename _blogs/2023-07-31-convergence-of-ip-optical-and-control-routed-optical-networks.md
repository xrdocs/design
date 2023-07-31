---
published: true
date: '2023-07-31 14:14 +0300'
title: 'Convergence of IP, Optical and Control: Routed  Optical Networks'
author: Ori Gerstel
excerpt: The control system for Routed Optical Networks
tags:
  - iosxr
---

Convergence of packet and optical network technologies has been attempted for many years, but it is only happening now, why? Because of RON...
Routed Optical Networking (RON) is an architecture that combines optical networking technologies and packet network technologies into a single network that provides all types of services to customers from a single platform. This new paradigm has always been appealing because the resulting network has less layers and an overall simpler structure, but it is finally becoming reality for the following reasons:
* Advancements in Silicon enabling coherent WDM transceivers that fit into standard router ports without compromising on density
* Significant increase in volume for both pluggable and router Silicon due to massive deployments by web-scale companies driving down the cost
* These two factors enable dramatically lower overall cost (78%), power consumption (97%) and lower footprint (95%) - the numbers in parentheses are based on customer reported savings.
* Mature management and control architecture based on a hierarchical approach that allows for much easier and faster innovation compared to the control plane approach of the past (remember GMPLS?). 

Today’s collection of disparate management tools is being replaced with per technology SDN controllers orchestrated by a Hierarchical Controller (HCO) enabling simpler provisioning, troubleshooting, performance monitoring and much more. 
More details about Routed Optical Networking (RON) can be found here: 
[Routed Optical Networking - HLD](https://xrdocs.io/design/blogs/latest-routed-optical-networking-hld)

Achieving a full multivendor solution implies that the control system must have in depth understanding of how each domain and each technology must be configured, as well as an understanding of the possible failure modes, allowing for detailed troubleshooting. This implies that the control system must comprise of vendor specific tools that provide intimate knowledge of the different domains, and an umbrella controller on top that integrates information from all the domains into a single vendor-agnostic database. In other words, a hierarchical structure of domain controllers and a hierarchical controller as shown in the following figure.

![Picture1.png]({{site.baseurl}}/images/Picture1.png)

This architecture has been widely adopted by standardization bodies (e.g. IETF, ONF, MEF) and the major Service Providers all over the world. It is part of a larger hierarchy that includes OSS tools like service orchestrators and assurance systems and includes other parts of the Service Provider network such as access networks and data center resources. 
The following figure provides its mapping against IETF ACTN architecture (RFC 8453), where a clear and clean boundary separation between the packet and optical domains is provided and the Hierarchical Controller (MDSC) is the only entity capable to manage multidomain and multilayer services from a single UI/NBI.

![Picture2.png]({{site.baseurl}}/images/Picture2.png)

A few alternative architectures have been proposed by different vendors, spanning from an ambitious single “godbox” controller that knows everything about the IP layer and optical layer, to more modest attempts to just control the pluggable in the router via the optical controller of the line system.
The godbox idea is clearly a bad idea: have we not learned from the past? This didn’t work well even when controlling IP and optical gear of the same vendor, let alone attempting to do it across the entire industry and keep it up to speed and fully tested against all possible gear...
The more modest concept of controlling the pluggables in the router by the optical controller doesn’t sound so bad – after all this controller controls transponders, so what’s the difference? The state of the router and its pluggables is mainly owned by the IP controller and having multiple owners (optical controller for the WDM pluggables, IP controller for the rest of the router) is a recipe for resource contention, race conditions, synchronization issues, and security headaches. 
Add to this the fact that the state of the pluggables is not independent of the state of the rest of the router: when you modify parameters on the pluggable, you affect IP layer behavior. For example, if you change the modulation format to a lower bitrate format, the IP layer needs to know about this and send less traffic down the link.
But the problem is even worse than that: many routers connect to other routers via different vendor line systems: for example, an edge router may connect to a core router via a core WDM system and to aggregation routers via a metro WDM system, typically not from the same vendor. So now we don’t have just two controllers wanting to change router state but 3 or 4... 
We believe that the right solution is simple: all changes in the router must be done via a single owner to the router state and this owner is the IP controller. This is aligned with the hierarchical SDN architecture and standards and allows for clean roles & responsibilities for the different controllers. Allowing an optical controller to be this single owner will block the evolution of RON – and this evolution holds much more value than mere transponder replacement.
