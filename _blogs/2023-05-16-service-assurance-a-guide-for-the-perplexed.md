---
published: false
date: '2023-05-16 09:31 -0600'
title: 'Service Assurance: A Guide For the Perplexed'
author: scadora
---
## A New Post


## What's a Service, Anyway?
As a network engineer, I am often guilty of thinking of "services" in terms of the technologies that enable them: L3VPN, L2VPN, EVPN, BGP, Segment-Routing, Traffic Engineering, etc.  But if you take a step back, the only reason to invest in these cool technologies is because they deliver value to end customers. Those customers don’t buy EVPN or BGP; they buy point-to-point or multi-point services (along with enhancements like security and management) to meet specific business needs. Ultimately, it's the end customer’s “digital experience” that determines the success or failure of the service.  As one service provider CTO put it, “The measure of success is if the customer gets the level of service they want. It sounds so simple, but it’s so difficult.” 

## Why Bother With Assurance?

A service has a contract, a price, start and end dates, and a Service Level Agreement (SLA).  The SLA defines the expected quality level of the service and can include metrics around availability, bandwidth, latency, loss and jitter. 

SPs are highly motivated to assure that the service meets the SLA for several reasons: 

- Lost Revenue: If the SP cannot deliver the terms of the contract, then the end customer is entitled to a refund.  In the case of outages and degradations, the SP must be able to prove that the failure did not occur in the SP network or else pay up. 

- Service Differentiation:  SPs are always looking for ways to differentiate with new and improved services.  Increasingly, monitoring and reporting are considered part of a differentiated service.  One SP called this “Service Assurance as a Service” (SAaaS).  For example, if you are providing a low latency service, you must be able to prove that the service meets delay guarantees.  Service assurance is becoming monetizable. 

- Customer-driven Measurement: The end customer has the most immediate experience of the quality of the service.  After all, it’s their data that’s traversing the service, and their applications that are directly impacted by service quality.  Increasingly, Enterprises are investing in tools that can deliver very detailed metrics about the services they pay for.  These tools include hardware-based probes from Netscout or Accedian, monitoring solutions like Thousand Eyes, or even the integrated tools in SD-WAN solutions like Viptela.  These tools are often more accurate than anything the SP measures, leading to a situation where 61% of customer outages and degradation issues are reported by the end customers, not the SPs.  

- Customer Experience and Retention: One survey showed that 90% of customers only contact their SPs when they are ready to end their contracts due to service disruptions.  Because service assurance data can give an objective measurement of customer experience, it can be used to understand and improve customer retention. 

- Reduced Customer Support Calls: One SP estimated that well over 50% of customer support calls concerned outages that did not involve the SP’s network or past outages that had been already fixed.  By collecting service assurance data and providing it to their customers, SPs could offload those support calls to a self-service portal, conserving valuable technical support resources. 

- Streamline Troubleshooting: Swamped by barrage of network events and alarms, network operators are challenged to understand the root cause of reported service disruptions as well as the service impact of network faults.  Good service assurance data can help localize failures and identify root causes more quickly. 

## Getting Specific
Service assurance is an elastic term that means different things to different people.  So I want to lay out a specific definition of service assurance as a function.   

Assurance typically refers to some combination of performance monitoring (PM) and fault management (FM).  Assurance addresses different questions depending on the layer it applies to.   

Network Assurance seeks to answer the question of whether the network is operating as designed: Are interfaces up? Are protocols running?  Are CPU, memory and bandwidth utilizations within acceptable ranges? 

Customer Assurance (also called Digital Experience Monitoring2) includes Application Performance Monitoring.  It measures things like page load times, voice & video quality and application responsiveness.  This data could be provided by the application itself (e.g. WebEx metrics) or measured by a third party tool (e.g. ThousandEyes, AppDynamics, Accedian). 

Service Assurance sits between Network Assurance and Customer Assurance.  The specific goal of Service Assurance is to ensure that the service is delivering the contracted SLAs.  These include: 

- Bandwidth 
- Availability 
- Latency 
- Jitter 
- Loss 
- Path 

## The Holy Grail: Assured Services

From a functional perspective, services and service assurance seem distinct, with unique configurations, protocols and lifecycles.  However, from an operator’s perspective, a service that cannot be assured might not be sellable.  Instead of providing service assurance sometime after a service has been deployed, the goal is to deliver an “assured service.”  An assured service has the following characteristics: 

