---
published: true
date: '2024-02-10 15:22 -0600'
title: Monitoring Digital Coherent Optics in IOS-XR   
author: Phil Bedard 
excerpt: Cisco DCO Monitoring and Troubleshooting  
permalink: /blogs/xr-dco-monitoring
tags:
  - iosxr
  - design
  - optical  
  - ron 
  - routing
  - zr 
  - controller  
  - dco 
position: hidden  
---
{% include toc %}

# Revision History

| Version          |Date                    |Comments| 
| ---------------- | ---------------------- |-----|
| 1.0       | 02/10/2024| Initial Publication 

<br>

# Document Overview 

In this blog we will focus on the ongoing monitoring of digital coherent optics
(DCO) once they are deployed in the network. More information on the
installation and provisioning of DCO optics can be found in the above locations.
We will look at the types of Performance Measurement data important to
be monitored and how to monitoring the PM data through several interfaces
including the router CLI interface, using streaming telemetry, and using 
Cisco's Crosswork family of network automation products.  

# Routed Optical Networking  

Routed Optical Networking introduced by Cisco in 2020 introduced a fundamental 
shift in how IP+Optical networks are built. Collapsing previously disparate 
network layers and services into a single unified domain, Routed Optical 
Networking simplifies operations and lowers overall network TCO.  More 
information on Routed Optical Networking can be found at the following locations:  


 - <https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking.html> 
 - <https://xrdocs.io/latest-routed-optical-networking-hld> 

# Pluggable Digital Coherent Optics Overview 
One of the foundations of Routed Optical Networking is the use of small form
factor pluggable digital coherent optics. These optics can be used in a wide
variety of network applications, reducing CapEx/OpEx cost and reducing
complexity vs. using traditional external transponder equipment.  

## OIF 400ZR and OpenZR+ Standards using QSFP-DD Transceivers  

The networking industry saw a point to improve network efficiency by shifting
coherent DWDM functions to router pluggables. Technology advancements have
shrunk the DCO components into the standard QSFP-DD form factor, meaning no
specialized hardware and the ability to use the highest capacity routers
available today.  ZR/OpenZR+ QSFP-DD optics can be used in the same ports as the
highest speed 400G non-DCO transceivers. 


### Cisco High-Power OpenZR+ "Bright" Transceiver (DP04QSDD-HE0)

<img src="http://xrdocs.io/design/images/ron-hld/bright_zrp.png" width="500"/>

### Cisco OpenZR+ Transceiver (QDD-400G-ZRP-S)

<img src="http://xrdocs.io/design/images/ron-hld/zrp.png" width="500"/>

### Cisco OIF 400ZR Transceiver (QDD-400G-ZR-S)
 <img src="http://xrdocs.io/design/images/ron-hld/zr.png" width="500"/>

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
|OpenZR+ High-Power (Bright) | DP04QSDD-HE0| 


The Cisco datasheet for these transceivers can be found at <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/datasheet-c78-744377.html> and <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/400g-qsfp-dd-high-power-optical-module-ds.html>  


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


# Pluggable Digital Coherent Optics Detail  

Digital Coherent Optics are complex technology. In this blog we will not go into
low level detail of the inner workings of the optics, but look at some of the
basic components to help understand where specific performance measurement 
data is derived from and how best to monitor the optics.   


## Coherent DSP 

At the heart of "Digital" coherent optics is a Digital Signal Processor (DSP).
The definition of a DSP is in the nane, the DSP analyzes an incoming signal,
typically performs some type of manipulation, and then passes that signal onto
another element. DSPs can be very simple, such as one re-sampling digital audio
signals, or complex DSPs can perform many functions.The input interface into the
DSP and the output interface may or may not be the same format, requiring the
DSP to perform the conversion. Signal conversion is just one of many jobs
performed by the DSP in a DCO. It is a bit of a stretch to only call the
processing component inside the DCO a "DSP" due to the number of functions it
performs. 

We will consider the Analog to Digital Conversion (ADC) and Digital to Analog
Conversion (DAC) components part of the "DSP" since they are typically packaged
with the actual signal processor and associated digital components. The ADC/DAC 
is responsible for converting electrical analog to digital, not converting 
electrical analog to an optical signal. 

## Photonic (Optical) Components 

While the DSP takes on many functions performed by photonic components in the
past, we still need photonic components to send/receive light across fiber. The
optical components within the DCO are similar to those found in non-DCO
applications. Common components are the TLA (Tunable Laser Assembly) along with 
TOFs (Tunable Optical Filter) and other pure photonic components such as 
splitters, combiners, and waveguides.   

## Modulators and Receivers  

At some point in the signal path a conversion between an electrical signal to an
optical signal must take place. This is done through the use of modulators and
receivers. The modulators are responsible for modulating an optical signal based
on the electric signal. Receivers are likwise responsible for processing the
optical signal into electrical domain signals which after some pre-processing
can be processed by the DSP. These are commonly referred to as opto-electric
modules because they span both signal domains.     


# IOS-XR DCO Components 
As we have seen the DCO has both Digital and Optical components, IOS-XR
represents the optics in a similar manner as a way to better manage each layer, 
including the PM data we gather and analyze.    

## Optics Controller 
The optics controller on a IOS-XR router represents the physical pluggable
faceplate port on the router. Whether the pluggable is DCO or not, there will be 
an optics controller.  The optics controller is used to manage and monitor the 
physical layer characteristics of an optic. A plug and play gray pluggable optic
doesn't require any physical layer configuration since it's built into the standard
and can only be configured one way. However, there are a number of optical 
properties which can still be monitored on the optics, such as per-lane receive 
power.  

In the case of the DCO, the optics controller is where we configure physical
layer properties such as the frequency and output power. It's also where we
monitor physical layer alarms and PM data.  

## CoherentDSP Controller 
The CoherentDSP controller matches the naming conventions of the optics
controller.  
It represents the digital layer of the DCO and is responsible for alarms and PM
data associated with the digital signal.  


# DCO Performance Data 


# IOS-XR CLI Monitoring 

# IOS-XR DCO Alarms 

## Digital 

# IOS-XR Performance Monitoring and Threshold Crossing Alerts 


