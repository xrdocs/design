---
published: true
date: '2017-07-31 10:00 -0600'
title: The Future of Highly Available Networks
author: scadora
excerpt: Describes current challenges and future directions for network availability.
tags:
  - iosxr
  - HA
  - ISSU
position: hidden
---
## Introduction

Nobody will dispute the importance of availability in today's service provider networks.  What is less obvious is how you achieve it. A network is a complex, dynamic system that must continually adapt to changing conditions. Some changes are normal, necessary and planned (e.g. software and hardware upgrades, configuration changes), while others are unplanned and unpredictable (e.g. software or hardware faults, human error).  This whitepaper discusses different approaches to availability in both cases and lays out current best practices which can be distilled into a few simple themes:

- Solve for the Network First

- Reduce Complexity

- Automate Operations


## Defining Availability

Before diving into the mechanics of availability, it’s worth considering what a highly available network means _to you_.  Tolerance for failure is driven by your SLAs.  Not every service requires the same kind of availability as high frequency trading or systems supporting hospitals.  So take some time to understand the availability requirements for your services across your network.  Many people assume that higher availability is always better.  This may result in over-engineering the network and introducing unneeded complexity and cost.

## Approaches to Availability

When planning for availability, network architects often consider two levels of availability strategies: device level and network level.  Getting the right balance between these levels is key to service availability in your network.

### Network Level

The idea of building a reliable network from unreliable components is as old as the Internet itself. Originally designed for military survivability, the Internet assumes that nodes and links will fail.  Under such conditions, you can still deliver network availability through resilient protocols and well-designed architectures.  

On the protocol side, availability is improved by reducing both the Mean Time To Detection (MTTD) and the Mean Time To Repair (MTTR) of protocol failures.   For reducing MTTD, Bidirectional Forwarding Detection (BFD) and/or Ethernet OAM (802.3ah) are your best friends.  BFD operates at Layer 3 and EOAM at Layer 2, but both provide fast-failure detection that allows network protocols to begin convergence. 

Reducing MTTR involves multiple approaches, starting with optimizing protocol convergence after the failure has been detected.  Incremental SPF (iSPF) has long been used in IGPs to reduce the time it takes to recompute the best path.  Convergence can also be improved by installing a precomputed backup path in the routing table using BGP Protocol Independent Convergence (PIC) and Loop Free Alternative Fast ReRoute (LFA FRR).

MPLS Traffic Engineering (TE) can also help reduce MTTR by giving you the ability to use links and nodes that are not necessarily in the shortest path.  By increasing the pool of available resources, TE helps ensure that the loss of one link or node results in the loss of only a small amount of total capacity.  When links or nodes fail, MPLS TE Fast Reroute (MPLS TE FRR) can locally repair LSPs while the headend re-establishes the end-to-end LSP with an impressive 50 millisecond failover time. Still, it’s also worth remembering that FRR might be overkill for some services: not all SLAs actually require 50 millisecond failover.  

For these convergence optimizations and fast reroute technologies to work, the underlying architecture must support them.  Multiple paths are essential to delivering fault tolerance.  Well-designed architectures use redundant uplinks and multi-homing to avoid single points of failure.  In such architectures, fast failure detection and fast convergence optimizations together provide a good balance of minimizing packet loss while re-converging the control plane at a pace that doesn’t destabilize the network.

One of the main drawbacks with network-level availability mechanisms is that you either have to overprovision your network or accept that availability may be degraded during a failure.  After all, if a link or node fails and your backup paths don’t have enough capacity, you will drop traffic.  Again, it’s worth considering your SLAs.  Some temporary degradation may be acceptable and you could use QoS to ensure that lower priority traffic is dropped first, allowing you to provision just enough redundant capacity for high priority traffic in failure conditions.   In any event, be sure to weigh the higher capex costs of an overbuilt network against the lower operating costs of simpler hardware, software and network designs.  Many operators have found that the opex savings ultimately far outweighs the capex cost of network-level availability.

### Device Level 

