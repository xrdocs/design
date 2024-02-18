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


# IOS-XR DCO Conventions  
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

# DCO Performance Measurement Data 

## Overview 

In this section we will introduce the performance measurement data available
with an explanation of what each component is, why it's important, and what is
considered an ideal range of values.  

## TL;DR What PM values to monitor and what to look for?  

The rest of the document goes into detail about the PM data available and how to
monitor them.  These however are the key values which should be monitored.      

- Optical Receive Power (RX Power) 
- Optical Transmit Power (TX Power)
- Q-Margin 
- OSNR 

RX power, Q-Margin, and OSNR will be have some absolute value once the circuit 
is operational. Each of these values have known minimums based on the DCO 
specifications, but will be different for every operational circuit. It's 
important to monitor the values for changes.  

Note changes may occur due to other network changes such as adding more channels 
on the same fiber span.  

## Optical Layer PM Data 

### Optical Power - Transmit  
Transmit power, or TX power represents the strength of the optical signal
leaving the DCO. It can be adjusted by user configuration.  In most cases
leaving the TX power to the default value is advised. On the QDD-400G-ZR-S and
QDD-400G-ZRP-S the default power for 400G mode is -10dBm. On the DP04QSDD-HE0
the default power for 400G mode is 0dBm. Changes in the TX power can help identify 
hardware issues. The TX power will fluctuate slightly but changes of more than 
.2 dBm may indicate an issue.   

### Optical Power - Receive  
Receive power, or RX power represents the strength of the optical signal
entering the DCO from the line side. There is a direct correlation between the
RX power and the quality of the signal. The DCO has a defined receiver
sensitivity range. If the RX power drops below the minimum RX sensitivity value,
it may not be able to process the incoming signal. It is difficult to give a specific 
acceptable range for RX power since other impairments such as noise may also degrade 
the signal even at acceptable RX power levels. However, in practice the signal should 
always be above -20 dBm.  

### Optical Signal to Noise Ratio 
OSNR is one of the key PM values in monitoring an optical signal. Background noise is
inherent in almost all analog signals. It can also be introduced into the signal
by different photonic elements. Amplifiers can introduce and increase noise. As
the primary signal is amplified so is the noise inherent in analog signals.
Measuring the true optical signal to noise ratio requires test measurement
equipment which cannot be packaged in the size of a DCO. The DCO estimates the
OSNR based on DSP compensation.   

### Chromatic Dispersion
Chromatic Dispersion is a linear optical impairment which degrades the
overall signal and must be corrected by the DCO. Different wavelengths travel at
different speeds, this can commonly be seen using a prism. 400G signals use
75Ghz of spectrum which covers a wide band of frequencies so they do not all
travel together. As the signal travels along the fiber the signal spreads out
meaning the receive end must compensate for the delay between the beginning and
end of the signal. The spread of the signal is measured in picoseconds/nanometer
or ps/nm.   

In IOS-XR the user can configure the "sweep" range used to compensate for the chromatic dispersion 
using the cd-min and cd-max threshold values.  

The user can also monitor the current estimated Chromatic Dispersion values.
These values are derived from calculations done by the DSP during its CD
compensation. 

### Polarization Mode Dispersion 
PMD is another type of signal dispersion or spread. PMD is unique to coherent
signals and measures the spread in time between the X and Y polarized signals
comprising the coherent signal. We don't measure PMD directly but several of the
following PM values are used to measure the effect of PMD. PMD is typically
introduced via imperfections in the fiber or mechanical manipulation of the
fiber such as bending.   

### Differential Group Delay 
DGD is the measure of PMD and is used to express the difference in arrival time
of the two orthogonal signals. DGD can also be known as First Order PMD or
FOPMD. DGD is measured in picoseconds. Perfect fiber has no PMD/DGD but fiber
used in the field is not perfect. Modern SM fiber can introduce .5-1ps of DGD
over 100km, it is directly related to fiber length.  Optical components can also
introduce DGD. DGD is wavelength dependent so may be different for different
wavelengths, it can also be introduced by temperature fluctuations in the fiber.   

### Polarization Dependent Loss 
Signal polarization can also have another effect in introducing signal 
loss as the signal propagates through the fiber. The value is dependent 
on fiber length and is compensated for by the coherent DSP. The value should 
remain low in most instances and is measured in dB. Keep in mind this is 
dB and not the dBm that optical power is expressed in.   

