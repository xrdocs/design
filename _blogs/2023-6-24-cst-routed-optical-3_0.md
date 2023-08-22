---
published: true
date: '2023-06-24 15:22 -0600'
title: Cisco Routed Optical Networking 
author: Phil Bedard 
excerpt: Routed Optical Networking 2.1  
tags:
  - iosxr
  - design
  - optical  
  - ron 
  - routing 
position: hidden 
---
{% include toc %}


### PDF Download 

<https://github.com/ios-xr/design/blob/master/Routed-Optical-Networking/2022-12-01-cst-routed-optical-2_0.pdf> 

# Revision History

| Version          |Date                    |Comments| 
| ---------------- | ---------------------- |-----|
| 1.0       | 01/10/2022| Initial Routed Optical Networking Publication 
| 2.0       | 12/01/2022| Private Line Services, NCS 1010, CW HCO updates 
| 2.1       | 06/24/2023| High-Power ZR+ Optics, Bandwidth Guaranteed PLE, Connectivity Verification


# Solution Component Software Versions  

| Element          |Version                    |
| ---------------- | --------------------------|
| Router IOS-XR      | 7.9.1 | 
| NCS 2000 SVO | 12.3.1 | 
| NCS 1010 IOS-XR | 7.9.1 | 
| Cisco Optical Network Controller | 2.1 |
| Crosswork Network Controller | 5.0 | 
| Crosswork Hierarchical Controller | 7.0 |  
| Cisco EPNM | 7.0.1 | 

<br>


# What is Routed Optical Networking?   

Routed Optical Networking as part of Cisco's Converged SDN Transport
architecture brings network simplification to the physical network
infrastructure, just as EVPN and Segment Routing simplify the service and
traffic engineering network layers. Routed Optical Networking collapses complex
technologies and network layers into a single cost efficient and easy to manage
network infrastructure. Here we present the Cisco Routed Optical Networking
architecture and validated design.   