Network-level availability mechanisms are great for coping with unreliable components but, over the years, we’ve also put a lot of work into making those individual networking devices more reliable, too. In the industry parlance, device-level availability often focuses on reducing the Mean Time Between Failure (MTBF) of the router and its components.  Following the paradigm of traditional hardware fault tolerance, device-level HA duplicates major components in the system (RPs, power supplies, fan trays) for 1+1 redundancy.  The duplicate components may load share in an Active-Active configuration (e.g. redundant power shelves in the CRS) or run in an Active-Standby configuration (e.g. Active-Standby RPs).

As reassuring as backup power supplies and fans may be, straight-forward hardware redundancy doesn’t cut it for complex components with significant software elements.  Take the route processor (RP), a complex bundle of hardware and software that is responsible for running the control plane and programming the data plane.  In the event of a failover, a standby RP cannot assume the duties of the active RP without either 1) re-building the state (re-establishing neighbor relationships, re-building routing tables, etc) or 2) having an exact, real-time copy of the active state.  The first takes time and the second has proven difficult to achieve in practice.

One way to buy time is to separate the control plane and data plane by allowing the data plane to continue to forward traffic using the existing FIB even when the control plane on the RP is unavailable.  At Cisco, we call this Non-Stop Forwarding (NSF).  Modifications to higher-level protocols (BGP, ISIS, OSPF, LDP) allow a router to alert neighbors that a restart is in progress (“Graceful Restart”).  The NSF-aware neighbors will continue to maintain neighbor relationships and forwarding entries while the RP reboots and/or the standby RP transitions to active.  Stale FIB entries may cause sub-optimal routing or even black holes, but the effect is temporary and usually tolerable.  NSF may also impose additional CPU and memory requirements which increase the complexity and cost of the device.  Nevertheless, over years of industry hardening, NSF has matured into an effective technique for minimizing network downtime.

Instead of buying time with NSF and Graceful Restart, IOS XR also supports Non-Stop-Routing (NSR).  With NSR, all the protocol state required to maintain peering state is precisely synchronized across the active and standby RPs.  When the active fails over, the standby can immediately take over the peering sessions.  Because the failure is handled internally, it is hidden from the outside world.  In practice, NSR is a very complex, resource-intensive operation that doesn’t always result in the perfect state synchronization that is required.  And precisely because NSR “hides” the failure of the RP from neighbors, troubleshooting can be very difficult if something goes wrong.

Building on NSF and NSR, Cisco tackled the specific problem of planned outages by developing an upgrade process called In Service Software Upgrades (ISSU).  ISSU is a complex process that coordinates the standard RP failover with various other mechanisms to ensure that the line card stops forwarding for only as much time as it takes to re-program the hardware.  Under ideal conditions, the outage is less than 10 seconds.  However, in real networks, conditions are almost never ideal.  Like NSR, ISSU has proven difficult to achieve in practice _for the entire industry_.  Even when an in-service upgrade is possible, the operational overhead of understanding and troubleshooting ISSU’s many stages and caveats often outweighs the value of keeping the node in service during the upgrade.  

#### The Complexity Problem

In __Normal Accidents__, Charles Perrow introduced the now widely-accept idea that systems with interactive complexity and tight coupling are at higher risk for accidents and outages. It simply isn’t possible for engineers to imagine, anticipate, plan for and prevent every possible interaction in the system.  Moreover, system designers have learned the hard way that, in practice, using redundancy to compensate for local failures often has the effect of increasing complexity which, in turn, causes the outage you were trying to avoid!  Given that a router with two-way active-standby redundancy is a complex, tightly coupled system, it is perhaps inevitable that successfully and reliably executing NSR and ISSU at service provider speeds and scale has proven challenging industry-wide.


## New Heuristics

As daunting as availability can be, the solutions are relatively straightforward.  You don’t need a lot of new features and functionality, but you may need to rethink your operations and architecture.

### Solve for Network Availability First

Given the unavoidable cost and complexity of device-level availability, it makes sense to focus your availability strategy on network-level availability mechanisms.  Fast detection, convergence and re-route in a redundant, multi-path topology will get you the most bang for the buck.

### Minimize Impact with Scale-Out Architectures

If you’re looking at your next-generation network architecture, it’s worth considering emerging design patterns that can significantly improve availability.  Traditional, hierarchical network designs typically include an aggregation layer, where a small number of large devices with a large number of ports aggregate traffic from southbound layers in preparation for transit northbound.  These aggregation devices are commonly deployed in a 1+1 redundancy topology.

