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

## Service Assurance
While passive metrics like interface statistics or sflow records can be very helpful at determining the health of the network, they can't actually assure a service.  The only way to know if the service actually meets the SLA is to measure traffic sent through the service’s data path.  This most common way to do this is by injecting traffic to probe the network.  This process is commonly referred to as "active monitoring."  There are different kinds of probes that can be generated from different devices at different parts of the network, but active monitoring involves sending, receiving and tracking traffic to directly measure the end customer’s experience of the service.   