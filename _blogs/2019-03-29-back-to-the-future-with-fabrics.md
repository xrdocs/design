---
published: false
date: '2019-03-29 18:47 -0600'
title: Back to the Future with Fabrics
author: Shelly Cadora
---
## Turning Fabrics Inside Out

Until recently, the notion of a "fabric" in a network was confined to single device: the switching fabric inside an NPU or ASIC, the fabric module of a modular router, or the fabric chassis of a multi-chassis system.  All these internal fabrics fulfilled the same basic function: non-blocking connectivity between the input and output ports of the system. [some reference to Clos here, the basics of the design]

Largely driven by the needs of the massively scalable data center, a new network design has quite literally turned fabrics inside out.  Instead of building fabrics inside silicon, data center fabrics provide (statistically!) non-blocking connectivity by connecting simple devices in a densely connected spine and leaf topology.  These designs can be relatively small (e.g. replacing a large modular devices with a fabric of small devices) or as massive as the data centers they enable (e.g. [Facebook's Data Center Fabric](link)).

Insert picture of internal vs. external fabrics.

As every network architect knows, there is no one "best" design for a network.  Every design represents tradeoffs.  This blog highlights the tradeoffs associated with fabric designs.

## Scale And Availability

When designing networks, scale is almost always top of mind.  There are two basic paradigms for scale:  scale out and scale up.  Networking has traditionally relied primarily on a scale up paradigm:  when you need to scale, buy a bigger box or more line cards or denser line cards.  Scale out, used almost exclusively in cloud-scale computing, takes the oppositive approach.  Instead of buy bigger boxes, buy lots of smaller smaller ones.

How you scale has a direct impact on how you achieve high availability in a given design.  When you scale up, those ever-larger and denser devices create an ever-larger "blast radius."  If you'll lose most or all of your network capacity if that device goes down, you'll want to harden that device with lots of redundant hardware and complex software to keep it all running.  But complex, tightly coupled systems are almost always harder to upgrade, harder to troubleshoot and, according to normal accident theory at least, in danger of failing  




Precision in language is important to me, so a word like "fabric" can be vexing.  The word "fabrics", like "SDN" and "Cloud", has the potential to mean something very specfic and something entirely nebulous.  It's a slippery word.  Fabrics have existed for a long time.  In a NPU, a fabric module in a chassis, a fabric chassis in a multi-chassis system or the mesh created by spines and leafs in a data center fabric.  Typically refers to a statistically non-blocking architecture as first described by Charles Clos.  Others have stretched (sorry) the fabric concept.  The idea has also been extended to be any system with a unified underlay (e.g. segment routing) and an overlay for services or one that is just "meshier" than normal, with more links and devices, that leverages scale out principles.  Everyone can agree that a common underlay is a good idea.

### Scale and Availability
Cattle vs. Pets, Upgradability

### Traffic patterns

### Cost of fabrics
cables, optics

### Automation -- scaling ops

### Fabrics today  -- Conclusion
scale out good, horizontal traffic.  PoP as a Fabric.
