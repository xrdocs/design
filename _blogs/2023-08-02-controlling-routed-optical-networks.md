---
published: true
date: '2023-08-02 09:59 +0300'
title: Controlling Routed Optical Networks
author: Ori Gerstel
excerpt: The control architecture required for managing routed optical networks
tags:
  - iosxr
---

{% include toc %}

<br>

# Scope

Routed Optical Networking is an architecture that combines optical networking technologies and packet network technologies into a single network that provides all types of services to customers from a single platform. The resulting network has less layers and an overall simpler structure, and lower overall cost, lower power consumption and lower footprint. The solution is a lot simpler to manage than today’s collection of disparate management tools and enables simpler troubleshooting. More detailed about Routed Optical Networking (RON) can be found [here](https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking/index.html).

The focus of this document is the control and management solution for RON. We explain the goals of this solution, and the selected control architecture that allows achieving its goals in a multivendor environment. We also contrast and compare the solution to other alternatives proposed in the industry.

# Goals of RON control and management

The control and management capabilities for RON are derived from the innovation that the solution introduces. In order to allow SPs to operationalize these innovations, existing management tools are often insufficient, and a new management approach is needed. The goal of this approach is to provide a better management platform than the existing siloed approach of per-domain tools.

**Provisioning of RON links:** provisioning of IP links between WDM pluggables on routers and the open WDM line system (OLS) that connects those routers. 

**Assurance of RON links:** providing information about the operational state of RON links, including both the pluggables and the OLS to enable troubleshooting.

M**anagement of “circuit style” services:** RON enables the creation of services that have the same properties as services provided by optical networks. This allows the creation of a single service platform for the SP. Such services must be managed by the RON control system.

M**anagement of multidomain services:** Optical services often span multiple domains. If part of such a service goes over a RON domain, the entire service should be managed end-to-end and not just the part that goes over RON. In fact even if the service just goes over multiple optical domains, it should be managed end-to-end, improving the management solution over today’s “swivel chair” approach of managing the service parts separately by domain-specific management systems

**Management of all services from a single UI/NBI:** RON advocates a unified solution for managing all service from a single UI or a single NBI to higher layer systems. This includes both traditional IP services, traditional optical services, new network slices and circuit style services. 

**Simplified network visibility:** allowing the operations team to understand the entire IP+optical network with all its layers - both the legacy network and the new RON assets.

**Tools for analyzing network and service issues:** allowing the operations team to quickly understand issues and pinpoint the root cause, based on knowledge of entire network and based on its history.
Unified service assurance: troubleshooting the network from a single pane of glass, leveraging both a high level vendor agnostic view and vendor specific tools.

**Adaptive networking:** tight collaboration between the optical layer and the IP layer to be able to extract the highest performance out of optical connections that are used by RON links, relying on the dynamicity of the IP layer to divert some traffic away from these links, should their optical performance deteriorate.

**Multivendor solution:** the above capabilities must be achieved in a multivendor environment, since networks are rarely single vendor.
Modular solution: The solution must be modular to allow SPs to replace parts they don’t like without an extremely high cost of reimplementing the new solution (vendor lock-in avoidance).

# Control system architecture implications

Most of the above goals are functional requirements that can be met once the control system has access to the various network domains. We will describe how this is done in a section below. However, the last two goals dictate the architecture of the control system and therefore deserve special attention. Achieving a full multivendor solution implies that the control system must have in depth understanding of how each domain and each technology must be configured, as well as an understanding of the possible failure modes, allowing for detailed troubleshooting. This implies that the control system must comprise of vendor specific tools that provide intimate knowledge of the different domains, and an umbrella controller on top that integrates information from all the domains into a single vendor-agnostic database. In other words, a hierarchical structure of domain controllers and a hierarchical controller as shown in the following figure.

![Picture 1.png]({{site.baseurl}}/images/Picture 1.png)