### Second-Order Polarization Mode Dispersion 
SOPMD is a measure of the PMD rates of change related to signal frequency. The
name is due to SOPMD being the second order diffeential with respect to PMD. The
DGD value does not remain static so the DSP must compensate not only for the DGD
value but the rates of change in DGD. This value is measured in picoseconds
squared or ps^2.  

### Polarization Change Rate 
As the polarized signals travel through the fiber, the fiber can cause changes
in the polarization state or SOP (State of Polarization). The coherent DSP 
must compensate for these changes to properly decode the signal. The PCR is 
measured in radians/second or rad/s since the state change is rotational. On 
shorter fiber spans this value should be 0 but longer spans or poor fiber may 
introduce higher values.   

## Digital Layer PM Data 
We've covered optical impairments in the analog optical domain. Ultimately any 
type of signal degradation can lead to errors at the digital layer.  

### Pre-FEC Bit Error Rate
High speed coherent optics expect transmission errors at the bit level due to 
optical signal impairments. Forward error correction uses an algorithm to send 
extra data with the signal so it can be used to "correct" the original signal 
when information is lost or incorrect. Modern algorithms are used to minimize 
the amount of extra data which needs to be sent.  

The Pre-FEC BER is expressed in a ratio of bit errors per samples bits, and at 
the rates being used for ZR/ZR+ optics is very small. It's expressed in scientific 
notation such as 3.7E-04, which is .00037.  It's difficult to monitor this as 
an absolute value, but can be monitored for change over time to help identify 
issues with the fiber. Q-Margin explained below is an easier value to monitor.   

### Post-FEC BER and Uncorrectable Words 
Post-FEC BER measures the amount of bits which are unable to be corrected, and
ultimately lead to UCs or uncorrectable words/bytes. 

Any value other than 0 means there is a critical issue with the signal, and in
most cases due to the sharp FEC cliff it's unlikely the interface will be up if
you are receiving bit errors.   

### Quality Factor (Q-Factor) 
The Q-Factor is a DSP calculated value closely related to the Pre-FEC BER and
OSNR. The Q-Factor provides a minimum SNR value required to meet a certain BER
requirement. Based on the properties of the DCO optics we know at a certain BER
we require a specific SNR, if the BER is very high then we need a higher SNR.
The Q-Factor expressed this relationship via a single value. On the DCO the
value is based on measured BER over a specific time period.   

### Quality Margin (Q-Margin)  
The Q-Margin is a calculated value used to convey the health of the overall
signal after being processed. It indicates how much signal margin exists and if
degradation occurs how much it can degrade before the signal is lost. A Q-Margin
less than 1 is considered unhealthy. The Q-Margin is useful during both circuit
turn-up as well as checking the ongoing health of the circuit.   

## Environmental PM  

### Temperature 
Modern DCO have multiple temperature sensors. The "temperature" reading reported 
by the operating system may be the case temperature or the DSP temperature. The 
laser component typically has its own temperature sensor and is reported as the 
laser temperature.   

### Voltage 
The voltage supplied to the DCO is specified by various standards such as the 
QSFP-DD MSA specifications. The voltage should be approximately 3.3v but can 
vary. Larger fluctuations in the voltage indicate a hardware problem.  

### Laser Bias Current (LBC) 
The LBC is a measure of the bias current applied to the transmit laser used to
maintain stable optical transmit power. This value may change due to
fluctuations in voltage and temperature. The value can be measured either as an
absolute value in milliamps or a percentage of the operating threshold of the 
laser.  


# IOS-XR DCO Monitoring 
The PM values we've introduced will be used in monitoring the health of our 
DCO circuit. We'll focus on IOS-XR as the network operating system, but similar 
methods are usually available with other network operating systems.   

## IOS-XR DCO Events and Alarms 
There are many alarms associated with the state of the optics. This is not meant 
to be an exhaustive list of alarms, just highlight some of the more commonly seen 
alarms during specific events.

Platform alarms generated by DCO events are reported via the system log and
output to remote monitoring tools as syslog messages and SNMP traps if enabled.
These alarms will show up when executing a "show alarms" command on the CLI and
retrieving alarms via YANG models such as openconfig-alarms and
Cisco-IOS-XR-alarmgr-server-oper.   


### Note on built-in PM alarm thresholds 
Some DCO PM values are defined within the optics and optics driver and are not 
user-configurable. These thresholds trigger system level alarms. 

The following PM threshold values are defined for the optics controller.  

