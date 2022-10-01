---
published: true
date: '2022-10-01 15:22 -0600'
title: Managing OpenZR+ and OIF ZR transceivers using OpenConfig  
author: Phil Bedard 
excerpt: Openconfig ZR/ZR+ Provisioning  
permalink: /blogs/zr-openconfig-mgmt
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


 - <https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking.html> 
 - <https://xrdocs.io/latest-routed-optical-networking-hld> 



In this blog we will discuss one major component of Routed Optical Networking,
the pluggable digital cohereent optics, and how they are managed using open
models from the OpenConfig consortium. Management includes both provisioning the
transceivers as well as monitoring them via telemetry.  

We will focus primarily on constructs such as OpenConfig YANG models and
provisioning via NETCONF or gNMI. Users looking for a more UI-driven approach to
managing Routed Optical Networking services, the Crosswork Hierarchical
Controller application provides a point and click user interface, but still using
open models to interface with Cisco routers. 

# Pluggable Digital Coherent Optics 
One of the foundations of Routed Optical Networking is the use of small form
factor pluggable digital coherent optics. These optics can be used in a wide
variety of network applications, reducing CapEx/OpEx cost and reducing
complexity vs. using traditional external transponder equipment.  

## QSFP-DD and 400ZR and OpenZR+ Standards   

The networking industry saw a point to improve network efficiency by shifting
coherent DWDM functions to router pluggables. Technology advancements have
shrunk the DCO components into the standard QSFP-DD form factor, meaning no
specialized hardware and the ability to use the highest capacity routers
available today.  ZR/OpenZR+ QSFP-DD optics can be used in the same ports as the
highest speed 400G non-DCO transceivers. 