- Service assurance functionality should be delivered at the same time as the service itself.   
- Service assurance scale should match service scale.
- The service and service assurance should be orchestrated together and configured at the same time. 
- Service assurance data should be easy to correlate to a specific service. 

Outside of OTN-based services (which carry performance, fault and path data in the header of every frame), most Enterprise services today do not fit the definition of an “assured service.”  Services are deployed first, assured later (sometimes partially and if at all).

## What’s A Service Provider To Do? 

Service assurance can be a complex undertaking.  One option is to do nothing: build a solid network with plenty of redundancy, monitor for network faults (“Network Assurance”) and have faith that the architecture can deliver the needed SLAs.  This “best-effort” approach to service assurance has several weaknesses: 

- If an SP can’t measure the latency or jitter of a service, they can’t sell more expensive low-latency or jitter-bound services.  For lack of assurance, money is left on the table. 

- The end customer knows more about the quality of their service and can detect impairments long before the SP is even aware of a problem. 

- Troubleshooting a faulty service requires a lot of legwork on the part of smart (i.e. expensive) network engineers.  When the number of services is small and commands a high price, that isn’t a big problem.  But as Cloud and Video push bandwidth demand ever higher and competing services like SD-WAN push prices lower, that approach to services just doesn’t scale.  

Surmounting these challenges requires a dedicated service assurance function.   

## Active Assurance

While passive metrics like interface statistics or sflow records can be very helpful at determining the health of the network, they can't actually assure a service.  The only way to know if the service actually meets the SLA is to measure traffic sent through the service’s data path.  Since services like L2 and L3VPNs do not define an embedded assurance function, an additional mechanism must be employed to probe the data path of the service.  The most common way to do this is by injecting synthetic traffic to probe the network.  This process is commonly referred to as "active monitoring."  There are different kinds of probes that can be generated from different devices at different parts of the network, but all active monitoring involves sending, receiving and tracking traffic to directly measure the end customer’s experience of the service.   


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

Embedded probes have a unique advantage in that they can test the internals of the network infrastructure. SR-PM, IOS XR's performance measurement toolkit, includes the capability to test link performance as well as end-to-end traffic engineering paths.  This is something external probes simply can't do.

One common argument against embedded probes is performance.  And that was certainly true in the past, when probes were punted to the RP for processing. If the punt path was congested, the probe would report poor performance when the actual datapath was fine. In modern systems, however, hardware timestamping ensures that the NPU stamps the probes in the datapath, giving a much more accurate measurement of network delay.  In addition, many systems can support "hardware offload" which pushes the entire process of probe generation into the NPU, giving you a high fidelity measurement of the actual datapath and much higher performance than was possible in the past.  So if you looked at IP-SLA a decade ago and dismissed it because of performance, you should take another look at modern implementations.

Another consideration is interoperability.  If you're using embedded probes in a multi-vendor network, then you have to ensure interoperability between the vendors' implementations of the protocol.  This was especially painful with TWAMP-Light, since the lack of specificity in the RFC made it easy to interpret differently.  This should get better as STAMP becomes the industry standard.

Functionality is the final consideration for embedded probes. The limited memory and compute on a router means that more elaborate customer experience tests like page download times are really not well-suited.  Moreover, your upgrade cycle for assurance features is tied to the upgrade cycle of the entire router which can easily be multiple years.  That's a long time to wait for a new assurance feature.

In sum, embedded probes offer an inexpensive way to get simple, scalable measurements of services, traffic engineered-paths and physical links with excellent fidelity to the actual data path and better performance than ever before. But if interoperability is a problem or you need more complex and/or end-to-end tests, then you may have to consider an external probing system.

### External
External probing devices come in all shapes and sizes, from Network Interface Devices (NIDs) to pluggable SFPs to containers running in generic compute.  They can be deployed at any place in the network that a service provider has a presence, including the end customer site (if the SP has deployed a managed service). Most external probes support all of the assurance protocols we discussed earlier, including of the service activation tests.

External probes can be deployed in-line which measures the service exactly as the end customer experiences it. This is very accurate but also very expensive, as you need one device for every service.  Other deployment models place the NID or SFP at a place in the network where probes can be injected into multiple service paths (e.g. on a trunk port with many VLANs associated with many different VRFs).

Because external probes represent dedicated -> best functionality.

Vendors add value with analytics platforms.

Downside -- capex and opex.





