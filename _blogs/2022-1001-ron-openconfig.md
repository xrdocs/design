---
published: true
date: '2022-10-01 15:22 -0600'
title: Provisioning OpenZR+ and OIF ZR transceivers using Openconfig 
author: Phil Bedard 
excerpt: Openconfig ZR/ZR+ Provisioning  
permalink: /blogs/zr-openconfig-provisioning
tags:
  - iosxr
  - design
  - optical  
  - ron 
  - routing
  - sdn 
  - controller  
position: hidden  
---
{% include toc %}

# Revision History

| Version          |Date                    |Comments| 
| ---------------- | ---------------------- |-----|
| 1.0       | 10/01/2022| Initial Publication 


<br>


# Routed Optical Networking  

Routed Optical Networking introduced by Cisco in 2020 introduced a fundamental 
shift in how IP+Optical networks are built. Collapsing previously disparate 
network layers and services into a single unified domain, Routed Optical 
Networking simplifies operations and lowers overall network TCO.  More 
information on Routed Optical Networking can be found at the following locations. 


 - <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/datasheet-c78-744377.html> 
 - <https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking.html> 
 - <https://xrdocs.io/latest-routed-optical-networking-hld> 


## Pluggable Digital Coherent Optics 

Simple networks are easier to build and easier to operate. As networks scale to
handle traffic growth, the level of network complexity must decline or at least
remain flat. 

IPoDWDM has attempted to move the transponder function into the router to remove
the transponder and add efficiency to networks. In lower bandwidth applications,
it has been a very successful approach. CWDM, DWDM SFP/SFP+, and CFP2-DCO
pluggable transceivers have been used for many years now to build access,
aggregation, and lower speed core networks. The evolution to 400G and
advances in technology created an opportunity to unlock this potential
in higher speed networks.   

Transponder or muxponders have typically been used to aggregate multiple 10G or
100G signals into a single wavelength. However, with reach limitations, and the
fact transponders are still operating at 400G wavelength speeds, the transponder
becomes a 1:1 input to output stage in the network, adding no benefit. 

The Routed Optical Networking architecture unlocks this efficiency for networks
of all sizes, due to advancements in coherent plugable technology. 

## QSFP-DD and 400ZR and OpenZR+ Standards   

As mentioned, the industry saw a point to improve network efficiency by shifting
coherent DWDM functions to router pluggables. Technology advancements have
shrunk the DCO components into the standard QSFP-DD form factor, meaning no
specialized hardware and the ability to use the highest capacity routers
available today.  ZR/OpenZR+ QSFP-DD optics can be used in the same ports as the
highest speed 400G non-DCO transceivers. 

![](http://xrdocs.io/design/images/ron-hld/ron-design-zrp.png)


Two industry optical standards have emerged to cover a variety of use cases. The
OIF created the 400ZR specification,
<https://www.oiforum.com/technical-work/hot-topics/400zr-2> as a 400G interopable
standard for metro reach coherent optics. The industry saw the benefit of the
approach, but wanted to cover longer distances and have flexibility in
wavelength rates, so the OpenZR+ MSA was created, https://www.openzrplus.org.
The following table outlines the specs of each standard. ZR400 and OpenZR+ transceivers 
are tunable across the ITU C-Band, 196.1 To 191.3 THz.  

![](http://xrdocs.io/design/images/ron-hld/zr_zrp_specs.png)

The following part numbers are used for Cisco's ZR400 and OpenZR+ MSA transceivers 

|Standard|Part| 
|--------|----|
400ZR| QDD-400G-ZR-S| 
|OpenZR+| QDD-400G-ZRP-S|

Cisco datasheet for these transceivers can be found at <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/datasheet-c78-744377.html> 