Attempting to implement all these functions in a single controller – like some vendors have suggested – results in creating a “godbox” – a system that knows everything about everything. This has been tried in the past and is doomed for failure since every innovation that a vendor implements in their domain must be supported by the godbox, which means an endless backlog of features and a slow implementation schedule that slows down the innovation that vendors can introduce. This problem becomes worse by the fact that in the optical domain each vendor has their proprietary “secret sauce” of how to compute which optical connections are feasible from a transmission perspective. Skipping the IP controller in an attempt to control the WDM pluggables from an optical controller is also a bad idea: it may work if the solution is only attempting to manage RON links, but it caps the solution to just the first step in the RON evolution and does not allow for advanced functions that require sophisticated IP layer functionality that is implemented in the IP controller.

Even testing a new software release of a godbox becomes a huge undertaking requiring collaboration of all vendors that are part of the solution. Good examples for such a godbox are the Telcordia systems of the past – such as TIRKS, which required an extremely high cost for each change (via the OSMINE process) and took forever to evolve.

Moreover, having a single controller that knows everything violates another requirement from the above list: having a solution in which components can be replaced to avoid vendor lock-in. After a while, a godbox cannot be replaced as it contains too much knowledge making it impossible (or very costly) to replicate. By contrast, a hierarchical structure in built on controllers (both domain and hierarchical) that are small enough to allow another vendor to replicate. Use of standard interfaces between these controllers makes it even easier to substitute one controller by another.

For these reasons the control hierarchy has been agreed upon by various standards bodies such as IETF and ONF and is part of a larger hierarchy that includes OSS tools like service orchestrates and assurance systems and includes other parts of the Service Provider network such as access networks and data center resources as show in in the following figure. Again, the guiding principle is “divide and conquer”: use smaller building blocks specializing in different domains and different technologies, aggregated into systems with wider scope but less domain-specific knowledge to build a hierarchy that together provides full functionality without requiring any component to be too large and complex.

![Picture 2.png]({{site.baseurl}}/images/Picture 2.png){:height="90%" width="90%"}


# Meeting the functional requirements for RON

The functional goals described above are supported by the RON control system as follows.

**Provisioning of RON links:** the hierarchical controller (HCO) identifies the WDM pluggables and the OLS ports that connect to the pluggables. It asks the optical controller of the OLS about connection feasibility and receives the parameters for configuring the pluggables. Once they are configured, the link it turned on. 

**Assurance of RON links:** HCO retries the relevel alarm and performance data from the pluggables as well as from the OLS (via its controller) and displays the information to the user – including the history.

**Management of “circuit style” services:** HCO receives service parameters via the UI or NBI and provisions it via the IP controller in charge of the RON domain (CNC).

**Management of multidomain services:** HCO receives service parameters via UI or NBI and decomposes the service into per-domain service parts, with its specific parameters. It then asks each of the controllers to provision their part of the service.

**Management of all services from a single UI/NBI:** HCO implements a modular service manager to provision all services, allowing each addition of new types of services. 

**Simplified network visibility:** HCO has an up-to-date view of the entire network with all its layers, and how one layer uses resources of underlying layers. This data is exposed via an intuitive vendor-agnostic view to the user.

**Tools for analyzing network and service issues:** The unified view that HCO has of the entire network allows it to quickly analyze existing network issues and identify the root causes. HCO also keeps a historical record of this data, allowing the user to figure out what has changed and when.

**Unified service assurance:** HCO provides a vendor agnostic view of each service, including operational data and performance at the service level as well as the underlying network resources. When this information is no sufficient for in-depth troubleshooting, HCO allows the user to seamlessly drill down to the view of the service or network resource via the UI of the right domain controller.

**Adaptive networking:** HCO coordinates between the optical controller and IP controller to ensure RON links carry the maximum feasible capacity at each point in time, subject to the constraints of the optical layer. If the optical capacity needs to be reduced to allow keeping the link up in the face of deteriorating transmission conditions, the IP controller diverts some traffic away from the link.

# Summary

In this paper we reviewed the goals of the control and management solution for RON. The requirement for a modular multivendor solution implies that this control system must be a hierarchical system, comprising of vendor domain controllers and a vendor-agnostic hierarchical controller. We also explain how the functional goals for RON will be implemented by such a hierarchy.