### Cisco OpenZR+ Transceiver (QDD-400G-ZRP-S)
![](http://xrdocs.io/design/images/ron-hld/zrp.png)

### Cisco OIF 400ZR Transceiver (QDD-400G-ZR-S)
![](http://xrdocs.io/design/images/ron-hld/zr.png)

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



The Cisco datasheet for these transceivers can be found at <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/datasheet-c78-744377.html> 


## Cisco Hardware Support for 400G ZR/ZR+ DCO Transceivers 
Cisco supports the OpenZR+ and OIF ZR transceivers across all IOS-XR product
lines with 400G QSFP-DD ports, including the ASR 9000, NCS 540, NCS 5500, NCS
5700, and Cisco 8000.  Please see the Routed Optical Networking Design or the
individual product pages below for more information on each platform.  

### Cisco 8000 
<https://www.cisco.com/c/en/us/products/collateral/routers/8000-series-routers/datasheet-c78-742571.html>  

### NCS 500 
<https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-500-series-routers/ncs-540-large-density-router-ds.html>

### Cisco ASR 9000 
<https://www.cisco.com/c/en/us/products/collateral/routers/asr-9000-series-aggregation-services-routers/data_sheet_c78-501767.html> 

### NCS 5500 and NCS 5700 
<https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-736270.html><br>  
<https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-744698.html>

## Optical Provisioning Parameters
Optical transceivers are responsible for taking information on their electrical 
"host" interface and translating it into a format suitable for transmission across an 
analog medium, and vice version. Thus the name "transceiver".  The aforementioned 
standards bodies have defined the electrical host interface and optical line 
interface specifications. The resulting configuration of those internal transceiver interfaces 
and parameters are driven by user configuration.  The following represents the 
user-configurable attributes for Cisco ZR/ZR+ DCO transceivers. 


|Parameter|Units|Meaning|  
|--------|----|------| 
|Output Frequency | Hz | Frequency is another method to define the DWDM wavelength being used on the line side of the transceiver   
|Transmit Power | dBm | The transmit power defines the signal power level. dBm is the power ratio of dB referenced to 1mW using the expression dBm = 10log(mW).  As an example 0dBm = 1mW, -3dBm=.50mW, +3dBM=2mW
|Line Rate|Gbps|This is the output trunk rate of the signal, and may be determined by configuration or implicitly by the number of channels assigned| 
|Operational Mode|Integer|The operational mode is an integer representing optical parameters specific to the transceiver. This includes settings such as the line rate, modulation, and other vendor specific settings.     

The Frequency, Line Rate, and Operational Mode are required components. The
Transmit Power is optional, a default power will be used based on the
operational mode if none is supplied.  

### Operational Mode Details 
It's worth expanding on the role of the "Operational Mode" used in provisioning 
the transceivers.  


# OpenConfig 

Taken from <https://www.openconfig.net>  

```
OpenConfig defines and implements a common, vendor-independent software layer for managing network devices. OpenConfig operates as an open source project with contributions from network operators, equipment vendors, and the wider community. OpenConfig is led by an Operator Working Group consisting of network operators from multiple segments of the industry.
```

OpenConfig is advancing the paradigm of an abstract set of YANG models used to perform device configuration and monitoring regardless of vendor. Cisco has worked with the OpenConfig consortium since its inception to implement these open community models across IOS-XR, IOS-XE, and NX-OS devices. In IOS-XR 7.7.1 more than 100 OpenConfig models and sub-models are implemented covering a wide variety of network configuration including device management, routing protocols, and optical transceiver configuration.  We will focus on the models specific to configuring the ZR/ZR+ DCO transceivers.  

The official repository for all OpenConfig models can be found at <https://github.com/openconfig/public/> 

## OpenConfig Models for DCO provisioning 

|Model|Use|
|--------|----| 
|Output Frequency | Hz | Frequency is another method to define the DWDM wavelength being used on the line side of the transceiver   
|Transmit Power | dBm | The transmit power defines the signal power level. dBm is the power ratio of dB referenced to 1mW using the expression dBm = 10log(mW).  As an example 0dBm = 1mW, -3dBm=.50mW, +3dBM=2mW
|Line Rate|Gbps|This is the output trunk rate of the signal, and may be determined by configuration or implicitly by the number of channels assigned| 
|Operational Mode|Integer|The operational mode is an integer representing optical parameters specific to the transceiver. This includes settings such as the line rate, modulation, and other vendor specific settings.     

### Model List 
The following models are used in provisioning and telemetry for DCO transceivers

### Openconfig Terminal Device 
In the context of optical device provisioning, one OpenConfig model used is
the Terminal Device model. The original intent of the model was to provision
external optical transponders, and has been implemented by Cisco for use with
the Cisco 1004 family of muxponders. The model has been recently enhanced to cover the
router pluggable DCO use cases where the "clients" are not physical external
facing ports, but internal to the host router and always associated with a single
external line facing interface.  

### Openconfig Platform and Transceiver Component
The optical parameters used to provision the parent optical-channel and
subsequent physical channel are applied at the component level of the
openconfig-platform model. The OpticalChannel component type is a logical
component with a 1:1 correlation with a physical port. In Cisco routers The
OpticalChannel component is populated when a transceiver capable of supporting
it is inserted. The OpticalChannel will always be represented as
[Rack]/[Slot]/[Instance]-OpticalChannel[Rack][Slot][Instance][Port].  The rack
component will always be 0.  On fixed systems the initial instance value is
omitted.  As an example on the 8201-32FH the OpticalChannel for port 20 is
represented as 0/0-OpticalChannel0/0/0/20. On the NCS-57C3-MOD router with a
QSFP-DD MPA in MPA slot 3 and ZR+ transceiver in Port 3 the OpticalChannel is
0/0-OpticalChannel0/0/3/2.  
### Traditional Muxponder Use Case 

A traditional muxponder maps client physical interfaces to framed output
timeslots, which can then be further aggregated or mapped to a physical output
channel on the DWDM line side. There is no connection between the client port
and line port until the mapping is created.  The Terminal Device model follows
this structure by using a hierarchical structure of channels from client to
eventually output line port. Physical client channels are mapped to intermediate
logical channels, which are ultimately mapped to a physical line output channel.
The model is flexible based on the multiplexing/aggregation required.  

The example below shows the mapping for a 2x100G muxponder application where the
two client ports each map to a 100G logical channel, those map to a 200G logical
channel, and ultimately to a 200G line port associated with the output optical
channel. Note the numbers assigned to the logical channels are arbitrary
integers.  

![](http://xrdocs.io/design/images/ron-hld/ron-oc-muxponder.png)

### Pluggable in Router Use Case 

In the case where a pluggable coherent optic is inserted into a router, the
hierarchical model can be simplified. In the traditional muxponder use case
above, there is a physical client transceiver with its own properties which must
be mapped into an intermediate logical channel. In the case of a router
pluggable, there is no physical client component, only the logical components
associated with the host side of the DCO transceiver.  In Cisco routers, it is
represented as one more Ethernet interfaces depending on the configuration.  

The example below shows a similar 200G application, but instead of two client
physical ports, there are two HundredGigE interfaces created which are
implicitly connected to the line port since they are integrated into the same
transceiver. This is a fundamental difference from the muxponder use case where
there is no implicit mapping between client and output port. The host side Ethernet 
interfaces of the DCO cannot be mapped to another line port. 


![](http://xrdocs.io/design/images/ron-hld/ron-oc-generic.png)

Note this example is only possible with the OpenZR+ transceiver since it
supports line rates of 100G, 200G, 300G, and 400G where the OIF 400ZR only
supports 400G.  

![](http://xrdocs.io/design/images/ron-hld/ron-oc-zrp-200g.png)