![](http://xrdocs.io/design/images/ron-hld/ron-cst-overview.png)

# Key Drivers
## Changing Networks  

Internet traffic has seen a compounded annual growth rate of 30% or higher over
the last ten years, as more devices are connected, end user bandwidth speeds
increase, and applications continue to move to the cloud. The introduction of 5G
in mobile carriers and backhaul providers is also a disruptor, networks must be
built to handle the advanced services and traffic increase associated with 5G.
Networks must evolve so the infrastructure layer can keep up with the service
layer. 400G Ethernet is the next evolution for SP IP network infrastructure, and
we must make that as efficient as possible.  
## Network Complexity  

Computer networks at their base are a set of interconnected nodes to deliver
data between two endpoints. In the very beginning, these networks were designed
using a layered approach to separate functions. The OSI model is an example of
how functional separation has led to innovation by allowing different standards
bodies to work in parallel at each layer. In some cases even these OSI layers
are further split into different layers. While these layers can bring some cost
benefit, it also brings added complexity. Each layer has its own management,
control plane, planning, and operational model. 
## Inefficiences Between Network Layers  

OTN and IP network traffic must be converted into wavelength
signals to traverse the DWDM network. This has traditionally required dedicated
external hardware, a transponder.  All of these layers bring complexity, and
today some of those layers, such as OTN, bring little to the table in terms of
efficiency or additional value. OTN switching, like ATM previously, has not been
able to keep up with traffic demands due to very complex hardware. Unlike
Ethernet/IP, OTN also does not have a widely interoperable control plane, 
locking providers into a single vendor or solution long-term.   


## Operational Complexity 

Networks involving opaque layers are difficult to plan, build, and operate.  IP
and optical networks often have duplicate teams covering similar tasks. Network
protection and restoration is also often complicated by different schemes
running independently across layers. The industry has tried over decades to
solve some of these issues with complex control planes such as GMPLS, but we are
now at an evolution point where simplifying the physical layers and reducing
control plane complexity in the optical layer allows a natural progression to a
single control-plane and protection/restoration layer.  

## Network Cost  

Simplyfing networks reduces both capex and opex. As we move to 400G, the network
cost is shifted away from routers and router ports to optics. Any way we can
reduce the number of 400G interconnects on the network will greatly reduce cost.
Modeling networks with 400ZR and OpenZR+ optics in place of traditional
transponders and muxponders shows this in almost any network scenario. It also 
results in a reduced space and power footprint.  

# Routed Optical Networking Solution Overview 

As part of the Converged SDN Transport architecture, Routed Optical Networking
extends the key tenet of network simplification.  Routed Optical Networking
tackles the challenges of building and managing networks by simplifying both the
infrastructure and operations.  

## Today's Complex Multi-Layer Network Infrastructure


![](http://xrdocs.io/design/images/ron-hld/multi-layer-network.png)

### DWDM 

Most modern SP networks start at the physical fiber optic layer. Above the
physical fiber is technology to allow multiple photonic wavelengths to traverse
a single fiber and be switched at junction points, we will call that the DWDM
layer. 
### OTN

In some networks, above this DWDM layer is an OTN layer, OTN being the
evolution of traditional SONET/SDH networks. OTN grooms low speed TDM services
into higher speed containers, and if OTN switching is involved, allows switching
these services at intermediate points in the network. OTN is primarily used 
in network to carry guaranteed bandwidth services.   
### Ethernet/IP 

In all high bandwidth networks today, there is an Ethernet layer on which IP
services traverse, since almost all data traffic today is IP. Ethernet
and IP is used due to its ability to support statistical multiplexing, topology
flexibility, and widespread interoperability between different vendors based on
well-defined standards. In larger networks today carrying Internet traffic, the
Ethernet/IP layer does not typically traverse an OTN layer, the OTN layer is
primarily used only for business services. 
# Enabling Technologies  
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

## QSFP-DD, 400ZR, and OpenZR+ Standards   

As mentioned, the industry saw a point to improve network efficiency by shifting
coherent DWDM functions to router pluggables. Technology advancements have
shrunk the DCO components into the standard QSFP-DD form factor, meaning no
specialized hardware and the ability to use the highest capacity routers
available today.  ZR/OpenZR+ QSFP-DD optics can be used in the same ports as the
highest speed 400G non-DCO transceivers. 


### Cisco High Power OpenZR+ Transceiver (DP04QSDD-HE0) **New for 2.1** 
Routed Optical Networking 2.1 introduces the Cisco HIgh Power +1dB ZR+ transceiver. This high launch power 
DCO enables the use of the optics with optical add drop systems requiring higher input power, and enables 
longer distances when used in passive or dark fiber applications without amplification.  

![](http://xrdocs.io/design/images/ron-hld/bright_zrp.png)

### Cisco OpenZR+ Transceiver (QDD-400G-ZRP-S)
![](http://xrdocs.io/design/images/ron-hld/zrp.png){:height="50%" width="50%"}

### Cisco OIF 400ZR Transceiver (QDD-400G-ZR-S)
![](http://xrdocs.io/design/images/ron-hld/zr.png){:height="50%" width="50%"}

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
|High Power OpenZR+| DP04QSDD-HE0|

Cisco datasheet for the QDD-400G-ZRP-S and QDD-400G-ZR-S transceivers can be found at <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/datasheet-c78-744377.html> 

Cisco datasheet for the DP04QSDD-HE0 can be found at <https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/transceiver-modules/400g-qsfp-dd-high-power-optical-module-ds.html> 
## Cisco Routers

We are at a point in NPU development where the pace of NPU bandwidth growth has
outpaced network traffic growth. Single NPUs such as Cisco's Silicon One have a
capacity exceeding 12.8Tbps in a single NPU package without sacrificing
flexibility and rich feature support. This growth of NPU capacity also brings
reduction in cost, meaning forwarding traffic at the IP layer is more
advantageous vs. a network where layer transitions happen often.  

Cisco supports 400ZR and OpenZR+ optics across the NCS 540, NCS 5500, NCS 5700,
ASR 9000, and Cisco 8000 series routers. This enabled providers to utilize the architecture 
across their end to end infrastructure in a variety of router roles. See   

![](http://xrdocs.io/design/images/ron-hld/npu_bandwidth.png)


## Cisco Private Line Emulation 

Starting in Routed Optical Networking 2.0, Cisco now supports Private Line
Emulation (PLE) hardware and IOS-XR support to provide bit-transparent private
line services over the converged packet network. Private Line Emulation supports
the transport of Ethernet, SONET/SDH, OTN, and Fiber Channel services. See the
PLE section of the document for in-depth information on PLE.   
## Circuit Style Segment Routing 
Circuit Style Segment Routing (CS-SR) is another Cisco advancement bringing 
TDM circuit like behavior to SR-TE Policies. These policies use deterministic 
hop by hop routing, co-routed bi-directional paths, hot standby protect paths
with end to end liveness detection, and bandwidth guaranteed services.
Standard Ethernet services not requiring bit transparency can be transported
over a Segment Routing network similar to OTN networks without the additional
cost, complexity, and inefficiency of an OTN network layer.   
## Cisco DWDM Network Hardware  

Routed Optical Networking shifts an expensive and now often redundant
transponder function into a pluggable transceiver. However, to make the most
efficient use of a valuable resource, the underlying fiber optic network, we
still need a DWDM layer. Routed Optical Networking is flexible enough to work
across point to point, ROADM based optical networks, or a mix of both. Cisco
multiplexers, amplifiers, and ROADMs can satisfy any network need.  

*Cisco NCS 1010* 

Routed Optical Networking 2.0 introduces the new Cisco NCS 1010 open optical
line system.  The NCS 1010 represents an evolution in open optical line
systems, utilizing the same IOS-XR software as Cisco routers and NCS 1004
series transponders. This enables the rich XR automation and telemetry support
to extend to the DWDM photonic line system. The NCS 1010 also simplifies how
operators build DWDM networks with advanced integrated functions and a
flexible twin 1x33 WSS.  

See the validated design hardware section for more information.  


<br>


# Routed Optical Networking Network Use Cases

Cisco is embracing Routed Optical Networking in every SP router role. Access,
aggregation, core, peering, DCI, and even PE routers can be enabled with high
speed DCO optics. Routed Optical Networking is also not limited to SP networks,
there are applications across enterprise, government, and education networks.  

![](http://xrdocs.io/design/images/ron-hld/network-use-cases.png)

## Where to use 400ZR and where to use OpenZR+

The OIF 400ZR and OpenZR+ MSA standards have important differences. 

400ZR supports 400G rates only, and targets metro distance point to point
connections up to 120km. 400ZR mandates a strict power consumption of 15W as
well. Networks requiring only 400G over distances less than 120km may benefit
from using 400ZR optics. DCI and 3rd party peering interconnection are good use
cases for 400ZR.  

If a provider needs flexibility in rates and distances and wants to standardize
on a single optics type, OpenZR+ can fulfill the need. In areas of the network
where 400G may not be needed, OpenZR+ optics can be run at 100G or 200G.
Additionally, hardware with QSFP-DD 100G ports can utilize OpenZR+ optics in
100G mode.  This can be ideal for high density access and aggregation networks.

## Supported DWDM Optical Topologies

For those unfamiliar with DWDM hardware, please see the overview of DWDM network
hardware in [Appendix A](#appendix-a) 
{: .notice--warning}

The future of networks may be a flat L3 network with simple point to point
interconnection, but it will take time to migrate to this type of architecture.
Routed Optical Network supports an evolution to the architecture by working over
most modern photonic DWDM networks.  Below gives just a few of the supported
optical topologies including both point to point and ROADM based DWDM networks.

### NCS 2000 64 Channel FOADM P2P Deployment 
This example provides up to 25.6Tb on a single network span, and highlights the
simplicity of the Routed Optical Networking solution.  The "optical" portion of
the network including the ZR/ZR+ configuration can be completed in a matter of
minutes from start to finish.   

![](http://xrdocs.io/design/images/ron-hld/optical-zr-p2p.png)

### NCS 1010 64 Channel FOADM P2P Deployment 
The NCS 1010 includes two add/drop ports with embedded bi-directional EDFA
amplifiers, ideal for connecting the new MD-32-E/O 32 channel, 150Ghz spaced
passive multiplexer. Connecting both even and odd multiplexers allows the use of 
64 total channels. 

![](http://xrdocs.io/design/images/ron-hld/optical-zr-p2p-1010.png)
### NCS 2000 Colorless Add/Drop Deployment 
Using the NCS2K-MF-6AD-CFS colorless NCS2K-MF-LC module along with the LC16 LC
aggregation module, and SMR20-FS ROADM module, a scalable colorless add/drop
complex can be deployed to support 400ZR and OpenZR+.   

![](http://xrdocs.io/design/images/ron-hld/optical-zrp-colorless.png)

### NCS 2000 Multi-Degree ROADM Deployment 
In this example a 3 degree ROADM node is shown with a local add/drop degree. The
Routed Optical Networking solution fully supports ROADM based networks with
optical bypass. The traffic demands of the network will dictate the most
efficient network build. In cases where an existing or new build requires DWDM
switching capability, ZR and ZR+ wavelengths are easily provisioned over the
infrastructure.  

![](http://xrdocs.io/design/images/ron-hld/optical-zrp-roadm.png)


### NCS 1010 Multi-Degree Deployment
A multi-degree NCS 1010 site utilizes a separate NCS 1010 OLT device for each 
degree. The degree may be an add/drop or bypass degree. In our example Site 3 
can support the add/drop of wavelengths via its A/D ports on the upper node, or
express those wavelengths through the interconnect to site 4 via the additional 
1010 OLT unit connected to site 4. In our example the wavelength originating at 
sites 1 and 4 using ZR+ optics is expressed through site 3.  

![](http://xrdocs.io/design/images/ron-hld/optical-zr-bypass-1010.png)


### Long-Haul Deployment 
Cisco has demonstrated in a physical lab 400G OpenZR+ services provisioned
across 1200km using NCS 2000 and NCS 1010 optical line systems. 300G, 200G,
and 100G signals can achieve even greater distances. OpenZR+ is not just for
shorter reach applications, it fulfills an ideal sweet spot in most provider
networks in terms of bandwidth and reach.  

## Core Networks
Long-haul core networks also benefit from the CapEx and OpEx savings of moving
to Routed Optical Networking. Moving to a simpler IP enabled converged
infrastructure makes networks easier to manage and operate vs. networks with
complex underlying optical infrastructure.  The easiest place to start in the
journey is replacing external transponders with OpenZR+ QSFP-DD transceivers. At
400G connecting a 400G gray Ethernet port to a transponder with a 400G or 600G
line side is not cost or environmentally efficient. Cisco can assist in modeling 
your core network to determine the TCO of Routed Optical Networking compared to 
traditional approaches. 

## Metro Aggregation 
Tiered regional or metro networks connecting hub locations to larger aggregation 
site or datacenters can also benefit from Routed Optical Networking. Whether 
deployed in a hub and spoke topology or hop by hop IP ring, Routed Optical Networking 
satisfied provider's growth demands at a lower cost than traditional approaches.  

## Access 
Access deployments in a ring or point-to-point topology are ideal for Routed 
Optical Networking. Shorter distances over dark fiber may not require 
active optical equipment, and with up to 400G per span may provide the bandwidth
necessary for growth over a number of years without the use of additional 
multiplexers.  

## DCI and 3rd Party Location Interconnect 
In this use case, Routed Optical Networking simplifies deployments by eliminating 
active transponders, reducing power, space, and cabling requirements between end 
locations. 25.6Tbps of bandwidth is available over a single fiber using 64 400G 
wavelengths and simple optical amplifiers and multiplexers requiring no additional 
configuration after initial turn-up.   

# Routed Optical Networking Private Line Services 
Release 2.0 introduces Circuit Style Segment Routing TE Policies and Private Line 
Emulation hardware to enable traditional TDM-like private line services over the 
converged Segment Routing packet network. The following provides an overview of 
the hardware and software involved in supporting PL services. The figure below 
gives an overview of PLE service signaling and transport.  

![](http://xrdocs.io/design/images/ron-hld/ron-ple-overview.png)


## Circuit Style Segment Routing 
CS-SR provides the underlying TDM-like transport to support traditional
private line Ethernet services without additional hardware and bit-transparent
Ethernet, OTN, SONET/SDH, and Fiber Channel services using Private Line
Emulation hardware. 

### CS SR-TE paths characteristics 

* Co-routed Bidirectional - Meaning the paths between two client ports are symmetric
* Deterministic without ECMP - Meaning the path does not vary based on any load balancing criteria
* Persistent - Paths are routed on a hop by hop basis, so they are not subject to path changes induced by network changes 
* End-to-end path protection - Entire paths are switched from working to protect with the protect path in a hot standby state for fast transition  
* Bandwidth guaranteed paths  

SR CS-TE policies are built using link adjacency SIDs without protection to ensure the paths do not take a TI-LFA path during 
path failover and instead fail over to the pre-determined protect path.  

### CS SR-TE path liveness detection 
Paths can be configured with end to end liveness detection. Liveness detection
uses STAMP (Simple Two-Way Active Measurement Protocol) probes which are
looped at the far end to determine if the end to end path is up
bi-directionally. If more than the set number of probes is missed (set by the
multiplier) the path will be considered down. Once liveness detection is
enabled probes will be sent on all candidate paths. Either the default
liveness probe profile can be used or if you want to modify the default
parameters a customized one can be created. 
### CS SR-TE path failover behavior 
CS SR-TE policies contain multiple candidate paths.  The highest preference
candidate path is considered the working path, the second highest preference
path is the protect path, and if a third lower preference path is configured
would be a dynamic restoration path.  This provides 1:1+R protection for CS
SR-TE policies.   

![](http://xrdocs.io/design/images/ron-hld/ron-srcs-paths.png)

### Static CS SR-TE Policies 
Static CS SR-TE policies are policies using explicit pre-defined paths for the
working and protect paths in both the forward and reverse direction. These
explicit paths define the hop by hop path using adjacency-SIDs. The explicit SID
lists are defined on the head-end routers and part of the persistent device
configuration. They can be built by a user or an external controller which is
creating the paths by available network information.  Explicit SID lists can be 
referenced by multiple SR Policies.   
 
The following below shows the configuration of a static CS SR-TE
Policy with a working, protect, and restoration path.  

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  policy to-55a2-1
   color 1001 end-point ipv4 100.0.0.44
   path-protection
   !
   candidate-paths
    preference 25
     dynamic
       metric-type igp 
     !
    !
    preference 50
     explicit segment-list protect-forward-path
      reverse-path segment-list protect-reverse-path
     !
    !
    preference 100
     explicit segment-list working-forward-path
      reverse-path segment-list working-reverse-path
     !
    !
   !
   performance-measurement
    liveness-detection
     liveness-profile name liveness-check
</pre>
</div>

### CS SR-TE Policy operational details 
<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs55a2-1#show segment-routing traffic-eng policy color 1001
Sat Dec  3 13:32:38.356 PST

SR-TE policy database
---------------------

Color: 1001, End-point: 100.0.0.42
  Name: srte_c_1001_ep_100.0.0.42
  Status:
    adjmin: up  Operational: up for 2d09h (since Dec  1 04:08:12.648)
  Candidate-paths:
    Preference: 100 (configuration) (active)
      Name: to-100.0.0.42
      Requested BSID: dynamic
      PCC info:
        Symbolic name: cfg_to-100.0.0.42_discr_100
        PLSP-ID: 1
      Constraints:
        Protection Type: protected-preferred
        Maximum SID Depth: 12
      Explicit: segment-list forward-adj-path-working (valid)
        Reverse: segment-list reverse-adj-path-working
        Weight: 1, Metric Type: TE
          SID[0]: 15101 [adjacency-SID, 100.1.1.21 - 100.1.1.20]
          SID[1]: 15102
          SID[2]: 15103
          SID[3]: 15104
        Reverse path:
          SID[0]: 15001
          SID[1]: 15002
          SID[2]: 15003
          SID[3]: 15004
      Protection Information:
        Role: WORKING
        Path Lock: Timed
        Lock Duration: 300(s)
        State: ACTIVE
    Preference: 50 (configuration) (protect)
      Name: to-100.0.0.42
      Requested BSID: dynamic
      PCC info:
        Symbolic name: cfg_to-100.0.0.42_discr_50
        PLSP-ID: 2
      Constraints:
        Protection Type: protected-preferred
        Maximum SID Depth: 12
      Explicit: segment-list forward-adj-path-protect(valid)
        Reverse: segment-list reverse-adj-path-protect
        Weight: 1, Metric Type: TE
          SID[0]: 15119 [adjacency-SID, 100.1.42.1 - 100.1.42.0]
        Reverse path:
          SID[0]: 15191
      Protection Information:
        Role: PROTECT
        Path Lock: Timed
        Lock Duration: 300(s)
        State: STANDBY
  Attributes:
    Binding SID: 24017
    Forward Class: Not Configured
    Steering labeled-services disabled: no
    Steering BGP disabled: no
    IPv6 caps enable: yes
    Invalidation drop enabled: no
    Max Install Standby Candidate Paths: 0
</pre>
</div>


### Dynamic Circuit Style SR-TE 

Release 2.1 of Routed Optical Networking introduces the concept of dynamic CS
SR-TE. Dynamic CS SR-TE utilizes Cisco's SR-PCE Path Computation Element to
compute the working and protection paths of the CS SR-TE policy. Each head-end
node acts as a Path Computation Client, utilizing PCEP and Circuit Style PCEP
extensions to communicate the required path characteristics to SR-PCE. Utilizing
SR-PCE to compute the optimal disjoint paths simplifies the configuration and
deployment of Circuit-Style Policies.   

#### Bi-directional Association ID 

Circuit style paths are bi-directional and co-routed. The association ID value
is used to communicate the constraint to SR-PCE. SR-PCE will then compute the
same co-routed path from each head-end router to the tail-end router. The
working paths on both head-end routers require using the same ID.  The protect
paths will also use the same ID, but unique from the working path ID. Likewise,
the optional restoration paths will also utilize a unique ID.  The identifiers
should be globally unique for each pair of CS SR-TE policies.  The following 
gives an example of the identifiers: 

|Router|Path Type|Association ID|  
|--------|--------|----------|
|A| Working|100|
|Z| Working|100| 
|A| Protect|200|
|Z| Protect|200| 
|A| Restoration|300|
|Z| Restoration|300| 

#### Disjoint path group ID  

Another property of CS SR-TE policies is working and protect paths are fully 
disjoint. A disjoint group ID is used to communicate the constraint to SR-PCE. On 
each head-end node the working and protect candidate paths are assigned the 
same disjoint group ID. The disjoint group ID will be globally unique on 
each head-end node. Available options for path disjointness are node, link, and 
srlg.  

|Router|Path Type|Disjoint Group ID|  
|--------|--------|----------|
|A| Working|101|
|Z| Working|201| 
|A| Protect|101|
|Z| Protect|201| 

#### Path constraints 

Circuit style paths utilize only unprotected adjacency SID, those constraints are 
communicated to SR-PCE using specific configuration and flags in PCEP. 

The following configuration shows the full dynamic SR-TE configuration on 
each head-end node.  

**Router A**

<div class="highlighter-rouge">
<pre class="highlight">
policy dynamic-cs-srte-to-55a2-p2
   color 119 end-point ipv4 100.0.0.44
   path-protection
   !
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type igp
      !
     !
     constraints
      segments
       protection unprotected-only
       adjacency-sid-only
      !
      disjoint-path group-id 10 type link
     !
     bidirectional
      co-routed
      association-id 101
     !
    !
    preference 200
     dynamic
      pcep
      !
      metric
       type igp
      !
     !
     lock 
       duration 30
     !
     constraints
      segments
       protection unprotected-only
       adjacency-sid-only
      !
      disjoint-path group-id 10 type link
     !
     bidirectional
      co-routed
      association-id 201
     !
    !
    preference 50
     dynamic
      pcep
      !
      metric
       type igp
      !
     !
     backup-ineligible
     lock 
       duration 60
     !
     constraints
      segments
       protection unprotected-only
       adjacency-sid-only
      !
     !
     bidirectional
      co-routed
      association-id 301
     !
    !
   !
   performance-measurement
    liveness-detection
     liveness-profile backup name CS-PROTECT
     liveness-profile name CS-WORKING
     invalidation-action down
</pre>
</div>


**Router Z**

<div class="highlighter-rouge">
<pre class="highlight">
policy dynamic-cs-srte-to-57c3-p2
   color 119 end-point ipv4 100.0.0.42
   path-protection
   !
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type igp
      !
     !
     constraints
      segments
       protection unprotected-only
       adjacency-sid-only
      !
      disjoint-path group-id 11 type link
     !
     bidirectional
      co-routed
      association-id 101
     !
    !
    preference 200
     dynamic
      pcep
      !
      metric
       type igp
      !
     !
     lock 
       duration 30
     !
     constraints
      segments
       protection unprotected-only
       adjacency-sid-only
      !
      disjoint-path group-id 11 type link
     !
     bidirectional
      co-routed
      association-id 201
     !
    !
    preference 50
     dynamic
      pcep
      !
      metric
       type igp
      !
     !
     backup-ineligible
     lock 
       duration 60
     !
     constraints
      segments
       protection unprotected-only
       adjacency-sid-only
      !
     !
     bidirectional
      co-routed
      association-id 301

   !
   performance-measurement
    liveness-detection
     liveness-profile backup name CS-PROTECT
     liveness-profile name CS-WORKING
     invalidation-action down
</pre>
</div>


### Circuit-Style SR-TE with Bandwidth Admission Control using Crosswork Circuit Style Manager 

Routed Optical Networking 2.1 with Crosswork Network Controller 5.0 and IOS-XR
7.9.1 now supports utilizing the new Circuit Style Manager to provide Bandwidth
Admission Controller and guaranteed bandwidth paths for Circuit-Style Policies.
CNC 5.0 also supports full provisioning, monitoring, and visualization of
Circuit-Style Policies.  

#### CNC Circuit-Style Manager Configuration 
In CNC 5.0, Circuit Style Manager uses a simple network-wide bandwidth
percentage setting to reserve a specific amount of bandwidth for BW-guaranteed
CS-SRTE policies. CNC's network model will track the allocation of bandwidth on
each link and the amount of capacity reserved by active CS-SRTE Policies. 

The Link CS BW Min Threshold configuration is used to trigger system alerts when
the BW on a link meets or exceeds the threshold percentage configured by the
user. 

![](http://xrdocs.io/design/images/ron-hld/ron-cnc-csm-configuration.png){:height="100%" width="100%"}

**Allocated and reserved link bandwidth**

![](http://xrdocs.io/design/images/ron-hld/ron-cnc-csm-link-bandwidth.png){:height="100%" width="100%"}

#### SR-PCE to Circuit Style Manager (CSM) Communication 

CSM communicates to SR-PCE through the SR-PCE northbound API. When the session is established 
between CSM and SR-PCE, SR-PCE will delegate all CS-SRTE Policies with bandwidth constraints 
to CSM for path computation. 

![](http://xrdocs.io/design/images/ron-hld/ron-cnc-csm-communication.png){:height="100%" width="100%"}

#### Bandwidth Admission Control Operation 
BW CAC is supported for dynamic CS SR-TE Policies. Utilizing the "bandwidth"
configuration option for the policy triggers the inclusion of the "bandwidth"
object in the PCEP request to SR-PCE. SR-PCE will delegate path computation
requests with bandwidth constraints to CNC CSM. Based on the CS-SRTE Policy
configuration, CSM will compute a Working, Protect, and Restoration path to be
used by the policy. The paths computed by CSM will adhere to the CS-SRTE
properties with Working and Protect paths being fully disjoint (link, node, or
SRLG) and each path will be co-routed meaning the Working path from A to Z will
be identical to the path from Z to A.  

**CS SR-TE Policy Bandwidth Configuration** 

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  policy srte_c_3000_ep_100.0.0.44
   bandwidth <b>10000000</b> 
   color 3000 end-point ipv4 100.0.0.44
   path-protection
</pre>
</div>

**Operational Information** 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs55a2-2#show segment-routing traffic-eng policy color 3000  | beg Attributes
Mon Jun 12 17:55:44.238 PDT
  Attributes:
    Binding SID: 24010
    Forward Class: Not Configured
    Steering labeled-services disabled: no
    Steering BGP disabled: no
    IPv6 caps enable: yes
    <b>Bandwidth Requested: 10.000 Gbps
    Bandwidth Current: 10.000 Gbps</b>
</pre>
</div>

### CNC CS-SRTE Monitoring 
Starting in CNC 5.0, CS-SRTE Policies are fully supported. CS-SRTE Policies 
are identified as a specific type of policy in the Traffic Engineering dashboard 
and have enhanced visualization and monitoring capabilities.  

**CS-SRTE Dashboard** 
![](http://xrdocs.io/design/images/ron-hld/ron-cnc-csm-policy-dashboard.png){:height="100%" width="100%"}

**CS-SRTE Policy Visualization of Working and Protect Paths** 
When visualizing CS-SRTE policies we can see both Working and Protect paths 
including which path is currently active, denoted by the A icon.  

![](http://xrdocs.io/design/images/ron-hld/ron-cnc-csm-policy-overview.png){:height="100%" width="100%"}

**CS-SRTE Policy Path Details** 
When we inspect the policy path details we can see both the requested and reserved bandwidth for the path 
and then the additional details such as path constraints and hop by hop path.  

![](http://xrdocs.io/design/images/ron-hld/ron-cnc-csm-policy-path-details.png){:height="100%" width="100%"}

### CNC CS-SRTE Provisioning 
In CNC 5.0 the Circuit-Style SR-TE Function Pack is supported. This service type
simplifies CS-SRTE provisioning by dynamically allocating IDs for the
bi-directional association ID and disjoint-group ID. It also simplifies
provisioning by provisioning both the A to Z and Z to A policies at one time vs.
defining each one independently.   

**NSO Payload for Dynamic CS SR-TE Policy** 

```json
{
  "data": {
    "cisco-cs-sr-te-cfp:cs-sr-te-policy": [
      {
        "name": "clus-demo-ple-fc8",
        "head-end": {
          "device": "ron-ncs55a2-1",
          "ip-address": "100.0.0.44"
        },
        "tail-end": {
          "device": "ron-ncs55a2-2",
          "ip-address": "100.0.0.22"
        },
        "color": 3000,
        "bandwidth": 10000000,
        "disjoint-path": {
          "forward-path": {
            "type": "node",
            "group-id": 900
          },
          "reverse-path": {
            "type": "node",
            "group-id": 901
          }
        },
        "path-protection": {
        },
        "performance-measurement": {
          "liveness-detection": {
            "profile": "ple",
            "backup": "ple",
            "logging": {
              "session-state-change": [null]
            }
          }
        },
        "working-path": {
          "dynamic": {
            "constraints": {
              "segments": {
                "protection": "unprotected-only"
              }
            },
            "pce": {
            },
            "metric-type": "latency",
            "bidirectional-association-id": 1000
          }
        },
        "protect-path": {
          "dynamic": {
            "constraints": {
              "segments": {
                "protection": "unprotected-only"
              }
            },
            "pce": {
            },
            "metric-type": "latency",
            "bidirectional-association-id": 1001
          },
          "revertive": true,
          "wait-to-revert-timer": 30
        }
      }
    ]
  }
}
``` 

## Private Line Emulation Hardware 
Starting in IOS-XR 7.7.1 the NC55-OIP-02 Modular Port Adapter (MPA) is supported
on the NCS-55A2-MOD and NCS-57C3-MOD platforms. The NC55-OIP-02 has 8 SFP+ ports 

![](http://xrdocs.io/design/images/ron-hld/ron-hardware-ple-mpa-2.png){:height="100%" width="100%"}

Each port on the PLE MPA can be configured independently. The PLE MPA is responsible 
for receiving data frames from the native PLE client and packaging those into fixed 
frames for transport over the packet network.  

More information on the NC55-OIP-02 can be found in its datasheet located at 
<https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/network-con-5500-series-ds.pdf>. A full 
detailed to end to end configuration for PLE can be found in the Routed Optical Networking 2.0 Solution Guide found at <https://www.cisco.com/c/en/us/td/docs/optical/ron/2-0/solution/guide/b-ron-solution-20/m-ron.pdf>

### Supported Client Transceivers 

|Transport Type| Supported Transceivers| 
|--------|--------|
|Ethernet| SFP-10G-SR/LR/ER, GLC-LH/EX/ZX-SMD, 1G/10G CWDM |  
|OTN (OTU2e)| SFP-10G-LR-X, SFP-10G-ER-I, SFP-10G-Z |  
|SONET/SDH| ONS-SC+-10G-LR/ER/SR (OC-192/STM-64), ONS-SI-2G-L1/L2/S1 (OC-48/STM-16) | 
|Fiber Channel|DS-SFP-FCGE, DS-SFP-FC8G, DS-SFP-FC16G, DS-SFP-FC32G, 1/2/4/8G FC CWDM|

Note FC32G transceivers are supported in the even ports only and will disable the adjacent odd 
SFP+ port.  
{: .notice--warning}

## Private Line Emulation Pseudowire Signaling 
PLE utilizes IETF SAToP pseudowire encoding carried over dynamically signalled EVPN-VPWS 
circuits. Enhancements to the EVPN VPWS service type have been introduced to the IETF via 
<https://datatracker.ietf.org/doc/draft-schmutzer-bess-ple>. 

PLE services use Differential Clock Recovery (DCR) to ensure proper frame timing between 
the two PLE clients. In order to mmaintain accuracy of the clock each PLE endpoint router 
must have its frequency source traceable to a common primary reference clock (PRC).   

## Private Line Emulation EVPN-VPWS Configuration 
PLE services can be configured to utilize a CS SR-TE Policy or use dynamic MPLS 
protocols. The example belows shows the use of CS SR-TE Policy as transport for the 
PLE EVPN-VPWS service. Note the name of the sr-te policy in the preferred path command 
is the persistent generated name and not the name used in the CLI configuration. This can 
be determined using the "show segment-routing traffic-engineering policies" command.   

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 pw-class circuit-style-srte
  encapsulation mpls
   preferred-path sr-te policy srte_c_1001_ep_100.0.0.42
  !
 !
 xconnect group ple
  p2p ple-cs-1
   interface CEM0/0/2/1
   neighbor evpn evi 100 target 4201 source 4401
    pw-class circuit-style-srte
   !
  !
</pre>
</div>

## PLE Monitoring and Telemetry 

The following "show" command can be used to monitor the state of PLE ports and services. 

### Client Optics Port State 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs55a2-1#show controllers optics 0/0/2/1
Sat Dec  3 14:00:10.873 PST

 Controller State: Up

 Transport Admin State: In Service

 Laser State: On

 LED State: Not Applicable

 Optics Status

         Optics Type:  SFP+ 10G SR
         Wavelength = 850.00 nm

         Alarm Status:
         -------------
         Detected Alarms: None


         LOS/LOL/Fault Status:

         Laser Bias Current = 8.8 mA
         Actual TX Power = -2.60 dBm
         RX Power = -2.33 dBm

         Performance Monitoring: Disable

         THRESHOLD VALUES
         ----------------

         Parameter                 High Alarm  Low Alarm  High Warning  Low Warning
         ------------------------  ----------  ---------  ------------  -----------
         Rx Power Threshold(dBm)          2.0      -13.9          -1.0         -9.9
         Tx Power Threshold(dBm)          1.6      -11.3          -1.3         -7.3
         LBC Threshold(mA)              13.00       4.00         12.50         5.00
         Temp. Threshold(celsius)       75.00      -5.00         70.00         0.00
         Voltage Threshold(volt)         3.63       2.97          3.46         3.13

         Polarization parameters not supported by optics

         Temperature = 33.00 Celsius
         Voltage = 3.30 V

 Transceiver Vendor Details

         Form Factor            : SFP+
         Optics type            : SFP+ 10G SR
         Name                   : CISCO-FINISAR
         OUI Number             : 00.90.65
         Part Number            : FTLX8574D3BCL-CS
         Rev Number             : A
         Serial Number          : FNS23300J42
         PID                    : SFP-10G-SR
         VID                    : V03
         Date Code(yy/mm/dd)    : 19/07/25
</pre>
</div>


### PLE CEM Controller Stats 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs57c3-1#show controllers CEM 0/0/3/1
Sat Sep 24 11:34:22.533 PDT
Interface                          : CEM0/0/3/1
Admin state                        : Up
Oper  state                        : Up
Port bandwidth                     : 10312500 kbps
Dejitter buffer (cfg/oper/in-use)  : 0/813/3432 usec
Payload size (cfg/oper)            : 1280/1024 bytes
PDV (min/max/avg)                  : 980/2710/1845 usec
Dummy mode                         : last-frame
Dummy pattern                      : 0xaa
Idle pattern                       : 0xff
Signalling                         : No CAS
RTP                                : Enabled
Clock type                         : Differential
Detected Alarms                    : None

Statistics Info
---------------
Ingress packets          : 517617426962, Ingress packets drop     : 0
Egress packets           : 517277124278, Egress packets drop      : 0
Total error              : 0
        Missing packets          : 0, Malformed packets        : 0
        Jitter buffer underrun   : 0, Jitter buffer overrun    : 0
        Misorder drops           : 0
Reordered packets        : 0, Frames fragmented        : 0
Error seconds            : 0, Severely error seconds   : 0
Unavailable seconds      : 0, Failure counts           : 0

Generated L bits         : 0, Received  L bits         : 0
Generated R bits         : 339885178, Received  R bits         : 17

Endpoint Info
-------------
Passthrough     : No
</pre>
</div> 

### PLE CEM PM Statistics 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs57c3-1#show controllers CEM 0/0/3/1 pm current 30-sec cem
Sat Sep 24 11:37:02.374 PDT

CEM in the current interval [11:37:00 - 11:37:02 Sat Sep 24 2022]

CEM current bucket type : Valid
INGRESS-PKTS                : 2521591              Threshold : 0            TCA(enable) : NO
EGRESS-PKTS                 : 2521595              Threshold : 0            TCA(enable) : NO
INGRESS-PKTS-DROPPED        : 0                    Threshold : 0            TCA(enable) : NO
EGRESS-PKTS-DROPPED         : 0                    Threshold : 0            TCA(enable) : NO
INPUT-ERRORS                : 0                    Threshold : 0            TCA(enable) : NO
OUTPUT-ERRORS               : 0                    Threshold : 0            TCA(enable) : NO
MISSING-PKTS                : 0                    Threshold : 0            TCA(enable) : NO
PKTS-REORDER                : 0                    Threshold : 0            TCA(enable) : NO
JTR-BFR-UNDERRUNS           : 0                    Threshold : 0            TCA(enable) : NO
JTR-BFR-OVERRUNS            : 0                    Threshold : 0            TCA(enable) : NO
MIS-ORDER-DROPPED           : 0                    Threshold : 0            TCA(enable) : NO
MALFORMED-PKT               : 0                    Threshold : 0            TCA(enable) : NO
ES                          : 0                    Threshold : 0            TCA(enable) : NO
SES                         : 0                    Threshold : 0            TCA(enable) : NO
UAS                         : 0                    Threshold : 0            TCA(enable) : NO
FC                          : 0                    Threshold : 0            TCA(enable) : NO
TX-LBITS                    : 0                    Threshold : 0            TCA(enable) : NO
TX-RBITS                    : 0                    Threshold : 0            TCA(enable) : NO
RX-LBITS                    : 0                    Threshold : 0            TCA(enable) : NO
RX-RBITS                    : 0                    Threshold : 0            TCA(enable) : NO

</pre>
</div> 

### PLE Client PM Statistics  

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:ron-ncs57c3-1#show controllers EightGigFibreChanCtrlr0/0/3/4 pm current 30-sec fc
Sat Sep 24 11:51:55.168 PDT

FC in the current interval [11:51:30 - 11:51:55 Sat Sep 24 2022]

FC current bucket type : Valid
 IFIN-OCTETS                           : 16527749196          Threshold : 0            TCA(enable) : NO
 RX-PKT                                : 196758919            Threshold : 0            TCA(enable) : NO
 IFIN-ERRORS                           : 0                    Threshold : 0            TCA(enable) : NO
 RX-BAD-FCS                            : 0                    Threshold : 0            TCA(enable) : NO
 IFOUT-OCTETS                          : 0                    Threshold : 0            TCA(enable) : NO
 TX-PKT                                : 0                    Threshold : 0            TCA(enable) : NO
 TX-BAD-FCS                            : 0                    Threshold : 0            TCA(enable) : NO
 RX-FRAMES-TOO-LONG                    : 0                    Threshold : 0            TCA(enable) : NO
 RX-FRAMES-TRUNC                       : 0                    Threshold : 0            TCA(enable) : NO
 TX-FRAMES-TOO-LONG                    : 0                    Threshold : 0            TCA(enable) : NO
 TX-FRAMES-TRUNC                       : 0                    Threshold : 0            TCA(enable) : NO
</pre>
</div> 




# Routed Optical Networking Architecture Hardware 
All Routed Optical Networking solution routers are powered by Cisco IOS-XR.   
## Routed Optical Networking Validated Routers 

Below is a non-exhaustive snapshot of platforms validated for use with ZR and
OpenZR+ transceivers. Cisco supports Routed Optical Networking in the NCS 540,
NCS 5500/5700, ASR 9000, and Cisco 8000 router families. The breadth of coverage
enabled the solution across all areas of the network.   

![](http://xrdocs.io/design/images/ron-hld/ron-validated-hardware.png)
### Cisco 8000 Series  

The Cisco 8000 and its Silicone One NPU represents the next generation in
routers, unprecedented capacity at the lowest power consumption while supporting
a rich feature set applicable for a number of network roles. 

See more information on Cisco 8000 at <https://www.cisco.com/c/en/us/products/collateral/routers/8000-series-routers/datasheet-c78-742571.html>  

Specific information on ZR/ZR+ support can be found at <https://www.cisco.com/c/en/us/td/docs/iosxr/cisco8000/Interfaces/73x/configuration/guide/b-interfaces-config-guide-cisco8k-r73x/m-zr-zrp-cisco-8000.html> 

### Cisco 5700 Systems and NCS 5500 Line Cards 

The Cisco 5700 family of fixed and modular systems and line cards are flexible
enough to use at any location in the networks. The platform has seen widespread
use in peering, core, and aggregation networks.  

See more information on Cisco NCS 5500 and 5700 at <https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-736270.html> and 
<https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-744698.html>

Specific information on ZR/ZR+ support can be found at <https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/73x/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-73x/m-zr-zrp.html> 

### ASR 9000 Series 

The ASR 9000 is the most widely deployed SP router in the industry.  It has a
rich heritage dating back almost 20 years, but Cisco continues to innovate on
the ASR 9000 platform. The ASR 9000 series now supports 400G QSFP-DD on a
variety of line cards and the ASR 9903 2.4Tbps 3RU platform.  

See more information on Cisco ASR 9000 at <https://www.cisco.com/c/en/us/products/collateral/routers/asr-9000-series-aggregation-services-routers/data_sheet_c78-501767.html> 

Specific information on ZR/ZR+ support can be found at <https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r7-3/interfaces/configuration/guide/b-interfaces-hardware-component-cg-asr9000-73x/m-zr-zrp.html#Cisco_Concept.dita_59215d6f-1614-4633-a137-161ebe794673> 

### NCS 500 Series 

The 1Tbps N540-24QL16DD-SYS high density router brings QSFP-DD and Routed Optical Networking
ZR/OpenZR+ optics to a flexible access and aggregation platform. Using OpenZR+ optics it allows a 
migration path from 100G to 400G access rings or uplinks when used in an aggregation role.   

See more information on Cisco NCS 540 at <https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-500-series-routers/ncs-540-large-density-router-ds.html>
## Routed Optical Networking Optical Hardware

![](http://xrdocs.io/design/images/ron-hld/ron-optical-hardware.png)


### Network Convergence System 1010 

The NCS 1010 Open Optical Line System (O-OLS) is a next-generation DWDM platform 
available in fixed variants to satisfy building a modern flexible DWDM photonic 
network.  

The NCS 1010 Optical Line Terminal (OLT) uses a twin 33-port WSS architecture
allowing higher scale for either add/drop or express wavelengths. The OLT also
has two LC add/drop ports with integrated fixed gain EDFA to support the
add/drop of lower power optical signals. OLTs are available in models with or
without RAMAN amplification.  NCS 1010 Inline Amplifier nodes are available as
bi-directional EDFA, EDFA with RAMAN in one direction, or bi-directional RAMAN.
Each model of NCS 1010 is also available to support both C and L bands. In Routed Optical Networking 
2.0 ZR and ZR+ optics utilize the C band, but may be used on the same fiber with
L band signals using the NCS 1010 C+L combiner. 

The NCS 1010 utilizes IOS-XR, inheriting the advanced automation and telemetry
features similar to IOS-XR routers.   

**NCS 1010 OLT with RAMAN** 
![](http://xrdocs.io/design/images/ron-hld/ron-1010-olt-raman.png)

**NCS 1010 ILA with RAMAN** 
![](http://xrdocs.io/design/images/ron-hld/ron-1010-ila-raman.png)

The NCS1K-MD32-E/O-C 32-port 150Ghz spaced passive multiplexer is used with the NCS 1010, supporting the 75Ghz 
ZR/ZR+ signals and future higher baud rate signals. The MD-32 contains photodiodes to monitor RX power levels 
on each add/drop port.   

**NCS 1010 MD-32 Passive Filter** 
![](http://xrdocs.io/design/images/ron-hld/ron-1010-md32.png)

The NCS 1010 supports point to point and express DWDM optical topologies in Routed Optical Networking 
2.0. All NCS 1010 services in Routed Optical Networking are managed using Cisco Optical Network
Controller.  

See more information on the NCS 1010 series at <https://www.cisco.com/c/en/us/products/collateral/optical-networking/network-convergence-system-1000-series/network-conver-system-1010-ds.html>

### Network Convergence System 2000 

The NCS 2000 Optical Line System is a flexible platform supporting all modern
optical topologies and deployment use cases. Simple point to point to
multi-degree CDC deployments are all supported as part of Routed Optical
Networking.   

See more information on the NCS 2000 series at <https://www.cisco.com/c/en/us/products/optical-networking/network-convergence-system-2000-series/index.html>  
### Network Convergence System 1000 Multiplexer

The NCS1K-MD-64-C is a new fixed multiplexer designed specifically
for the 400G 75Ghz 400ZR and OpenZR+ wavelengths, allowing up to 25.6Tbps on a
single fiber. 
### Network Convergence System 1001 

The NCS 1001 is utiized in point to point network spans as an amplifier and
optionally protection switch. The NCS 1001 now has specific support for 75Ghz
spaced 400ZR and OpenZR+ wavelengths, with the ability to monitor incoming
wavelengths for power. The 1001 features the ability to determine the proper
amplifier gain setpoints based on the desired user power levels.  

See more information on the NCS 1001 at <https://www.cisco.com/c/en/us/products/collateral/optical-networking/network-convergence-system-1000-series/datasheet-c78-738782.html>

### NCS 2000 and NCS 1001 Hardware 
The picture below does not represent all available hardware on the NCS 2000, 
however does capture the modules typically used in Routed Optical Networking 
deployments.  

![](http://xrdocs.io/design/images/ron-hld/ron-optical-hardware.png)

# Routed Optical Networking Automation 
## Overview 
Routed Optical Networking by definition is a disaggregated optical solution,
creating efficiency by moving coherent endpoints in the router. The solution
requires a new way of managing the network, one which unifies the IP and Optical
layers, replacing the traditional siloed tools used in the past. Real
transformation in operations comes from unifying teams and workflows, rather
than trying to make an existing tool fit a role it was not originally designed
for. Cisco's standards based hierarchical SDN solution allows providers to
manage a multi-vendor Routed Optical Networking solution using standard
interfaces and YANG models.   


## IETF ACTN SDN Framework 
The IETF Action and Control of Traffic Engineered Networks group (ACTN) has
defined a hierarchical controller framework to allow vendors to plug components
into the framework as needed.  The lowest level controller, the Provisioning
Network Controller (PNC), is responsible for managing physical devices. These
controller expose their resources through standard models and interface to a
Hierarchical Controller (HCO), called a Multi-Domain Service Controller (MDSC)
in the ACTN framework.  

Note that while Cisco is adhering to the IETF framework proposed in 
[RFC8453](https://datatracker.ietf.org/doc/html/rfc8453/) , Cisco is supporting the most
widely supported industry standards for controller to controller communication
and service definition. In optical the de facto standard is Transport API from
the ONF for the management of optical line system networks and optical services.
In packet we are leveraging Openconfig device models where possible and IETF
models for packet topology (RFC8345) and xVPN services (L2NM and L3NM) 

![](http://xrdocs.io/design/images/ron-hld/actn-framework.png){:height="90%" width="90%"}
## Cisco's SDN Controller Automation Stack
Aligning to the ACTN framework, Cisco's automation stack includes a
multi-vendor IP domain controller (PNC), optical domain controller (PNC), and
multi-vendor hierarchical controller (HCO/MDSC).    

![](http://xrdocs.io/design/images/ron-hld/cisco-automation-stack.png)

## Cisco Open Automation 
Cisco believes not all providers consume automation in the same way, so we are
dedicated to make sure we have open interfaces at each layer of the network
stack. At the device level, we utilize standard NETCONF, gRPC, and gNMI
interfaces along with native, standard, and public consortium YANG models. There
is no aspect of a Cisco IOS-XR router today not covered by YANG models. At the
domain level we have Cisco's network controllers, which use the same standard
interfaces to communicate with devices and expose standards based NBIs. Our
multi-layer/multi-domain controller likewise uses the same standard interfaces.


![](http://xrdocs.io/design/images/ron-hld/open-automation.png){:height="90%" width="90%"}

## Crosswork Hierarchical Controller 
Responsible for Multi-Layer Automation is the Crosswork Hierarchical Controller. Crosswork Hierarchical Controller is responsible for the following network functions: 

* CW HCO unifies data from the IP and optical networks into a single network
  model. HCO utilizes industry standard IETF topology models for IP and TAPI for
  optical topology and service information. HCO can also leverage legacy EMS/NMS
  systems or use device interrogation.   
* Responsible for managing multi-layer Routed Optical Networking links using a
  single UI.    
*  Providing assurance at the IP and optical layers in a single tool. The
   network model allows users to quickly correlate faults and identify at which
   layer faults have occurred.
* Additional HCO applications include the following  
  - Root Cause Analysis: Quickly correlate upper layer faults to an underlying cause.   
  - Layer Relations: Quickly identify the lower layer resources supporting higher layer network resource or all network resources reliant on a selected 
  lower layer network resource.  
  - Network Inventory: View IP and optical node hardware inventory along with with network resources such as logical links, optical services, and traffic engineering tunnels  
  - Network History: View state changes across all network resources at any point in time  
  - Performance: View historical link utilization  

Please see the following resources for more information on Crosswork HCO. <https://www.cisco.com/c/en/us/products/collateral/cloud-systems-management/crosswork-network-automation/solution-overview-c22-744695.html> 

![](http://xrdocs.io/design/images/ron-hld/hco-multi-layer-circuit-2.png){:height="100%" width="100%"}

## Crosswork Network Controller 
Crosswork Network Controller is a multi-vendor IP domain controller. Crosswork
Network Controller is responsible for the following IP network functions. 

* Collecting Ethernet, IP, RSVP-TE, and SR network information for internal
  applications and exposing northbound via IETF RFC 8345 topology models 
* Collecting traffic information from the network for use with CNC's traffic
  optimization application, Crosswork Optimization Engine 
* Enable Bandwidth Guaranteed Circuit-Style Segment Routing paths using Circuit Style Manager **New in CNC 5.0 / RON 2.1**  
* Perform provisioning of SR-TE, RSVP-TE, L2VPN, and L3VPN using standard
  industry models (IETF TEAS-TE, L2NM, L3NM) via UI or northbound API 
* Visualization and assurance of SR-TE, RSVP-TE, and xVPN services
* Optmization of network resources with its industry first Tactical Traffic Engineering applications 
  to perform traffic when needed. 
* Use additional Crosswork applications to perform telemetry collection/alerting,
  zero-touch provisioning, and automated and assurance network changes  

More information on Crosswork and Crosswork Network Controller can be found at <https://www.cisco.com/c/en/us/products/collateral/cloud-systems-management/crosswork-network-automation/datasheet-c78-743456.html>

## Cisco Optical Network Controller 
Cisco Optical Network Controller (Cisco ONC) is responsible for managing Cisco 
optical line systems and circuit services. Cisco ONC exposes a ONF TAPI northbound 
interface, the de facto industry standard for optical network management. Cisco ONC 
runs as an application on the same Crosswork Infrastructure as CNC.  

More information on Cisco ONC can be found at <https://www.cisco.com/c/en/us/support/optical-networking/optical-network-controller/series.html> 

## Cisco Network Services Orchestrator and Routed Optical Networking ML Core Function Pack 
Cisco NSO is the industry standard for service orchestration and device
configuration management. The RON-ML CFP can be used to fully configure an IP
link between routers utilizing 400ZR/OpenZR+ optics over a Cisco optical line
system using Cisco ONC. This includes IP addressing and adding links to an
existing Ethernet LAG. The CFP can also support optical-only provisioning on the
router to fit into existing optical provisioning workflows.  

# Routed Optical Networking Service Management

## Supported Provisioning Methods
We support multiple ways to provision Routed Optical Networking services based 
on existing provider workflows.  

* [Unified IP and Optical using Crosswork Hierarchical Controller](#crosswork-hco-ui-provisioning)
* [IP router DCO provisioning using Cisco NSO Routed Optical Networking Multi-Layer Function Pack](#nso-ron-ml-cfp-provisioning) 
* [ZR/ZR+ Optics using IOS-XR CLI](#ios-xr-cli-configuration) 
* [Cisco Native Model-driven ZR/ZR+ Optics configuration using Netconf or gNMI](#ios-xr-netconf-configuration)
* [OpenConfig ZR/ZR+ Optics configuration using Netconf or gNMI](#ios-xr-oc-configuration)

## OpenZR+ and 400ZR Properties 
### ZR/ZR+ Supported Frequencies 

The frequency on Cisco ZR/ZR+ transceivers may be set between 191.275Thz and
196.125Thz in increments of 6.25Ghz, supporting flex spectrum applications. To
maximize the available C-Band spectrum, these are the recommended 64
75Ghz-spaced channels, also aligning to the NCS1K-MD-64-C fixed channel add/drop
multiplexer. 


|     |    |    |      |    |    |    |   |
|-----|----|----|------|----|----|----|----|
|196.100|196.025|195.950|195.875|195.800|195.725|195.650|195.575|
|195.500|195.425|195.350|195.275|195.200|195.125|195.050|194.975| 
|194.900|194.825|194.75|194.675|194.600|194.525|194.450|194.375|
|194.300|194.225|194.150|194.075|194.000|193.925|193.850|193.775| 
|193.700|193.625|193.550|193.475|193.400|193.325|193.250|193.175|
|193.100|193.025|192.950|192.875|192.800|192.725|192.650|192.575| 
|192.500|192.425|192.350|192.275|192.200|192.125|192.050|191.975|
|191.900|191.825|191.750|191.675|191.600|191.525|191.450|191.375| 

### Supported Line Side Rate, FEC, and Modulation  

OIF 400ZR transceivers support 400G only with the cFEC FEC type per the OIF
specification. OpenZR+ transceivers can support 100G, 200G, 300G, or 400G line
side rate. See router platform documentation for supported rates on each
platform or line card. OpenZR+ optics can utilize the cFEC type in 400G mode to 
retain compatibility with OIF 400ZR.   
### 50Ghz Spectrum Compatiblity Modes (New) 

Starting in Routed Optical Networking 2.1 and IOS-XR 7.9.1 additional support 
has been added for the 200G-8QAM and 200G-16QAM modes. 200G-8QAM utilizes a 
symbol rate of 40.1Gbaud and 200G-16QAM utilizes a symbol rate of 30.1Gbaud. This 
ensures the signal width is compability with legacy 50Ghz filter optical line 
systems. 100G-QPSK also utilizes a 30.1Gbaud symbol rate.   

**OpenZR+ Supported Configurations** 

| Transceiver Type | Rate | FEC | Modulation | Standard |  
|--------|--------|------|------|------------| 
|QDD-400G-ZR-S|400|cFEC|16QAM|OIF 400ZR|  
|QDD-400G-ZRP-S|400|cFEC|16QAM|OIF 400ZR|  
|QDD-400G-ZRP-S|400|oFEC|16QAM|OpenZR+|  
|QDD-400G-ZRP-S|300|oFEC|8QAM|OpenZR+|  
|QDD-400G-ZRP-S|200|oFEC|QPSK|OpenZR+|  
|QDD-400G-ZRP-S|200|oFEC|8QAM|OpenZR+|  
|QDD-400G-ZRP-S|200|oFEC|16QAM|OpenZR+|  
|QDD-400G-ZRP-S|100|oFEC|QPSK|OpenZR+|  
|DP04QSDD-HE0|400|cFEC|16QAM|OIF 400ZR|  
|DP04QSDD-HE0|400|oFEC|16QAM|OpenZR+|  
|DP04QSDD-HE0|300|oFEC|8QAM|OpenZR+|  
|DP04QSDD-HE0|200|oFEC|QPSK|OpenZR+|  
|DP04QSDD-HE0|200|oFEC|8QAM|OpenZR+|  
|DP04QSDD-HE0|200|oFEC|16QAM|OpenZR+|  
|DP04QSDD-HE0|100|oFEC|QPSK|OpenZR+|  


## Crosswork Hierarchical Controller UI Provisioning 

End-to-End IP+Optical provisioning can be done using Crosswork Hierarchical Controller's GUI IP Link
provisioning. Those familiar with traditional GUI EMS/NMS systems for service
management will have a very familiar experience. Crosswork Hierarchical Controller provisioning will provision
both the router optics as well as the underlying optical network to support the
ZR/ZR+ wavelength.   

### Cross-Layer Link Definition 

End to end provisioning requires first defining the Cross-Layer or Inter-Layer links between the
router ZR/ZR+ optics and the optical line system add/drop ports. This is done
in Crosswork HCO using a UI based "Link Manager" application, used to define 
the Network Media Channel (NMC) interconnection between ZR/ZR+ port and optical 
add/drop port.

The below screenshot shows defined NMC cross-links.   

![](http://xrdocs.io/design/images/ron-hld/ron-hco-cross-layer-link.png)

### Cross-Layer Link Validation (New) 

Starting in RON 2.1 and HCO 7.0 users now have the ability to validate the 
connectivity of an NMC Cross-Layer link. Validation is done by manipulating the 
transmit power of the optics on the routers and continuously monitoring the power
seen on the receive side of the optical line system add/drop port. In RON 2.1 the 
link validation solution is supported using all XR based Cisco routers and NCS 1010 
optical line systems.  Validation can be done for all links or per-link using the 
"Validate Link" option.

Using the Cross-Layer Link Validation is service affecting 
{: .notice--warning}

The screenshot below shows a successful validation.  

![](http://xrdocs.io/design/images/ron-hld/ron-hco-cl-link-validation.png)

### IP Link Provisioning using Crosswork HCO 

Crosswork HCO supports end-to-end multi-layer provisioning of Routed Optical 
Networking circuits, providing a simplified way to provisioning DCO optics in 
the routers and the supporting optical line system OTSiMC channel in a single 
operation.  

HCO also supports separating the router and optical line system provisioning as 
two separate tasks, and also supports router-only provisioning for use cases where either
dark fiber or passive optical components are being used.  

Once the cross layer links are created, the user can then proceed in
provisioning an end to end circuit spanning both IP and optical networks.  The
provisioning UI takes as input the two router endpoints, the associated ZR/ZR+
ports, and the IP addressing or bundle membership of the link. The optical line
system provisioning is abstracted from the user, simplifying the end to end
workflow. The frequency and power is automatically derived by Cisco Optical
Network Controller based on the add/drop port and returned as a parameter to be
used in router optics provisioning.  

![](http://xrdocs.io/design/images/ron-hld/ron-hco-ip-link-provisioning-2.png)

### Operational Discovery 
The Crosswork Hierarchical Controller provisioning process also performs a discovery phase to ensure the
service is operational before considering the provisioning complete. If
operational discovery fails, the end to end service will be rolled back.  
## NSO RON-ML CFP Provisioning
Providers familiar with using Cisco Network Service Orchestrator have an option
to utilize NSO to provision optical and IP layer configuration for ZR/ZR+ router
DCOs.  
Cisco has created the Routed Optical Network Multi-Layer Core Function Pack,
RON-ML CFP to perform the provisioning of Routed Optical Networking services on
the router endpoints. The aforementioned Crosswork HCO provisioning utilizes the
RON-ML CFP to perform end device provisioning.  

Please see the Cisco Routed Optical Networking RON-ML CFP documentation for more
details.  

### RON-ML End to End Service 
The RON-ML service is responsible for router DCO provisioning. All IP layer configuration 
such as bundle membership and IP addressing is optional, allowing potentially different 
teams to perform optical parameter provisioning vs. Ethernet/IP layer configuration.  

### RON-ML API Provisioning
Use the following URL for NSO RESTCONF provisioning using a PATCH operation:
```http://<nso host>:<port>/restconf/data``` 

**Provisioning ZR+ optics and adding interface to Bundle-Ether 100 interface**  

```json
{
    "cisco-ron-cfp:ron": {
      "ron-ml": [
        {
          "name": "E2E_Bundle_ZRP_ONC57_2",
          "mode": "transponder",
          "bandwidth": "400",
          "circuit-id": "E2E Bundle ONC-57 S9|chan11 - S10|chan11",
          "grid-type": "100mhz-grid",
          "end-point": [
            {
              "end-point-device": "ron-8201-1",
              "terminal-device-optical": {
                "line-port": "0/0/0/11",
                "transmit-power": -100
              },
              "ols-domain": {
                "end-point-state": "UNLOCKED"
              },
              "terminal-device-packet": {
                "bundle": [
                  {
                    "id": 100
                  }
                ],
                "interface": [
                  {
                    "index": 0,
                    "membership": {
                      "bundle-id": 100,
                      "mode": "active"
                    }
                  }
                ]
              }
            },
            {
              "end-point-device": "ron-8201-2",
              "terminal-device-optical": {
                "line-port": "0/0/0/11",
                "transmit-power": -100
              },
              "ols-domain": {
                "end-point-state": "UNLOCKED"
              },
              "terminal-device-packet": {
                "bundle": [
                  {
                    "id": 100
                  }
                ],
                "interface": [
                  {
                    "index": 0,
                    "membership": {
                      "bundle-id": 100,
                      "mode": "active"
                    }
                  }
                ]
              }
            }
          ]
        }
      ]
    }
  }
```
## IOS-XR CLI Configuration
Configuring the router portion of the Routed Optical Networking link is very
simple.  All optical configuration related to the ZR/ZR+ optics configuration is
located under the optics controller relevent to the faceplate port. Default
configuration the optics will be in an up/up state using a frequency of
193.10Thz.  The default transmit power is dependent on the optics type.  The default 
transmit power for the QDD-400G-ZR-S (OIF 400ZR) and QDD-400G-ZRP-S is -10 dBm. 
The default transmit power for the High-Power ZR+ DP04QSDD-HE0 is 0 dBm.  

The basic configuration with a specific frequency of 195.65 Thz is located below,   
the only required component is the bolded channel frequency setting.  

**ZR/ZR+ Optics Configuration** 

<div class="highlighter-rouge">
<pre class="highlight">
controller Optics0/0/0/20
 transmit-power -100
 <b>dwdm-carrier 100MHz-grid frequency 1956500</b>
 logging events link-status
</pre>
</div>

## Model-Driven Configuration using IOS-XR Native Models using NETCONF or gNMI  
All configuration performed in IOS-XR today can also be done using NETCONF/YANG. The following payload exhibits the models 
and configuration used to perform router optics provisioning. This is a more complete example showing the FEC, power, and 
frequency configuration. .  

**Note in Release 2.0 using IOS-XR 7.7.1 the newer IOS-XR Unified Models are utilized for provisioning**

```xml
<data xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<controllers xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-interface-cfg">
    <controller>
        <controller-name>Optics0/0/0/0</controller-name>
        <transmit-power xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-cont-optics-cfg">-115</transmit-power>
        <fec xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-cont-optics-cfg">OFEC</fec>
        <dwdm-carrier xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-cont-optics-cfg">
          <grid-100mhz>
            <frequency>1913625</frequency>
          </grid-100mhz>
        </dwdm-carrier>
        <dac-rate xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-um-dac-rate-cfg">1x1.25</dac-rate>
      </controller>
</controllers> 

</data>
```

## Model-Driven Configuration using OpenConfig Models
Starting on Release 2.0 all IOS-XR 7.7.1+ routers supporting ZR/ZR+ optics can be 
configured using OpenConfig models. Provisioning utilizes the openconfig-terminal-device 
model and its extensions to the openconfig-platform model to support DWDM configuration 
parameters. 

Below is an example of an OpenConfig payload to configure ZR/ZR+ optics port 0/0/0/20 with a 
300G trunk rate with frequency 195.20 THz.  

Please visit the blog at <https://xrdocs.io/design/blogs/zr-openconfig-mgmt> for in depth information 
about configuring and monitoring ZR/ZR+ optics using OpenConfig models.  

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

# Routed Optical Networking Assurance 

## Crosswork Hierarchical Controller
### Multi-Layer Path Trace 
Using topology and service data from both the IP and Optical network CW HCO can
display the full service from IP services layer to the physical fiber.  Below is
an example of the "waterfall" trace view from the OTS (Fiber) layer to the
Segment Routing TE layer across all layers. CW HCO identifies specific Routed
Optical Networking links using ZR/ZR+ optics as seen by the ZRC (ZR Channel) and
ZRM (ZR Media) layers from the 400ZR specification.  

![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-trace-2.png){:height="100%" width="100%"}

When faults occur at a specific layer, faults will be highlighted in red,
quickly identifying the layer a fault has occurred. In this case we can see the
fault has occurred at an optical layer, but is not a fiber fault. Having the
ability to pinpoint the fault layer even within a specific domain is a powerful
way to quickly determine the root cause of the fault.  

![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-trace-fault-2.png){:height="100%" width="100%"}

### Routed Optical Networking Link Assurance 
The Link Assurance application allows users to view a network link and all of 
its dependent layers. This includes Routed Optical Networking multi-layer 
services. In addition to viewing layer information, fault and telemetry information 
is also available by simply selecting a link or port.   

#### ZRM Layer TX/RX Power 
![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-2.png){:height="100%" width="100%"}

#### ZRC Layer BER and Q-Factor / Q-Margin  
![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-3.png){:height="100%" width="100%"}

Optionally the user can see graphs of collected telemetry data to quickly identify trends or changes in specific operational 
data. Graphs of collected performance data is accessed using the "Performance" tab when a link or port is selected.   

#### OTS Layer RX/TX Power Graph
![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-graph-2.png){:height="100%" width="100%"}

#### Event Monitoring 
Crosswork HCO records any transition of a network resource between up/down operational states. This is reflected in the 
Link Assurance tool under the "Events" tab. 

![](http://xrdocs.io/design/images/ron-hld/ron-hco-link-assurance-events.png){:height="100%" width="100%"}
## IOS-XR CLI Monitoring of ZR400/OpenZR+ Optics

### Optics Controller 
The optics controller represents the physical layer of the optics.  In the case
of ZR/ZR+ optics this includes the frequency information, RX/TX power, OSNR, and
other associated physical layer information.   

```
RP/0/RP0/CPU0:ron-8201-1#show controllers optics 0/0/0/20
Thu Jun  3 15:34:44.098 PDT

 Controller State: Up
 Transport Admin State: In Service
 Laser State: On
 LED State: Green
 FEC State: FEC ENABLED

 Optics Status

         Optics Type:  QSFPDD 400G ZR
         DWDM carrier Info: C BAND, MSA ITU Channel=10, Frequency=195.65THz,
         Wavelength=1532.290nm

         Alarm Status:
         -------------
         Detected Alarms: None

         LOS/LOL/Fault Status:

         Alarm Statistics:

         -------------
         HIGH-RX-PWR = 0            LOW-RX-PWR = 0
         HIGH-TX-PWR = 0            LOW-TX-PWR = 4
         HIGH-LBC = 0               HIGH-DGD = 1
         OOR-CD = 0                 OSNR = 10
         WVL-OOL = 0                MEA  = 0
         IMPROPER-REM = 0
         TX-POWER-PROV-MISMATCH = 0
         Actual TX Power = -7.17 dBm
         RX Power = -9.83 dBm
         RX Signal Power = -9.18 dBm
         Frequency Offset = 9 MHz
         Baud Rate =  59.8437500000 GBd
         Modulation Type: 16QAM
         Chromatic Dispersion 6 ps/nm
         Configured CD-MIN -2400 ps/nm  CD-MAX 2400 ps/nm
         Second Order Polarization Mode Dispersion = 34.00 ps^2
         Optical Signal to Noise Ratio = 35.50 dB
         Polarization Dependent Loss = 1.20 dB
         Polarization Change Rate = 0.00 rad/s
         Differential Group Delay = 2.00 ps

```
Performance Measurement Data 

```
RP/0/RP0/CPU0:ron-8201-1#show controllers optics 0/0/0/20 pm current 30-sec optics 1
Thu Jun  3 15:39:40.428 PDT

Optics in the current interval [15:39:30 - 15:39:40 Thu Jun 3 2021]

Optics current bucket type : Valid
             MIN       AVG       MAX      Operational      Configured      TCA   Operational      Configured     TCA
                                          Threshold(min)   Threshold(min) (min) Threshold(max)   Threshold(max) (max)
LBC[% ]      : 0.0       0.0       0.0      0.0               NA              NO   100.0            NA              NO
OPT[dBm]     : -7.17     -7.17     -7.17    -15.09            NA              NO   0.00             NA              NO
OPR[dBm]     : -9.86     -9.86     -9.85    -30.00            NA              NO   8.00             NA              NO
CD[ps/nm]    : -489      -488      -488     -80000            NA              NO   80000            NA              NO
DGD[ps ]     : 1.00      1.50      2.00     0.00              NA              NO   80.00            NA              NO
SOPMD[ps^2]  : 28.00     38.80     49.00    0.00              NA              NO   2000.00          NA              NO
OSNR[dB]     : 34.90     35.12     35.40    0.00              NA              NO   40.00            NA              NO
PDL[dB]      : 0.70      0.71      0.80     0.00              NA              NO   7.00             NA              NO
PCR[rad/s]   : 0.00      0.00      0.00     0.00              NA              NO   2500000.00       NA              NO
RX_SIG[dBm]  : -9.23     -9.22     -9.21    -30.00            NA              NO   1.00             NA              NO
FREQ_OFF[Mhz]: -2        -1        4        -3600             NA              NO   3600             NA              NO
SNR[dB]      : 16.80     16.99     17.20    7.00              NA              NO   100.00           NA              NO
```

### Coherent DSP Controller
The coherent DSP controller represents the framing layer of the optics. It 
includes Bit Error Rate, Q-Factor, and Q-Margin information. 

```
RP/0/RP0/CPU0:ron-8201-1#show controllers coherentDSP 0/0/0/20
Sat Dec  4 17:24:38.245 PST

Port                                            : CoherentDSP 0/0/0/20
Controller State                                : Up
Inherited Secondary State                       : Normal
Configured Secondary State                      : Normal
Derived State                                   : In Service
Loopback mode                                   : None
BER Thresholds                                  : SF = 1.0E-5  SD = 1.0E-7
Performance Monitoring                          : Enable
Bandwidth                                       : 400.0Gb/s

Alarm Information:
LOS = 10        LOF = 0 LOM = 0
OOF = 0 OOM = 0 AIS = 0
IAE = 0 BIAE = 0        SF_BER = 0
SD_BER = 0      BDI = 0 TIM = 0
FECMISMATCH = 0 FEC-UNC = 0     FLEXO_GIDM = 0
FLEXO-MM = 0    FLEXO-LOM = 3   FLEXO-RDI = 0
FLEXO-LOF = 5
Detected Alarms                                 : None

Bit Error Rate Information
PREFEC  BER                                     : 1.7E-03
POSTFEC BER                                     : 0.0E+00
Q-Factor                                        : 9.30 dB
Q-Margin                                        : 2.10dB


FEC mode                                        : C_FEC
```

Performance Measurement Data 
```
RP/0/RP0/CPU0:ron-8201-1#show controllers coherentDSP 0/0/0/20 pm current 30-sec fec
Thu Jun  3 15:42:28.510 PDT

g709 FEC in the current interval [15:42:00 - 15:42:28 Thu Jun 3 2021]

FEC current bucket type : Valid
    EC-BITS   : 20221314973             Threshold : 83203400000            TCA(enable)  : YES
    UC-WORDS  : 0                       Threshold : 5                      TCA(enable)  : YES

                                      MIN       AVG        MAX      Threshold      TCA     Threshold     TCA
                                                                       (min)     (enable)    (max)     (enable)
PreFEC BER                     :   1.5E-03   1.5E-03   1.6E-03         0E-15        NO       0E-15        NO
PostFEC BER                    :     0E-15     0E-15     0E-15         0E-15        NO       0E-15        NO
Q[dB]                          :      9.40      9.40      9.40          0.00        NO        0.00        NO
Q_Margin[dB]                   :      2.20      2.20      2.20          0.00        NO        0.00        NO
```

## EPNM Monitoring of Routed Optical Networking 
Evolved Programmable Network Manager, or EPNM, can also be used to monitor router ZR/ZR+ performance measurement data 
and display device level alarms when faults occur. EPNM stores PM and alarm data for historical analysis.  

### EPNM Chassis View of DCO Transceivers 
The following shows a chassis view of a Cisco 8201 router. The default view is to show all active alarms on the 
device and its components. Clicking on a specific component will give information on the component and narrow 
the scope of alarms and data.   

#### Chassis View 

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-chassis-view.png){:height="100%" width="100%"}

#### Interface/Port View 

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-interface-view.png){:height="100%" width="100%"}

### EPNM DCO Performance Measurement 
EPNM continuously monitors and stores PM data for DCO optics for important KPIs such as TX/RX power, BER, and Q values.  The screenshots 
below highlight monitoring. While EPNM stores historical data, clicking on a speciic KPI will enable realtime monitoring by polling for 
data every 20 seconds.  

#### DCO Physical Layer PM KPIs  

The following shows common physical layer KPIs such as OSNR and RX/TX power.  This is exposed by monitoring the Optics layer of the interface.   
DCO.    

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-optics-phy-pm.png){:height="100%" width="100%"}

The following shows common framing layer KPIs such as number of corrected words per interval and  (BIEC) Bit Error Rate. This is exposed by monitoring the CoherentDSP 
layer of the interface.   

![](http://xrdocs.io/design/images/ron-hld/ron-epnm-optics-dsp-pm.png){:height="100%" width="100%"}

# Cisco IOS-XR Model-Driven Telemetry for Routed Optical Networking Monitoring 


All operational data on IOS-XR routers and optical line systems can be monitored using streaming telemetry 
based on YANG models. Routed Optical Networking is no different, so a wealth of 
information can be streamed from the routers in intervals as low as 5s.  

## ZR/ZR+ DCO Telemetry 

The following represents a list of validated sensor paths useful for monitoring
the DCO optics in IOS-XR and the data fields available within these
sensor paths.  Note PM fields also support 15m and 24h paths in addition to the 
30s paths shown in the table below.  

| Sensor Path | Fields |
|-------------|--------| 
|Cisco-IOS-XR-controller-optics-oper:optics-oper/optics-ports/optics-port/optics-info | alarm-detected, baud-rate, dwdm-carrier-frequency, controller-state, laser-state, optical-signal-to-noise-ratio, temperature, voltage |
|Cisco-IOS-XR-controller-optics-oper:optics-oper/optics-ports/optics-port/optics-lanes/optics-lane|receive-power, receive-signal-power, transmit-power|
|Cisco-IOS-XR-controller-otu-oper:otu/controllers/controller/info|bandwidth, ec-value, post-fec-ber, pre-fec-ber, qfactor, qmargin, uc | 
|Cisco-IOS-XR-pmengine-oper:performance-management/optics/optics-ports/optics-port/optics-current/optics-second30/optics-second30-optics/optics-second30-optic|dd__average, dgd__average, opr__average, opt__average, osnr__average, pcr__average, pmd__average, rx-sig-pow__average, snr__average, sopmd__average| 
|Cisco-IOS-XR-pmengine-oper:performance-management/otu/otu-ports/otu-port/otu-current/otu-second30/otu-second30fecs/otu-second30fec|ec-bits__data, post-fec-ber__average, pre-fec-ber__average, q__average, qmargin__average, uc-words__data |  


## NCS 1010 Optical Line System Monitoring  

The following represents a list of validated sensor paths useful for monitoring
the different optical resources on the NCS 1010 OLS. The OTS controller represents 
the lowest layer port interconnecting optical elements. The NCS 1010 supports 
per-channel monitoring, exposed as the OTS-OCH  

| Sensor Path | Fields |
|-------------|--------| 
|Cisco-IOS-XR-controller-ots-oper:ots-oper/ots-ports/ots-port/ots-info | total-tx-power, total-rx-power, transmit-signal-power, receive-signal-power, agress-ampi-gain, ingress-ampli-gain, controller-state |
|Cisco-IOS-XR-controller-ots-och-oper:ots-och-oper/ots-och-ports/ots-och-port/ots-och-info | total-tx-power, total-rx-power, transport-admin-state, line-channel, add-drop-channel | 
|Cisco-IOS-XR-controller-oms-oper | rx-power, tx-power, controller-state, led-state | 
|Cisco-IOS-XR-controller-och-oper:och-oper/och-ports/och-port/och-info |channel-frequency, channel-wavelength, controller-state, rx-power, tx-power, channel-width, led-state| 
|Cisco-IOS-XR-pmengine-oper:performance-management/optics/optics-ports | opr, opt, opr-s, opt-s| 
|Cisco-IOS-XR-olc-oper:olc/span-loss-ctrlr-tables/span-loss-ctrlr-table | neighbor-rid, rx-span-loss, tx-span-loss, name | 


## Open-source Monitoring 
Cisco model-driven telemetry along with the open source collector Telegraf and the open source dashboard software 
Grafana can be used to quickly build powerful dashboards to monitor ZR/ZR+ and NCS 1010 OLS performance.  

![](http://xrdocs.io/design/images/ron-hld/ron-telemetry-grafana.png)

![](http://xrdocs.io/design/images/ron-hld/ron-telemetry-grafana-1010.png)

<br>

---
# Additional Resources
## Cisco Routed Optical Networking 2.0 Solution Guide

<https://www.cisco.com/content/en/us/td/docs/optical/ron/2-0/solution/guide/b-ron-solution-20.html>
## Cisco Routed Optical Networking Home 
* <https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking.html> 
## Cisco Routed Optical Networking Tech Field Day 
* Solution Overview: <https://techfieldday.com/video/build-your-network-with-cisco-routed-optical-networking-solution/> 
* Automation Demo: <https://techfieldday.com/video/cisco-routed-optical-networking-solution-demo/> 

## Cisco Champion Podcasts 
* Cisco Routed Optical Networking Solution for the Next Decade <https://smarturl.it/CCRS8E24> 
* Simplify Network Operations with Crosswork Hierarchical Controller: <https://smarturl.it/CCRS8E48 >

# Appendix A

## Acronyms 

|     |     | 
|-----|-----| 
| DWDM | Dense Waveform Division Multiplexing | 
| OADM | Optical Add Drop Multiplexer |
| FOADM | Fixed Optical Add Drop Multiplexer |
| ROADM | Reconfigurable Optical Add Drop Multiplexer |
| DCO | Digital Coherent Optics | 
| FEC | Forward Error Correction | 
| OSNR| Optical Signal to Noise Ratio | 
| BER | Bit Error Rate | 



## DWDM Network Hardware Overview  

![](http://xrdocs.io/design/images/ron-hld/ron-design-optical-components.png)
### Optical Transmitters and Receivers 
Optical transmitters provide the source signals carried across the DWDM network.
They convert digital electrical signals into a photonic light stream on a
specific wavelength. Optical receivers detect pulses of light and and convert
signals back to electrical signals. In Routed Optical Networking, digital 
coherent QSFP-DD OpenZR+ and 400ZR transceivers in routers are used as optical 
transmitters and receivers.  

### Multiplexers/Demultiplexers  
Multiplexers take multiple wavelengths on separate fibers and combine them into
a single fiber. The output of a multiplexer is a composite signal.
Demultiplexers take composite signals that compatible multiplexers generate and
separate the individual wavelengths into individual fibers.

### Optical Amplifiers 
Optical amplifiers amplify an optical signal. Optical amplifiers increase the
total power of the optical signal to enable the signal transmission across
longer distances. Without amplifiers, the signal attenuation over longer
distances makes it impossible to coherently receive signals. We use different
types of optical amplifiers in optical networks. For example: preamplifiers,
booster amplifiers, inline amplifiers, and optical line amplifiers.

### Optical add/drop multiplexers (OADMs)
OADMs are devices capable of adding one or more DWDM channels into or dropping
them from a fiber carrying multiple channels. 

### Reconfigurable optical add/drop multiplexers (ROADMs)
ROADMs are programmable versions of OADMs. With ROADMs, you can change the
wavelengths that are added or dropped. ROADMs make optical networks flexible and
easily modifiable.


