---
published: false
date: '2023-05-16 09:31 -0600'
title: 'Service Assurance: A Guide For the Perplexed'
author: scadora
---
## Introduction

The concept of IP assurance is not quite as old as IP, but it's close: ARAPNET deployed IPv4 in January of 1983; by December, Mike Muuss had developed the ping utility to test reachability and delay. Techniques for assuring the network's forwarding functions through direct performance measurement have been evolving ever since, resulting in a confusing array of tools, protocols and choices.  Many network architects are left wondering: "What should I deploy for assurance, where, and why?"

These are big questions, so I'm going to tackle them across a series of blog posts.  This first one will cover the what and why of assurance.  In part two, I'll cover the protocols and tools you have to work with.  And in part three, I'll look at some key design questions.

## What Do I Mean By Assurance?

Assurance is the process of measuring performance and managing faults with the ultimate goal of optimization  You can "assure" many things -- a router, a path, a service, or an end-to-end digital experience. Assurance addresses different questions depending on the layer it applies to, so I think about three categories of assurance:

1. Network Assurance seeks to answer the question of whether the network devices are operating as designed: Are interfaces up? Are protocols running?  Are CPU, memory and bandwidth utilizations within acceptable ranges? 

2. Customer Assurance (also called Digital Experience Monitoring) includes Application Performance Monitoring.  It measures things like page load times, voice & video quality and application responsiveness.  This data could be provided by the application itself (e.g. WebEx metrics) or measured by a third party tool (e.g. ThousandEyes, AppDynamics). 

3. Service Assurance sits between Network Assurance and Customer Assurance.  A Service Provider is responsible for more than just making sure that routers are up and protocols are running (Network Assurance) but the provider isn't typically responsible for the full Digital Experience (Customer Assurance) since it doesn't own all the applications, compute and other resources involved.  The "service", then, defines the limits of the Service Provider's liability. 

## The Service Is The Product

As a network engineer, I am often guilty of thinking of "services" in terms of the technologies that enable them: L3VPN, L2VPN, EVPN, BGP, Segment-Routing, Traffic Engineering, etc.  But if you take a step back, the only reason to invest in these cool technologies is because they deliver value to end customers. Those customers don’t buy EVPN or BGP; they buy point-to-point or multi-point services (along with enhancements like security and management) to meet specific business needs. 

A service has a contract, a price, start and end dates, and a Service Level Agreement (SLA).  The SLA defines the expected quality level of the service and can include metrics around availability, bandwidth, latency, loss and jitter. 

SPs are highly motivated to assure that the service meets the SLA for several reasons: 

- Lost Revenue: If the SP cannot deliver the terms of the contract, then the end customer is entitled to a refund.  In the case of outages and degradations, the SP must be able to prove that the failure did not occur in the SP network or else pay up. 

- Service Differentiation:  SPs are always looking for ways to differentiate with new and improved services.  Increasingly, monitoring and reporting are considered part of a differentiated service.  One SP called this “Service Assurance as a Service” (SAaaS).  For example, if you are providing a low latency service, you must be able to prove that the service meets delay guarantees.  Service assurance is becoming monetizable. 

- Customer-driven Measurement: The end customer has the most immediate experience of the quality of the service.  After all, it’s their data that’s traversing the service, and their applications that are directly impacted by service quality.  Increasingly, Enterprises are investing in tools that can deliver very detailed metrics about the services they pay for.  These tools include hardware-based probes from Netscout or Accedian, monitoring solutions like Thousand Eyes, or even the integrated tools in SD-WAN solutions like Viptela.  These tools are often more accurate than anything the SP measures, leading to a situation where more than half of customer outages and degradation issues are reported by the end customers, not the SPs.  

- Customer Experience and Retention: One survey showed that 90% of customers only contact their SPs when they are ready to end their contracts due to service disruptions.  Because service assurance data can give an objective measurement of customer experience, it can be used to understand and improve customer retention. 

- Reduced Customer Support Calls: One SP estimated that well over 50% of customer support calls concerned outages that did not involve the SP’s network or past outages that had been already fixed.  By collecting service assurance data and providing it to their customers, SPs could offload those support calls to a self-service portal, conserving valuable technical support resources. 

- Streamline Troubleshooting: Swamped by barrage of network events and alarms, network operators are challenged to understand the root cause of reported service disruptions as well as the service impact of network faults.  Good service assurance data can help localize failures and identify root causes more quickly. 

