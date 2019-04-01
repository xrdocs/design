---
published: false
date: '2019-03-29 18:47 -0600'
title: Back to the Future with Fabrics
author: Shelly Cadora
---
## Turning Fabrics Inside Out

Until recently, the notion of a "fabric" in a network was confined to single device: the switching fabric inside an NPU or ASIC, the fabric module of a modular router, or the fabric chassis of a multi-chassis system.  All these internal fabrics fulfilled the same basic function: non-blocking connectivity between the input and output ports of the system. [some reference to Clos here, the basics of the design]

Largely driven by the needs of the massively scalable data center, a new network design has quite literally turned fabrics inside out.  Instead of building fabrics inside silicon, data center fabrics provide (statistically) non-blocking connectivity by connecting simple devices in a densely connected spine and leaf topology.  These designs can be relatively small (e.g. replacing a large modular devices with a fabric of small devices) or as massive as the data centers they enable (e.g. [Facebook's Data Center Fabric](link)).

Insert picture of internal vs. external fabrics.

As every network architect knows, there is no one "best" design for a network.  Every design represents tradeoffs.  Large scale fabrics have been very successful in the data center.  The question is: are fabrics sensible in other parts of the network as well?  Let's look at that from a couple of angles: scale, availability, traffic patterns, and cost.

## Scale: Up and Out

When designing networks, scale is almost always top of mind.  There are two basic paradigms for scale:  scale out and scale up.  Networking has traditionally relied primarily on a scale up paradigm:  when you need to scale, buy a bigger box or more line cards or denser line cards.  Scale out, used almost exclusively in cloud-scale computing, takes the oppositive approach.  Instead of buying bigger boxes, buy lots of smaller smaller ones.

Scale out can scale very large.  The NCS5516, one of the largest 100GE platforms available, today provides 576 100GE ports. An external fabric built entirely of of 48-port NCS 5502s could support up to twice that many ports. If you put NCS5508s in the spine, you could get up to 6912 100GE ports.  That's truly massive scale.  

## Availability and Scale

How you scale has a direct impact on how you achieve high availability in a given design.  When you scale up, those ever-larger and denser devices create an ever-larger "blast radius."  If you could lose a big chunk of your network capacity when a single device goes down, you'll want to harden that device with lots of redundant hardware and complex software to keep it all running.  In the popular [Cattle vs. Pets](http://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/) meme, these highly available devices are definitely pets!  The downside of deploying pets in your network is that these complex, tightly coupled systems can be difficult to upgrade and troubleshoot.    

When you scale out, the blast radius of an individual device is smaller.  The smaller the blast radius of a device, the easier it is to take it out of service for upgrade.  Without the complexities of large, redundantly systems, networks made out of multiple small, simple devices can achieve very high availability.  This is a Cattle mentality:  individual devices don't matter and can removed or replaced without impacting the function of the herd.

Traditionally, networking has had a bias for scale up, but in truth scale out and scale up exist on a continuum.  Even in traditional designs, two routers are usually deployed in a given role for redundancy. Two is the smallest amount of "scale out" possible, but it's still scale out.  Of course, the ultimate in "scale out" is the full spine and leaf fabric design of data center fame, but there are designs that balance "scale out" and "scale up" to achieve the right balance.

## Traffic Patterns: Is An East-West Wind Blowing Your Way?

Massive data center fabrics arose out of the need to provide equidistant, non-blocking bandwidth for traffic between storage and compute components distributed across the data center.  This is what is called an "east-west" traffic pattern, in contrast to the "north-south" pattern that characterizes classis Campus and Service Provider designs.  When north-south traffic predominates, one or two large chassis can provide the requisite aggregation function very efficiently.  

Having a clear understanding of the traffic pattern you need to support is crucial in understanding if fabrics are right for you.  There are certainly places in the network where traffic patterns are changing very rapidly today.  For example, the rise of local peering and caching in Service Provider networks means that traffic that might once have been aggregated and sent across the backbone can now be served locally instead.  That introduces a strong east-west component into what was once an almost entirely north-south pattern.  Introducing a fabric with a spine layer to provide connectivity between PE nodes and Peering or Caching nodes starts to make sense in that scenario.

## Good Fabrics Are Not Free

Some people assume that fabrics will be less expensive than modular systems because fabrics are built from smaller, simpler, cheaper devices.  This overlooks a couple of important points.  First, it takes a lot of spines and leafs to achieve a statistically non-blocking architecture. For example, to build a non-blocking 96-port fabric out of 48-port devices, you need...wait for it...**six** 48-port devices (2 spines and 4 leaves).  Now we've gotten very good at building cost-effective NPUs for routers but you still need to connect all those spines and leaves.  The optics required for all that connectivity quickly comes to dominate the cost of the system.  Using Active Optical Cables (AOCs) can help mitigate the optics cost but it remains non-neglible.  

Speaking of connectivity, remember that connecting spines and leaves will take a lot of cables.  For our example of 96 user-facing ports, you'll need 96 cables between the spines and leaves for fabric connectivity.  That effectively doubles the number of cables your ops team has to manage.

In terms of space, power and cooling, fabrics exact a cost as well.  We build large, modular systems for a reason: they are very efficient.  In apples-to-apples comparisons (e.g same ASIC family), a fabric of small devices always consumes more space and power for the equivalent number of ports in a large chassis.  

Given the ratio of capex to opex (typically 4:1 for Service Providers), it's also important to take a good hard look at the impact fabrics can have on the ops team more generally.  In our 96-port example, ops has to manage six devices (six IGP instances, six management address, six ACL and QoS domains, etc) where before they might have had only one.  We know from massive data center designs that the only way to scale ops like that is to go all-in on automation.  From cable plans to config to upgrade to troubleshooting, every aspect of network operations will have to be automated.  

## Conclusion: To Fabric or Not To Fabric?

Thanks to the work done in data centers large and small, we know the costs and benefits of network fabrics. For sheer scale, network availability, upgradability, and east-west traffic patterns, it's hard to beat a fabric of small, simple devices.  But the sheer scale of those devices makes large-scale automation an absolute requirement.  And don't expect to save capex or opex by deploying a fabric.

So are fabrics fated to stay inside NPUs, chassis and data centers forever?  Not necessarily.  At some point, the physics and engineering of NPUs will reach a limit such that we can no longer build highly efficient large modular systems.  Since so much of the cost of a fabric is in the optics, changes in optics will impact the feasibility of fabrics in other parts of the network.

Finally, many network architects are starting to realize that you can realize some of the benefits of fabrics without going to a full-scale spine and leaf design.  For example, a little bit more scale out (say 4 or 8 routers in a given role instead of the traditional 2) can enable a smaller blast radius and easier upgrades without the massive device scale of a full fabric.  This might stretch (sorry) our definition of fabrics, but it seems like a step in the right direction.



Network fabrics outside the data center





Precision in language is important to me, so a word like "fabric" can be vexing.  The word "fabrics", like "SDN" and "Cloud", has the potential to mean something very specfic and something entirely nebulous.  It's a slippery word.  Fabrics have existed for a long time.  In a NPU, a fabric module in a chassis, a fabric chassis in a multi-chassis system or the mesh created by spines and leafs in a data center fabric.  Typically refers to a statistically non-blocking architecture as first described by Charles Clos.  Others have stretched (sorry) the fabric concept.  The idea has also been extended to be any system with a unified underlay (e.g. segment routing) and an overlay for services or one that is just "meshier" than normal, with more links and devices, that leverages scale out principles.  Everyone can agree that a common underlay is a good idea.



### Cost of fabrics
cables, optics

### Automation -- scaling ops

### Fabrics today  -- Conclusion
scale out good, horizontal traffic.  PoP as a Fabric.