From an availability perspective, the weak point is those aggregation boxes.  If one of those boxes go down, you’ll lose half your network capacity.  That’s a large blast radius.   Network-level availability mechanisms like fast-reroute and QoS may mitigate the impact to high priority services, but unless you have vast amount of excess capacity, your network will run in a degraded state.  Hence, those devices are prime candidates for dual RPs with NSF and NSR.  But we’ve already seen that those strategies can introduce complexity and, consequently, reduce availability.

Faced with new traffic patterns and scale requirements, pioneers in massively-scaled data center design developed a new design pattern that continues to find new applications outside the data center [1].  The spine-and-leaf topology replaces the large boxes in the aggregation layer with many smaller leafs and spines that can be scaled horizontally. Because the traffic is spread across more, smaller boxes in a spine-leaf topology, the loss of any single device has a much smaller blast radius.  Cisco’s NCS 5500 product line is well aligned with this type of CLOS-based fabric design as it moves to the core and beyond.

### Manage Scale With Automation

The sheer numbers in fabric-based network architectures can be intimidating.  In the past, we’ve often assumed that complexity (and therefore, failure) increases with the number of devices.  But as the design of large-scale data centers have shown, you can easily manage vast numbers of devices if you have sufficiently hardened automation.  On the IOS-XR side, we are committed to [Model-Driven Programmability](https://xrdocs.github.io/programmability/blogs/2016-09-12-model-driven-programmability/) to enable complete automation to make the network full programmable through tools like Ansible and Cisco’s Network Service Orchestrator (NSO). 

### Understand Your Failures

Knowing what’s failed in the past is essential to avoiding that failure in the future.  The world’s largest web service providers routinely perform forensic analyses of past failures in order to improve design and operations [2].  It is also possible to automate the remediation of well-understood failures [3].  

### Automate Upgrades

More than one forensic analysis has shown that 60 – 90% of failures in the network are caused by having a human being in the loop: fat-fingering the configuration, killing the wrong process, applying the wrong software patch.  Maintenance operations are responsible for twice the number of failures as bugs.   Upgrades in particular are a magnet for these kinds of failure, as they are often complex, manual and multi-stage. 

When manual intervention is the cause of the problem, automation provides a way forward.  By automating and vigorously validating upgrade procedures, you can significantly improve device availability while reducing operational overhead.  This is the motivation for tools like the [Cisco Software Manager](https://www.youtube.com/watch?v=isxN08x-mr4) which has been proven to reduce errors and improve availability.  

### Simplify Upgrades 

We can’t just stop at automating the upgrade process.  Automating complex processes removes the human element, but as long as the complexity remains, so does the risk of “normal accidents.”  After all, in the worst case, automation just provides you a way to do stupid things faster! 
To get the most out of automation, the upgrade process itself needs to be simplified.  The current state of the art for software installs and upgrades is the standard Linux model of package management.  Starting with IOS XR 6.0, all [IOS XR packages use the RPM Package Manager](https://xrdocs.github.io/software-management/tutorials/2016-08-06-introduction-to-rpm/) (RPM) format, the first step in our upgrade simplification journey.

### Design Simple Failures

The truth is that device failure is normal at scale.  Even if an individual device has the vaunted 5 9s availability, if you have 10,000 of those devices, you will have downtime every day of the year.  Instead of trying to avoid failures at all costs, embrace it!  Just make sure you embrace the right kind of failure.  Experience has taught us that simple, obvious failures with predictable consequences are easier to manage than complex, subtle failures with poorly understood consequences _even when_ the simple failure has a larger “blast radius.” 

Take the case of a stateful switchover from the active to the standby RP.  There aren’t many network operators who haven’t been scarred by a planned or unplanned switchover that did not go as expected.  For whatever reason, the standby RP ends up in state that does not match the active RP’s state when it went down.  Because device-level availability techniques like NSR try to “hide” the failure from the outside world, it can be very difficult to diagnose and troubleshoot the underlying issue.   In many cases, the partial failure of the switchover is often worse than just having the router go away and come back.  A rebooting router is a well-understood event that can be handled by the control plane protocols.  On the other hand, a misbehaving RP in an unknown state is not well understood and may lead to much worse behavior than a temporary degradation.  

The RP example illustrates an emerging design principle: instead of trying to handle a large set of potential partial failures (e.g. different kinds of failed switchovers), group them into a common failure (router reset) that can be handled in a predictable way by the network.  The end result is a network that is simpler, more predictable and more reliable.

Following this principle, many service providers have settled on simpler switchover techniques like NSF over the more complex stateful switchover of NSR.  Going a step further, some providers have deployed single-RP systems in multi-homed roles, forgoing the upside of switchover altogether in favor of a single, well-understood failure.  Of course, single-RP systems cost less, but this is absolutely not about capex.  Some customers will actually buy the redundant RP and leave it, powered off, in the chassis so they don’t have to roll a truck in the event of a truly catastrophic RP failure.  The cost of the extra RP is trivial compared to the operational cost of detecting and fixing bad failovers.

### Drain Instead of Switchover

If you have a single RP system, you won’t be doing a switchover for maintenance operations.  Operators who have pioneered this kind of deployment have developed a “drain-and-maintain” strategy.  In this workflow, devices targeted for traffic-impacting maintenance will be drained of traffic in a controlled manner by shutting down links, lowering the preference of the link, or assigning an infinite cost to the link so routing neighbors will not select it.  Once the traffic has been redirected away from the router, the maintenance operation (such as software upgrade) can proceed.  When the maintenance is complete, links can be brought back into service.  This works well for redundant, transport-only nodes like LSRs and P routers.

For drain-and-maintain to be successful, you have to first validate that there is sufficient excess capacity to carry the drained traffic while the device is offline.  Because of all the moving parts, automation is a key element of this strategy.  You will want an orchestration system to validate the excess capacity, choreograph the drain, maintenance, and undrain, and validate the return to the desired steady-state.

### Return to A Known State

If a router reboot is to be a “normal failure” in the network, the system needs to ensure that the router returns to a known state as quickly as possible.  For this to work, the router’s “source of truth” (i.e. its configuration) needs to be stored off the box.  If the source of truth is on the router, the truth will be irretrievably lost if the router is wiped out.  With an off-box source of truth, you can [iPXE-boot your router](https://xrdocs.github.io/software-management/tutorials/2016-07-27-ipxe-deep-dive/) back to the known state with confidence.

### Protect Single Points of Failure

Redundant, multi-homed topologies are the hallmarks of good network design, but it is not always possible to design out all the single points point of failure. Some common examples include:

- Edge devices such as an LER or PE router may have single-attached customers.
- End-users are typically single-homed to an edge device that may contain significant amounts of non-duplicated user state (e.g. BNG or CMTS).  
- Long-haul transport can be prohibitively expensive, increasing the likelihood that of an architecture that leverages single devices with a limited number of links.

The first question to ask yourself is if there is any way to design redundancy into the network using new technologies.  For example, IOS XR supports Network Function Virtualization (NFV), allowing it to be deployed as a virtual PE (vPE). Deployed in a redundant pair, vPEs can provide edge customers with better network availability over the same physical link. 

If you can’t eliminate it altogether, a single point of failure is also a good candidate for the spine-and-leaf fabric described above.  If a 2 RU node in the fabric fails, the blast radius is much smaller than if a 20-slot chassis fails.  Barring that, in-chassis hardware redundancy and switchover techniques like NSF may be considered.  

## Conclusion

Decades of experience have proven that network availability mechanisms provide simpler, more efficient, lower cost alternatives when the architecture supports them.  Beyond that, the frontier for availability is all about operations.  By automating workflows, particularly upgrades, you can eliminate the most common causes of failure in the first place.  Investing in operations may seem like an odd strategy for availability but the truth is that simple, automated networks are the most available networks of all.

## References And Further Reading

[1] [Introducing Data Center Fabric](https://code.facebook.com/posts/360346274145943/introducing-data-center-fabric-the-next-generation-facebook-data-center-network/)

[2] [Evolve or Die - High-Availability Design Principles Drawn from Google’s Network Infrastructure](https://research.google.com/pubs/pub45623.html)

[3] [Making Facebook Self-Healing](https://code.facebook.com/posts/156810174519680/making-facebook-self-healing)

Normal Accidents: Living with High Risk Technologies (Updated). Princeton University Press, 1999, Charles Perrow.

Site Reliability Engineering, O'Reilly Media, April 2016, Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy.