## What’s A Service Provider To Do? 

Service assurance can be a complex undertaking.  One option is to do nothing: build a solid network with plenty of redundancy, monitor for network faults (“Network Assurance”) and have faith that the architecture can deliver the needed SLAs.  This “best-effort” approach to service assurance has several weaknesses: 

- If an SP can’t measure the latency or jitter of a service, they can’t sell more expensive low-latency or jitter-bound services.  For lack of assurance, money is left on the table. 

- The end customer knows more about the quality of their service and can detect impairments long before the SP is even aware of a problem. 

- Troubleshooting a faulty service requires a lot of legwork on the part of smart (i.e. expensive) network engineers.  When the number of services is small and commands a high price, that isn’t a big problem.  But as Cloud and Video push bandwidth demand ever higher, that approach to services just doesn’t scale.  

Surmounting these challenges requires a dedicated service assurance function.   

## Active Assurance

While passive metrics like interface statistics or sflow records can be very helpful at determining the health of the network ("Network Assurance"), they can't actually assure a service.  The only way to know if the service actually meets the SLA is to measure traffic sent through the service’s data path.  Since services like L2 and L3VPNs do not define an embedded assurance function, an additional mechanism must be employed to probe the data path of the service.  

The most common way to do this is by injecting synthetic traffic to probe the network.  This process is commonly referred to as "active monitoring."  There are different kinds of probes that can be generated from different devices at different parts of the network, but all active monitoring involves sending, receiving and tracking traffic to directly measure the end customer’s experience of the service.   

In the next part of this series, I'll take a deep dive into protocols and tools for active assurance. 

### Active Assurance at Layer 3
The original form of active assurance were our old friends, ping and traceroute.

To enhance these stalwart tools, Cisco developed a set of performance measurement features collectively referred to as "IP-SLA."  IP-SLA enabled measurements of loss, delay and jitter to IP endpoints using a variety of operations, including ICMP, UDP, HTTP and more. IP-SLA is a form of active assurance that sends and receives probes from the router itself.

The obvious usefulness of performance measurement made it an excellent candidate for standardization.  In 2006, the IETF standardized One-Way Active Measurement Protocol (OWAMP). OWAMP provided the precision of one-way measurements but the requirement for network-wide clock synchronization limited its adoption.  

