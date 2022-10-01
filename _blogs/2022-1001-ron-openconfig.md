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
|Operational Mode|Integer|The operational mode is an integer representing optical parameters specific to the transceiver. This includes settings such as the line rate, modulation, FEC type, and other vendor specific settings.     

The Frequency, Line Rate, and Operational Mode are required components. The
Transmit Power is optional, a default power will be used based on the
operational mode if none is supplied.  

### Operational Mode Details 
It's worth expanding on the role of the "Operational Mode" used in provisioning 
the transceivers. Cisco has defined a set of integer values used provision the 
QDD-400G-ZRP-S and QDD-400G-ZR-S optics based on standard parameters and Cisco 
Acacia specific parameters.  The following table lists these modes. 

|PID|Operational Mode|Line Rate|FEC Type|Modulation|Baud Rate|Pulse Shaping|
|--------|----|----|----|-----|-----|-----|
|QDD-400G-ZR-S|5003|400|cFEC|16QAM|59.84|No| 
|QDD-400G-ZRP-S|5004|400|cFEC|16QAM|59.84|No| 
|QDD-400G-ZRP-S|5005|400|oFEC|16QAM|60.14|Yes| 
|QDD-400G-ZRP-S|5006|400|oFEC|16QAM|60.14|No| 
|QDD-400G-ZRP-S|5007|300|oFEC|8QAM|60.14|Yes| 
|QDD-400G-ZRP-S|5008|300|oFEC|8QAM|60.14|No| 
|QDD-400G-ZRP-S|5009|200|oFEC|QPSK|60.14|Yes| 
|QDD-400G-ZRP-S|5010|200|oFEC|QPSK|60.14|No| 
|QDD-400G-ZRP-S|5011|200|oFEC|8QAM|40.10|Yes| 
|QDD-400G-ZRP-S|5012|200|oFEC|16QAM|30.08|Yes| 
|QDD-400G-ZRP-S|5013|100|oFEC|QPSK|30.08|No| 


# OpenConfig 

Taken from <https://www.openconfig.net>  

```
OpenConfig defines and implements a common, vendor-independent software layer for managing network devices. OpenConfig operates as an open source project with contributions from network operators, equipment vendors, and the wider community. OpenConfig is led by an Operator Working Group consisting of network operators from multiple segments of the industry.
```

OpenConfig is advancing the paradigm of an abstract set of YANG models used to
perform device configuration and monitoring regardless of vendor. Cisco has
worked with the OpenConfig consortium since its inception to implement these
open community models across IOS-XR, IOS-XE, and NX-OS devices. In IOS-XR 7.7.1
more than 100 OpenConfig models and sub-models are implemented covering a wide
variety of network configuration including device management, routing protocols,
and optical transceiver configuration.  We will focus on the models specific to
configuring the ZR/ZR+ DCO transceivers.  

The official repository for all OpenConfig models can be found at <https://github.com/openconfig/public/> 

## OpenConfig Models for DCO provisioning
This list is only the parent models utilized, and does include imported models.   

|Model|Use|
|--------|----| 
|openconfig-terminal-device| Primary model used to configure input interface to output line port structure and add optical parameters to oc-platform|    
|openconfig-platform|Used to provision optical channel parameters and for monitoring optical channel state|    
|openconfig-platform-transceiver|Used for monitoring physical channel state data such as RX/TX power, and output frequency|  

## Note on Operational Mode Discovery 
There is new work in OpenConfig to enable the discovery of the operational modes 
dynamically from the device/transceiver. As of this writing it's still a relatively 
new concept and has not been implemented in IOS-XR. This is implemented through 
the openconfig-terminal-device-properties model. Once implemented a management 
application can learn the supported optical parameters and constraints to be 
used in path calculation and provisioning.   

## Openconfig Platform and Transceiver Component
The optical parameters used to provision the parent optical-channel and
subsequent physical channel are applied at the component level of the
openconfig-platform model. The OpticalChannel component type is a logical
component with a 1:1 correlation with a physical port. In Cisco routers The
OpticalChannel component is populated when a transceiver capable of supporting
it is inserted. The OpticalChannel will always be represented as
[Rack]/[Slot]-OpticalChannel[Rack][Slot][Instance][Port].  The rack component
will always be 0.  As an example on the 8201-32FH the OpticalChannel for port 20
is represented as 0/0-OpticalChannel0/0/0/20. On the NCS-57C3-MOD router with a
QSFP-DD MPA in MPA slot 3 and DCO transceiver in Port 3 the OpticalChannel is
0/0-OpticalChannel0/0/3/2. On a Cisco 9904 modular router with a A9K-8HG-FLEX-TR
line card in slot 1 and DCO transceiver in port 0, the OpticalChannel is
0/1-OpticalChannel-0/1/0/0.  