<div class="highlighter-rouge">
<pre class="highlight">
 Parameter                 High Alarm  Low Alarm  High Warning  Low Warning
 ------------------------  ----------  ---------  ------------  -----------
 Rx Power Threshold(dBm)         13.0      -24.0          10.0        -21.0
 Rx Power Threshold(mW)          19.9        0.0          10.0          0.0
 Tx Power Threshold(dBm)          4.0      -18.0           2.0        -16.0
 Tx Power Threshold(mW)           2.5        0.0           1.5          0.0
 LBC Threshold(mA)               0.00       0.00          0.00         0.00
 Temp. Threshold(celsius)       80.00      -5.00         75.00        15.00
 Voltage Threshold(volt)         3.46       3.13          3.43         3.16
 LBC High Threshold = 98 %
</pre> 
</div> 

The following PM threshold values are defined for the coherent DSP controller.   
SF=Signal Fault and SD=Signal Degrade.    

<div class="highlighter-rouge">
<pre class="highlight">
BER Thresholds                                  : SF = 1.0E-5  SD = 1.0E-7
</pre> 
</div> 



### Counted Optical Alarms 
Based on either an event or built-in PM threshold, specific alarms increment 
over time. These are seen with the "show controller optics" CLI command or 
appropriate YANG model.   

<div class="highlighter-rouge">
<pre class="highlight">
Alarm Statistics:

         -------------
         HIGH-RX-PWR = 0            LOW-RX-PWR = 0
         HIGH-TX-PWR = 0            LOW-TX-PWR = 0
         HIGH-LBC = 0               HIGH-DGD = 0
         OOR-CD = 0                 OSNR = 18
         WVL-OOL = 0                MEA  = 0
         IMPROPER-REM = 0
         TX-POWER-PROV-MISMATCH = 0

</pre> 
</div> 

|Alarm| Expansion | Comment |  
|-----| ------- |-------|  
|HIGH-RX-PWR | RX power too high || 
|HIGH-TX-PWR | TX power too high || 
|LOW-RX-PWR | RX power too low|| 
|LOW-TX-PWR | TX power too low|| 
|HIGH-LBC | Laser bias current too high || 
|HIGH-DGD | Diffeential group delay too high || 
|OOR-CD | Chromatic dispersion out of range || 
|OSNR | Optical signal to noise ratio too low || 
|WVL-OOL ||  
|MEA| Mismatch equipment alarm | Optic speed not supported by port |  
|IMPROPER-REM|Improper removal| Not applicable to IOS-XR routers |  
|TX-POWER-PROV-MISMATCH| Difference in configured and actual value too large |  

### Counted Digital Alarms 
Based on either an event or built-in PM threshold, specific alarms increment 
over time. These are seen with the "show controller coherentdsp" CLI command or 
appropriate YANG model.   

<div class="highlighter-rouge">
<pre class="highlight">
Alarm Information:
LOS = 40        LOF = 0 LOM = 0
OOF = 0 OOM = 0 AIS = 0
IAE = 0 BIAE = 0        SF_BER = 0
SD_BER = 0      BDI = 0 TIM = 0
FECMISMATCH = 0 FEC-UNC = 0     FLEXO_GIDM = 0
FLEXO-MM = 0    FLEXO-LOM = 0   FLEXO-RDI = 0
FLEXO-LOF = 35
Detected Alarms                                 : LOS
</pre> 
</div> 

|Alarm| Expansion | Comment |  
|-----| ------- |-------|  
|LOS | Loss of signal || 
|LOF | Loss of frame  || 
|LOM | Loss of multi-frame|| 
|OOF | Out of frame || 
|OOM | Out of multi-frame|| 
|IAE | Incoming alignment error || 
|BIAE | Backward incoming alignment error || 
|SF_BER | Signal fault due to high Bit-Error rate  || 
|SD_BER | Signal degrade due to high Bit-Error rate  || 
|BDI | Backward defect indication |  
|TIM | Trace identifier mismatch|OTN TTI mismatch, not used|  
|FECMISMATCH|FEC Mismatch between endpointds, not used| 
|FEC-UNC| Uncorrectable words||   
|FLEXO_GIDM| FlexO framing group ID mismatch|Can happen when one end is channelized and the other isn't| 
|FLEXO_MM| FlexO mismatch|| 
|FLEXO_LOM|FlexO framing Loss of multi-frame||
|FLEXO-RDI|FlexO remote defect indicator|| 
|FLEXO-LOF|FlexO loss of frame|| 

### Common Optical Alarms 



```
optics_driver[389]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :OSNR :DECLARE :Optics0/0/0/20
``` 






## IOS-XR Performance Monitoring and Threshold Crossing Alerts 


