---
published: false
date: '2023-05-16 09:31 -0600'
title: 'Service Assurance: A Guide For the Perplexed'
author: scadora
---
## Introduction

The concept of IP assurance is not quite as old as IP, but it's close: ARAPNET deployed IPv4 in January of 1983; by December, Mike Muuss had developed the ping utility to test reachability and delay. Techniques for assuring the network's forwarding functions through direct performance measurement have been evolving ever since, resulting in a confusing array of tools, protocols and choices.  Many network architects are left wondering: "What should I deploy for assurance, where, and why?"

These are big questions, so I'm going to tackle them across a series of blog posts.  This first one will cover the what and why of assurance.  In part two, I'll cover the protocols and tools you have to work with.  And in part three, I'll look at some key design questions.

## What Do I Mean By Service Assurance?

Assurance is the process of measuring performance and managing faults with the ultimate goal of optimization  You can "assure" many things -- a router, a path, a service, or an end-to-end digital experience. Assurance addresses different questions depending on the layer it applies to, so I like to divide it into three categories:

![AssuranceTaxonomy2.png]({{site.baseurl}}/images/AssuranceTaxonomy2.png)


1. Network Assurance seeks to answer the question of whether the network devices are operating as designed: Are interfaces up? Are protocols running?  Are CPU, memory and bandwidth utilizations within acceptable ranges? 

2. Customer Assurance (also called Digital Experience Monitoring) includes Application Performance Monitoring.  It measures things like page load times, voice & video quality and application responsiveness.  This data could be provided by the application itself (e.g. WebEx metrics) or measured by a third party tool (e.g. ThousandEyes, AppDynamics). 

3. Service Assurance sits between Network Assurance and Customer Assurance.  A Service Provider is responsible for more than just making sure that routers are up and protocols are running (Network Assurance) but not usually responsible for the full Digital Experience (Customer Assurance) since the provider doesn't own all the applications, compute and other resources involved.  The "service", then, defines the limits of the Service Provider's liability. 

![ProviderLiability3.png]({{site.baseurl}}/images/ProviderLiability3.png)


## The Service Is The Product

As a network engineer, I am often guilty of thinking of "services" in terms of the technologies that enable them: L3VPN, L2VPN, EVPN, BGP, Segment-Routing, Traffic Engineering, etc.  But if you take a step back, the only reason to invest in these cool technologies is because they deliver value to end customers. Those customers don’t buy EVPN or BGP; they buy point-to-point or multi-point services (along with enhancements like security and management) to meet specific business needs. 

A service has a contract, a price, start and end dates, and a Service Level Agreement (SLA).  The SLA defines the expected quality level of the service and can include metrics around availability, bandwidth, latency, loss and jitter. 

SPs are highly motivated to assure that the service meets the SLA for several reasons: 

- **Lost Revenue**: If the SP cannot deliver the terms of the contract, then the end customer is entitled to a refund.  In the case of outages and degradations, the SP must be able to prove that the failure did not occur in the SP network or else pay up. 

- **Service Differentiation**:  SPs are always looking for ways to differentiate with new and improved services.  Increasingly, monitoring and reporting are considered part of a differentiated service.  One SP called this “Service Assurance as a Service” (SAaaS).  For example, if you are providing a low latency service, you must be able to prove that the service meets delay guarantees.  Service assurance is becoming monetizable. 

- **Customer-driven Measurement**: The end customer has the most immediate experience of the quality of the service.  After all, it’s their data that’s traversing the service, and their applications that are directly impacted by service quality.  Increasingly, Enterprises are investing in tools that can deliver very detailed metrics about the services they pay for.  These tools include hardware-based probes from Netscout or Accedian, monitoring solutions like Thousand Eyes, or even the integrated tools in SD-WAN solutions like Viptela.  These tools are often more accurate than anything the SP measures, leading to a situation where more than half of customer outages and degradation issues are reported by the end customers, not the SPs.  

- **Customer Experience and Retention**: One survey showed that 90% of customers only contact their SPs when they are ready to end their contracts due to service disruptions.  Because service assurance data can give an objective measurement of customer experience, it can be used to understand and improve customer retention. 

- **Reduced Customer Support Calls**: One SP estimated that well over 50% of customer support calls concerned outages that did not involve the SP’s network or past outages that had been already fixed.  By collecting service assurance data and providing it to their customers, SPs could offload those support calls to a self-service portal, conserving valuable technical support resources. 

- **Streamline Troubleshooting**: Swamped by barrage of network events and alarms, network operators are challenged to understand the root cause of reported service disruptions as well as the service impact of network faults.  Good service assurance data can help localize failures and identify root causes more quickly. 

## What’s A Service Provider To Do? 

Service assurance can be a complex undertaking.  One option is to do nothing: build a solid network with plenty of redundancy, monitor for network faults (“Network Assurance”) and have faith that the architecture can deliver the needed SLAs.  This “best-effort” approach to service assurance has several weaknesses: 

- If an SP can’t measure the latency or jitter of a service, they can’t sell more expensive low-latency or jitter-bound services.  For lack of assurance, money is left on the table. 

- The end customer knows more about the quality of their service and can detect impairments long before the SP is even aware of a problem. 

- Troubleshooting a faulty service requires a lot of legwork on the part of smart (i.e. expensive) network engineers.  When the number of services is small and commands a high price, that isn’t a big problem.  But as Cloud and Video push bandwidth demand ever higher, that approach to services just doesn’t scale.  

Surmounting these challenges requires a dedicated service assurance function.   

## Active Assurance

While passive metrics like interface statistics or sflow records can be very helpful at determining the health of the network ("Network Assurance"), they can't actually assure a service.  The only way to know if the service actually meets the SLA is to measure traffic sent through the service’s data path.  Since services like L2 and L3VPNs do not define an embedded assurance function, an additional mechanism must be employed to probe the data path of the service.  

The most common way to do this is by injecting synthetic traffic to probe the network.  This process is commonly referred to as "active monitoring."  There are different kinds of probes that can be generated from different devices at different parts of the network, but all active monitoring involves sending, receiving and tracking traffic to directly measure the end customer’s experience of the service.   

## Conclusion
Unlike optical services (which carry performance, fault and path data in the header of every frame), IP/MPLS-based services like L2 and L3VPNs do not define an embedded assurance function.  Nevertheless, end customers expect their SLAs to be delivered as promised. Assuring service performance through some form of active monitoring is increasingly a must-have for Service Providers.

In part two of this series, I'll take a deep dive into protocols and tools for active assurance. 