In 2008, [RFC 5357](https://datatracker.ietf.org/doc/html/rfc5357) introduced Two-Way Active Measurement Protocol (TWAMP) to extend OWAMP to allow for two-way and round-trip measurements (with or without clock synchronization). Because TWAMP defined multiple logical roles for session establishment, most vendors ended up implementing a simpler architecture, "TWAMP Light", that only needed a Sender and Responder.  

Unfortunately, TWAMP Light was only an appendix to RFC 5357 and not suffiently specified to prevent interoperability issues.  Hence we have [RFC 8762](https://datatracker.ietf.org/doc/html/rfc8762), Simple Two Way Active Measurement Protocol (STAMP), which codified and extended TWAMP Light. STAMP is extensible and backwards compatible with TWAMP-Light, so hopefully it will stick around for a while.

Other work is ongoing in the IETF to standardize STAMP extensions in order to leverage the forwarding capabilities of Segment Routing networks (sometimes called Segment Routing Performance Measurement or [SR-PM](https://datatracker.ietf.org/doc/draft-ietf-ippm-stamp-srpm/)).  Inside Cisco, "SR-PM" is sometimes used as a shorthand term for the superset of all L3 performance management features (SR and IP).  So if you're interested in TWAMP-Light and a Cisco person is talking to you about SR-PM, don't worry, you're both talking about the same thing.

### Active Assurance at Layer 2

A different set of standards governs service assurance in Layer 2 networks (think L2VPN, VPLS, EVPN VPWS, etc).  The building blocks of Ethernet Operation, Administration and Management (OAM) began with IEEE 802.1ag, which defined Connectivity Fault Management (CFM).  The ITU came out with its own Ethernet OAM standard, Y.1731. Both 802.1ag and Y.1731 cover fault management, while performance management is solely covered by ITU-T Y.1731. Nowadays, IEEE 802.1ag is considered a subset of ITU-T Y.1731.

The history of multiple standards means, again, that terminology can be confusing.  In Cisco documentation, CFM is used as a generic term for Ethernet OAM, which includes Y.1731.

### Service Activation Testing Methodologies

In addition to the "AMPs" and Y.1731, you may run across a few other measurement standards. 

[Y.1564](https://www.itu.int/rec/T-REC-Y.1564-201602-I/en) is an out-of-service ethernet test methodology that validates the proper configuration and performance of an ethernet service before it is handed over to the end customer.  

[RFC 6349](https://www.ietf.org/rfc/rfc6349.txt) defines a methodology for TCP throughput testing.

These methods should not be confused with service assurance since they are only intended to be used during service activation or active troubleshooting.

## Embedded or External

Active assurance methods like TWAMP and Y.1731 require a sender and receiver/responder.  The functionality can be embedded in an existing router or it can be a dedicated external device.

### Embedded
There are many reasons to take advantage of active probing capabilities built in to your routers.  First of all, it's free!  You've already paid for the device to forward traffic, so if it can also do active assurance, then by all means, try this first.  Operationally, it's also a slam dunk.  Whatever tools you use to manage your router config can also manage probe configuration.  There are no new systems to manage or integrate.

Embedded probes have a unique advantage in that they can test the internals of the network infrastructure. SR-PM, IOS XR's performance measurement toolkit, includes the capability to test link performance as well as end-to-end traffic engineering paths. Emerging measurement techniques like [Path Tracing](https://datatracker.ietf.org/doc/draft-filsfils-spring-path-tracing/) bring ECMP awareness to performance measurement.  These are things external probes simply do.

One common argument against embedded probes is performance.  And that was certainly true in the past, when probes were punted to the RP for processing. If the punt path was congested, the probe would report poor performance when the actual datapath was fine. In modern systems, however, hardware timestamping ensures that the NPU stamps the probes in the datapath, giving a much more accurate measurement of network delay.  In addition, many systems can support "hardware offload" which pushes the entire process of probe generation into the NPU, giving you a high fidelity measurement of the actual datapath and much higher performance than was possible in the past.  So if you looked at IP-SLA a decade ago and dismissed it because of performance, you should take another look at modern implementations.

Another consideration is interoperability.  If you're using embedded probes in a multi-vendor network, then you have to ensure interoperability between the vendors' implementations of the protocol.  This was especially painful with TWAMP-Light, since the lack of specificity in the RFC made it easy to interpret differently.  This should get better as STAMP becomes the industry standard.

Functionality is the final consideration for embedded probes. The limited memory and compute on a router means that more elaborate customer experience tests like page download times are really not well-suited.  Moreover, your upgrade cycle for assurance features is tied to the upgrade cycle of the entire router which can easily be multiple years.  That's a long time to wait for a new assurance feature.

In sum, embedded probes offer an inexpensive way to get simple, scalable measurements of services, traffic engineered and ECMP paths and physical links with excellent fidelity to the actual data path and better performance than ever before. But if interoperability is a problem or you need more complex and/or end-to-end tests, then you may have to consider an external probing system.

### External
External probing devices come in all shapes and sizes, from Network Interface Devices (NIDs) to pluggable SFPs to containerized agents running in generic compute.  They can be deployed at any place in the network that a service provider has a presence, including the end customer site (if the SP has deployed a managed service) and in the cloud. Network vendor interoperability is not an issue since the probes are generated and received by the external probing devices, not the networking devices.

External probes can be deployed in-line which measures the service exactly as the end customer experiences it. This is very accurate but also very expensive, as you need one device for every service.  Other deployment models place the NID, SFP or containerized agent at a place in the network where probes can be injected into multiple service paths (e.g. on a trunk port with many VLANs associated with many different VRFs).
[picture]

Unlike routers, whose primary function is to forward traffic, external probes are dedicated to the sole purpose of analyzing the network. The breadth of functionality they support can be much wider, encompassing Ethernet OAM, TWAMP, and service activation protocols as well as detailed insight into Layer 7 transactions (e.g. HTTP, DNS, TCP, SSL, etc) and high-level path analysis (e.g. using traceroute). Taken together, the information from external probes deployed at the right places in the network can give a good snapshot of the end customer's digital experience.

While external probes give good insight into end-to-end performance all the way up to the application layer, they can't dig into the internals of the service provider network.  The network is a black box to external probes. Things like link performance, path performance, and ECMP paths are essentially invisible to external probes.  

Probably the biggest drawback to external probes is cost, both capex and opex.  Hardware probes, whether NIDs or SFPs, are expensive.  Once service provider reported spending as much on NIDs as on routers in their latest edge deployment!  But operational costs can also be of concern.  Every external probe represents one more network element to manage: hardware has to be deployed and monitored, software has to be upgraded and maintained.  Adding thousands or tens of thousands probes is not a project to be taken lightly.

## Design Considerations

Now that we have a grasp of the protocols and tools available, it's time to look at design.

When designing a service assurance architecture, the notion of "provider liability" is a good place to start.

Place In Network  

Customer Edge. 
If the SP is managing the CE device, then the SP has the option to deploy probes at the customer premise to monitor a specific service.  This has the advantage of assuring the full end-to-end path. With this kind of visibility, an SP could provide a dashboard view of the performance of a particular service that a customer bought. For this to work, CE probes have to be orchestrated, managed, monitored and visualized for each customer site which can be a challenge at scale.  

Provider Edge: If the SP is not managing the CE device or does not want to deal with the scale issues of CE-based probing, probes can be exchanged between PE routers.  This has the advantage of having fewer probe generators to manage but it does lack the full end-to-end insight of a CE to CE probe.

Scale
While sending synthetic traffic to probe the service data path gives an accurate measurement of the service quality, it can be computationally expensive to generate, timestamp, send, receive and analyze the probe packets.  This becomes a significant issue when the number of services begins to scale, especially for embedded router probes which are more resource constrained than external third-party probes.

Multipoint Service Scale
Multipoint services include L3VPNs and ELAN L2VPN services.  To assure the multipoint services, you would need to measure the SLA of every possible connection in that service.  For example, take an L3VPN service. The number of required probes would be equal to the number of connections in the VRF which, for a fully meshed network, is calculated according to the formula (n*(n-1))/2.   If you have a small Enterprise L3VPN service with 5 customer sites, then that’s 10 probes for that service alone. 

But it’s actually worse than that.  Most L3VPN services are sold with multiple classes of service.  So if the customer paid for 25% Gold, 25% Silver and 50% Best Effort traffic, then you’d need a probe for each Class of Service.  So now you’re up to 3 * 10 = 30 probes for one small service.   Compare that to the number of probe packets that IOS XR platforms can actually support.  The NCS platforms support a total of 2000 probes, so clearly that won’t scale.  Purpose built premium edge platforms like the ASR9000 can do many more probes (e.g. 60,000 on the RP with more on the line cards) but even then, you will bump up against probe packet processing limits when deploying tens of thousands of services on a large edge platform.

You can mitigate some of these scale issues by leveraging some of the design principles discussed earlier.  For example, you could offload L3VPN probe generation to shadow routers or third-party probes.  For a managed service, you could offload probes to the CE device.   Or you could measure the transport path performance using Performance Measure (PM) for TE paths and MPLS LSP monitoring for BGP next-hops and use the aggregate measurement as a “best guess” for multiple services along those paths.  

Alternatively, some Service Providers tackle the scale problem by deploying service probes selectively.  Probes are expensive, so Service Providers can design their service offerings accordingly.  If an L3VPN service is going to include premium SLAs that can only be measured by expensive probes, then that expense can and should be included in the prices of the premium service.  In the end, perhaps only a fraction of the L3VPN services offered by a Provider would need regularly monitored SLAs.  Another option would be to probe all services but do it infrequently, e.g. at service activation time and for troubleshooting.

Ultimately, it's the end customer’s “digital experience” that determines the success or failure of the service.  As one service provider CTO put it, “The measure of success is if the customer gets the level of service they want. It sounds so simple, but it’s so difficult.” 

A Service Provider

The specific goal of Service Assurance is to ensure that the service is delivering the contracted SLAs.  These include: 

- Bandwidth 
- Availability 
- Latency 
- Jitter 
- Loss 
- Path 

As we will see, the boundaries between network, service and experience can be blurry.

## The Holy Grail: Assured Services

From a functional perspective, services and service assurance are often distinct, with unique configurations, protocols and lifecycles.  However, from an operator’s perspective, a service that cannot be assured might not be sellable.  Instead of providing service assurance sometime after a service has been deployed, the goal is to deliver an “assured service.”  An assured service has the following characteristics: 

- Service assurance functionality should be delivered at the same time as the service itself.   
- Service assurance scale should match service scale.
- The service and service assurance should be orchestrated together and configured at the same time. 
- Service assurance data should be easy to correlate to a specific service. 

Outside of OTN-based services (which carry performance, fault and path data in the header of every frame), most Enterprise services today do not fit the definition of an “assured service.”  Services are deployed first, assured later (sometimes partially and if at all).