### Optical Configuration/State Parameters for OpticalChannel Component 
|Parameter|Units|
|--------|----|
|frequency|Mhz | 
|target-output-power| dBm to two decimal places expressed in increments of .01dBm (+1dBM=100) | 
|operatonal-mode|Integer|

## Openconfig Terminal Device 
In the context of optical device provisioning, one OpenConfig model used is the
Terminal Device model. The original intent of the model was to provision
external optical transponders, and has been implemented by Cisco for use with
the Cisco NCS 1004 muxponder. The model has been recently enhanced to
cover the router pluggable DCO use cases where the "clients" are not physical
external facing ports, but internal to the host router and always associated
with a single external line facing interface.  The Terminal Device model
augments the Platform model to add the additional optical provisioning
configuration parameters to the OpticalChannel component type.   

### Logical Channel Configuration 
The logical channel has several configuration components, which will 
be the same across all similar configurations.

Each logical-channel created must be assigned an integer value. It is up to the 
user to determine the best overall values to use, but the values should not 
overlap between configuration on two different ports.   

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

![](https://xrdocs.io/design/images/ron-hld/ron-oc-muxponder.png)

### Pluggable in Router Use Case 

In the case where a pluggable coherent optic is inserted into a router, the
hierarchical model can be simplified. In the traditional muxponder use case
above, there is a physical client transceiver with its own properties which must
be mapped into an intermediate logical channel. In the case of a router
pluggable, there is no physical client component, only the logical components
associated with the host side of the DCO transceiver.  In Cisco routers, it is
represented as one more Ethernet interfaces depending on the configuration.  


![](https://xrdocs.io/design/images/ron-hld/ron-oc-generic.png)

The example below shows a similar 200G application, but instead of two client
physical ports, there are two HundredGigE interfaces created which are
implicitly connected to the line port since they are integrated into the same
transceiver. This is a fundamental difference from the muxponder use case where
there is no implicit mapping between client and output port. The host side Ethernet 
interfaces of the DCO cannot be mapped to another line port. 

Note this example is only possible with the OpenZR+ transceiver since it
supports line rates of 100G, 200G, 300G, and 400G where the OIF 400ZR only
supports 400G.  

![](https://xrdocs.io/design/images/ron-hld/ron-oc-zrp-200g.png)


# OpenConfig Provisioning Examples 
The following examples are used to illustrate the complete provisioning payloads 
used. The payloads are given in XML for use with NETCONF, supported by all IOS-XR 
routers.  We will go through two examples in detail and then the rest for the 
standard modes will provided in the appendix.  

## Standard OIF 400G ZR Example 
OIF 400ZR transceivers, Cisco PID QDD-400G-ZR-S, can be configured in either 1x400G 
or 4x100G mode. In this example we will show the 1x400G mode, which is the most common 
configuration. The details of the configuration are:  

|Router Type|Port Used|Operational Mode|Frequency|TX Power|  
|--------|----|-----|-----|----|
|8201-32FH|0/0/0/20|5003|194300000|-100| 

Note the 5003 operational mode code which can be expanded as: 

|PID|Rate|FEC|Modulation|Baud Rate|Pulse Shaping|  
|--------|----|-----|-----|----|----|
|QDD-400G-ZR-S|400G|cFEC|16QAM|60.14|No|


```xml
<config>
        <terminal-device xmlns="http://openconfig.net/yang/terminal-device">
            <logical-channels>
                <channel>
                    <index>100</index>
                    <config>
                        <index>100</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_400G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_400GE</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>400</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
                <channel>
                    <index>200</index>
                    <config>
                        <index>200</index>
                        <admin-state>ENABLED</admin-state>
                        <description>Coherent Logical Channel</description>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_OTN</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>400</allocation>
                                <assignment-type>OPTICAL_CHANNEL</assignment-type>
                                <description>Coherent to optical assignment</description>
                                <optical-channel>0/0-OpticalChannel0/0/0/20</optical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
            </logical-channels>
        </terminal-device>
        <components xmlns="http://openconfig.net/yang/platform">
            <component>
                <name>0/0-OpticalChannel0/0/0/20</name>
                <optical-channel xmlns="http://openconfig.net/yang/terminal-device">
                    <config>
                        <target-output-power>-100</target-output-power>
                        <operational-mode>5003</operational-mode>
                        <frequency>194300000</frequency>
                    </config>
                </optical-channel>
            </component>
        </components>
    </config>
  ```

Let's example some specific portions of the config in more detail. 

```xml
              <logical-channels>
                <channel>
                    <index>100</index>
                    <config>
                        <index>100</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_400G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_400GE</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
```

Here we create the first logical channel, associated with the host Ethernet interface, FourHundredGigE0/0/0/20. Since our application has a single 400G interface, the following is configured. The types following the idx: YANG component are defined in openconfig-transport-types.yang.    

|User-Defined Index|Tributary Rate Class|Tributary Protocol|Channel Type|   
|--------|----|-----|-----|
|100|400G|400GE|ETHERNET| 

Next we must map this logical-channel to either a parent logical-channel or output OpticalChannel. Cisco uses a specific "CoherentDSP" interface to represent the framing layer of the DCO transceiver, so there is a parent logical channel representing that layer of the connection. In this case I have a single 400G child interface, so all 400G is mapped to the parent logical channel.   

```xml
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>400</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
```

Next we must define the parent logical-channel associated with the internal interface CoherentDSP0/0/0/20, and map that to the output OpticalChannel associated with the physical port. The rate is configured as 400 to represent 400G.   

```xml
                <channel>
                    <index>200</index>
                    <config>
                        <index>200</index>
                        <admin-state>ENABLED</admin-state>
                        <description>Coherent Logical Channel</description>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_OTN</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>400</allocation>
                                <assignment-type>OPTICAL_CHANNEL</assignment-type>
                                <description>Coherent to optical assignment</description>
                                <optical-channel>0/0-OpticalChannel0/0/0/20</optical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
```

We will use the PROT_OTN encapsulation type for the channel, even though it's not technically a traditional G.709 OTN frame.  

|User-Defined Index|Channel Type|   
|--------|----|
|200|PROT_OTN|

This completes the configuration of the mappings between logical Ethernet and physical output port. Now we must configure the 
optical parameters using the openconfig-platform model.  

```xml
        <components xmlns="http://openconfig.net/yang/platform">
            <component>
                <name>0/0-OpticalChannel0/0/0/20</name>
                <optical-channel xmlns="http://openconfig.net/yang/terminal-device">
                    <config>
                        <target-output-power>-100</target-output-power>
                        <operational-mode>5003</operational-mode>
                        <frequency>194300000</frequency>
                    </config>
                </optical-channel>
            </component>
        </components>
```

As you can see the configuration is relatively straightforward, applying the 
target-output-power, operational-mode, and frequency configuration.  

## 300G Line Rate Configuration 
When we configure ZR+ optics in a 300G line rate configuration, we must map 
individual 100G channels and Ethernet interfaces to a parent 300G container. There
is no 300G Ethernet interface type defined, and based on how modern router NPUs 
are designed they are not typically well suited for creating intermediate 
containers of arbitrary size.  The same is true of the 200G line rate as well. 

Parameters of configuration are: 

|Router Type|Port Used|Operational Mode|Frequency|TX Power|  
|--------|----|-----|-----|----|
|8201-32FH|0/0/0/20|5007|195200000|Default|

Note the 5007 operational mode code which can be expanded as: 

|PID|Rate|FEC|Modulation|Baud Rate|Pulse Shaping|  
|--------|----|-----|-----|----|----|
|QDD-400G-ZRP-S|300G|oFEC|8QAM|60.14|Yes|


The full XML payload is: 

```xml
<config>
        <terminal-device xmlns="http://openconfig.net/yang/terminal-device">
            <logical-channels>
                <channel>
                    <index>100</index>
                    <config>
                        <index>200</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
                <channel>
                    <index>101</index>
                    <config>
                        <index>101</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
                <channel>
                    <index>102</index>
                    <config>
                        <index>102</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>200</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
                <channel>
                    <index>200</index>
                    <config>
                        <index>200</index>
                        <admin-state>ENABLED</admin-state>
                        <description>Coherent Logical Channel</description>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_OTN</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>300</allocation>
                                <assignment-type>OPTICAL_CHANNEL</assignment-type>
                                <description>Coherent to optical assignment</description>
                                <optical-channel>0/0-OpticalChannel0/0/0/20</optical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
            </logical-channels>
        </terminal-device>
        <components xmlns="http://openconfig.net/yang/platform">
            <component>
                <name>0/0-OpticalChannel0/0/0/20</name>
                <optical-channel xmlns="http://openconfig.net/yang/terminal-device">
                    <config>
                        <operational-mode>5007</operational-mode>
                        <frequency>195200000</frequency>
                    </config>
                </optical-channel>
            </component>
        </components>
    </config>
```

First we will inspect the channel configuration for the child logical channel associated with the 
router Ethernet interface HundredGigE0/0/0/20/0. Inspecting the first one we see the following:   

```xml
                <channel>
                    <index>201</index>
                    <config>
                        <index>201</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>100</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
```

Here are the attributes used for the logical channel configuration: 

User-Defined Index|Tributary Rate Class|Tributary Protocol|Channel Type|   
|--------|----|-----|-----|
|100|100G|100G_MLG|ETHERNET| 
|101|100G|100G_MLG|ETHERNET| 
|102|100G|100G_MLG|ETHERNET| 

Note the Tributary Protocol type is now 100G_MLG.  MLG stands for Multi-Link Group meaning this 
logical-channel is part of a larger MLG.  The logical channel is still mapped to the upstream 
CoherentDSP0/0/0/20 logical-channel representing the channel responsible for multiplexing the child 
signals into a single output frame. Details on how this is done in OpenZR+ can be found in the OpenZR+ specifications at 
<https://openzrplus.org>.  When IOS-XR receives the payload it will use the structured channel assignment information 
to properly allocate the child Ethernet interfaces.  

The second logical channel associated with HundredGigE0/0/0/20/1 is similar with the only difference being the index of 
101 instead of 100.  

```xml
                <channel>
                    <index>202</index>
                    <config>
                        <index>202</index>
                        <rate-class xmlns:idx="http://openconfig.net/yang/transport-types">idx:TRIB_RATE_100G</rate-class>
                        <admin-state>ENABLED</admin-state>
                        <description>ETH Logical Channel</description>
                        <trib-protocol xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_100G_MLG</trib-protocol>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_ETHERNET</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>100</allocation>
                                <assignment-type>LOGICAL_CHANNEL</assignment-type>
                                <description>ETH to Coherent assignment</description>
                                <logical-channel>100</logical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
```

The Coherent DSP level logical-channel configuration is similar to the first example, 
except the allocation is now configured as 300 instead of 400 to reflect the 300G line rate.   

```xml
                <channel>
                    <index>100</index>
                    <config>
                        <index>100</index>
                        <admin-state>ENABLED</admin-state>
                        <description>Coherent Logical Channel</description>
                        <logical-channel-type xmlns:idx="http://openconfig.net/yang/transport-types">idx:PROT_OTN</logical-channel-type>
                    </config>
                    <logical-channel-assignments>
                        <assignment>
                            <index>1</index>
                            <config>
                                <index>1</index>
                                <allocation>300</allocation>
                                <assignment-type>OPTICAL_CHANNEL</assignment-type>
                                <description>Coherent to optical assignment</description>
                                <optical-channel>0/0-OpticalChannel0/0/0/20</optical-channel>
                            </config>
                        </assignment>
                    </logical-channel-assignments>
                </channel>
```


The OpticalChannel configuration is also similar with the exception the *target-output-power* setting 
has been omitted. In this case the device default power of -100 (-10dBM) will be used.   

```xml
      <components xmlns="http://openconfig.net/yang/platform">
            <component>
                <name>0/0-OpticalChannel0/0/0/20</name>
                <optical-channel xmlns="http://openconfig.net/yang/terminal-device">
                    <config>
                        <operational-mode>5007</operational-mode>
                        <frequency>195200000</frequency>
                    </config>
                </optical-channel>
            </component>
        </components>
```


# OpenConfig Monitoring Examples 
The optics may also be monitored using the same OpenConfig models as used for 
provisioning, as in OpenConfig models both config and state are have leafs in 
the same model. We will look at two methods for retrieving operational state data, 
using a NETCONF GET and using GNMi which can be used in different ways to retrieve 
operational state data.   

## Using NETCONF 
