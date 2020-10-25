---
published: true
date: '2020-10-05 16:54 -0500'
title: Converged SDN Transport Implementation Guide
position: hidden 
author: Phil Bedard 
excerpt: Converged SDN Transport Implementation Guide - 3.5 
tags:
  - iosxr
  - cisco
  - 5G 
  - cin  
  - rphy 
  - Metro
  - Design
---

{% include toc %}

# Targets

  - Hardware:
    
      - ASR 9000 as Centralized Provider Edge (C-PE) router  
      - NCS 5500 and NCS 55A2 as Aggregation and Pre-Aggregation router  
      - NCS 5500 as P core router  
      - ASR 920, NCS 540, and NCS 5500 as Access Provider Edge (A-PE) 
      - cBR-8 CMTS with 8x10GE DPIC for Remote PHY 
      - Compact Remote PHY shelf with three 1x2 Remote PHY Devices (RPD)   

  - Software:
    
      - IOS-XR 6.6.3 on ASR 9000, NCS 540, NCS 5500, and NCS 55A2 routers  
      - IOS-XE 16.8.1 on ASR 920
      - IOS-XE 16.10.1f on cBR-8 

  - Key technologies
    
      - Transport: End-To-End Segment-Routing
      - Network Programmability: SR- TE Inter-Domain LSPs with On-Demand
        Next Hop
      - Network Availability: TI-LFA/Anycast-SID
      - Services: BGP-based L2 and L3 Virtual Private Network services
        (EVPN and L3VPN/mVPN)
      - Network Timing: G.8275.1 and G.8275.2
      - Network Assurance: 802.1ag  

# Testbed Overview

[](https://xrdocs.io/design/images/cmfi/image1.png)

_Figure 1: Compass Converged SDN Transport High Level Topology_

![](https://xrdocs.io/design/images/cmfi/cst-topology.png)

_Figure 2: Testbed Physical Topology_

![](https://xrdocs.io/design/images/cmfi/image3.png)

_Figure 3: Testbed Route-Reflector and SR-PCE physical connectivity_

![](https://xrdocs.io/design/images/cmfi/image4.png)

_Figure 4: Testbed IGP Domains_

## Devices

**Access PE (A-PE) Routers**

  - Cisco NCS5501-SE (IOS-XR) – A-PE7
  - Cisco NCS540 (IOS-XR) - A-PE1, A-PE2, A-PE3, A-PE8 
  - Cisco ASR920 (IOS-XE) – A-PE4, A-PE5, A-PE6, A-PE9

**Pre-Aggregation (PA) Routers**

  - Cisco NCS5501-SE (IOS-XR) – PA3, PA4  

**Aggregation (AG) Routers**

  - Cisco NCS5501-SE (IOS-XR) – AG2, AG3, AG4 
  - Cisco NCS 560-4 w/RSP-4E (IOS-XR) - AG1 


**High-scale Provider Edge Routers**

  - Cisco ASR9000 w/Tomahawk Line Cards (IOS-XR) – PE1, PE2, PE3, PE4 

**Area Border Routers (ABRs)**

  - Cisco ASR9000 (IOS-XR) – PE3, PE4 
  - Cisco 55A2-MOD-SE - PA2 
  - Cisco NCS540 - PA1 

**Service and Transport Route Reflectors (RRs)**

  - Cisco IOS XRv 9000 – tRR1-A, tRR1-B, sRR1-A, sRR1-B, sRR2-A, sRR2-B,
    sRR3-A, sRR3-B

**Segment Routing Path Computation Element (SR-PCE)**

  - Cisco IOS XRv 9000 – SRPCE-A1-A, SRPCE-A1-B, SRPCE-A2-A, SRPCE-A2-A, SRPCE-CORE-A, SRPCE-CORE-B  

# Key Resources to Allocate  
- IP Addressing 
  - IPv4 address plan
  - IPv6 address plan, recommend dual plane day 1
    - Plan for SRv6 in the future
- Color communities for ODN   
- Segment Routing Blocks
  - SRGB (segment-routing address block) 
   - Keep in mind anycast SID for ABR node pairs 
   - Allocate 3 SIDs for potential future Flex-algo use 
  - SRLB (segment routing local block) 
    - Local significance only
    - Can be quite small and re-used on each node
- IS-IS unique instance identifiers for each domain 


# Role-Based Router Configuration

## IOS-XR Router Configuration 

### Underlay physical interface configuration with BFD  

<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/10
 bfd mode ietf
 bfd address-family ipv4 timers start 180
 bfd address-family ipv4 multiplier 3
 bfd address-family ipv4 destination 10.1.2.1
 bfd address-family ipv4 fast-detect
 bfd address-family ipv4 minimum-interval 50
 mtu 9216
 ipv4 address 10.15.150.1 255.255.255.254
 ipv4 unreachables disable
 load-interval 30
 dampening
</pre>
</div>

### MPLS Performance Measurement 

#### Interface delay metric dynamic configuration 
Starting with CST 3.5 we now support end to end dynamic link delay measurements across all IOS-XR nodes. The feature in IOS-XR is called Performance Measurement and all configuration is found under the performance-measurement configuration hierarchy.  There are a number of configuration options utilized when configuring performance measurement, but the below configuration will enable one-way delay 
measurements on physical links.  The probe measurement-mode options are either <b>one-way</b> or <b>two-way</b>  One-way mode requires nodes be time synchronized to a common PTP clock, and should be used if available.  In the absence of a common PTP clock, two-mode can be used which calculates the one-way delay using multiple timestamps at the querier and responder.  

The advertisement options specify when the advertisements are made into the IGP. The periodic interval sets the minimum interval, with the threshold setting the difference required to advertise a new delay value.  The accelerated threshold option sets a percentage change required to trigger and advertisement prior to the periodic interval timer expiring.  Performance measurement takes a series of measurements within each computation interval and uses this information to derive the min, max, and average link delay.  

Full documentation on Performance Measurement can be found at:  https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/72x/b-segment-routing-cg-ncs5500-72x/configure-performance-measurement.html

Please note while this is the IOS-XR 7.2.1 documentation it also applies to IOS-XR 7.1.2.  


<div class="highlighter-rouge">
<pre class="highlight">
performance-measurement
 interface TenGigE0/0/0/20
  delay-measurement
  !
 !
 interface TenGigE0/0/0/21
  delay-measurement
  !
 !
 protocol twamp-light
  measurement delay
   unauthenticated
    querier-dst-port 12345
   !
  !
 !
 delay-profile interfaces
  advertisement
   accelerated
    threshold 25
   !
   periodic
    interval 120
    threshold 10
   !
  !
  probe
   measurement-mode two-way
   protocol twamp-light
   computation-interval 60
  !
 !
!
end
</pre>
</div> 

#### Interface delay metric static configuration  

In the absence of dynamic realtime one-way latency monitoring for physical interfaces, the interface delay can be set manually. The one-way delay measurement value is used when computing SR Policy paths with the "latency" constraint type. The configured value is advertised in the IGP using extensions defined in RFC 7810, and advertised to the PCE using BGP-LS extensions. Keep in mind the delay metric value is defined in microseconds, so if you are mixing dynamic computation with static values they should be set appropriately.  

<div class="highlighter-rouge">
<pre class="highlight">
performance-measurement
 interface TenGigE0/0/0/10
  delay-measurement
   advertise-delay 15000
 interface TenGigE0/0/0/20
  delay-measurement
   advertise-delay 10000
</pre>
</div>

### SR-MPLS Transport  

#### Segment Routing SRGB and SRLB Definition 
It's recommended to first configure the Segment Routing Global Block (SRGB) across all nodes needing connectivity between each other. In most instances a single SRGB will be used across the entire network. In a SR MPLS deployment the SRGB and SRLB correspond to the label blocks allocated to SR. IOS-XR has a maximum configurable SRGB limit of 512,000 labels, however please consult platform-specific documentation for maximum values. The SRLB corresponds to the labels allocated for SIDs local to the node, such as Adjacency-SIDs. It is recommended to configure the same SRLB block across all nodes. The SRLB must not overlap with the SRGB.  The SRGB and SRLB are configured in IOS-XR with the following configuration:   

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 global-block 16000 23999
 local-block 15000 15999
</pre> 
</div>

#### IGP protocol (ISIS) and Segment Routing MPLS configuration

**Key chain global configuration for IS-IS authentication**
<div class="highlighter-rouge">
<pre class="highlight">
key chain ISIS-KEY
 key 1
 accept-lifetime 00:00:00 january 01 2018 infinite
 key-string password 00071A150754
 send-lifetime 00:00:00 january 01 2018 infinite
 cryptographic-algorithm HMAC-MD5
</pre> 
</div>

#### IS-IS router configuration

All routers, except Area Border Routers (ABRs), are part of one IGP
domain and L2 area (ISIS-ACCESS or ISIS-CORE). Area border routers  
run two IGP IS-IS processes (ISIS-ACCESS and ISIS-CORE).  Note that Loopback0 is part of both IGP processes.

<div class="highlighter-rouge">
<pre class="highlight">
router isis ISIS-ACCESS
 set-overload-bit on-startup 360
 is-type level-2-only
 net 49.0001.0101.0000.0110.00
 nsr
 nsf cisco
 log adjacency changes
 lsp-gen-interval maximum-wait 5000 initial-wait 5 secondary-wait 100
 lsp-refresh-interval 65000
 max-lsp-lifetime 65535
 lsp-password keychain ISIS-KEY
 address-family ipv4 unicast
  metric-style wide
  advertise link attributes
  spf-interval maximum-wait 1000 initial-wait 5 secondary-wait 100
  segment-routing mpls
  spf prefix-priority high tag 1000
  maximum-redistributed-prefixes 100 level 2
 ! 
 address-family ipv6 unicast
  metric-style wide
  spf-interval maximum-wait 5000 initial-wait 50 secondary-wait 200
  maximum-redistributed-prefixes 100 level 2
</pre>
</div>

**Note:** ABR Loopback 0 on domain boundary is part of both IGP processes together with same “prefix-sid absolute” value

**Note:** The prefix SID can be configured as either _absolute_ or _index_.  The _index_ configuration is required for interop with nodes using a different SRGB. 

#### IS-IS Loopback and node SID configuration
<div class="highlighter-rouge">
<pre class="highlight">
 interface Loopback0
  ipv4 address 100.0.1.50 255.255.255.255
  address-family ipv4 unicast
   <b>prefix-sid absolute 16150</b>
   tag 1000 
</pre>
</div>

#### Anycast SID ABR node configuration 

Anycast SIDs are SIDs existing on two more ABR nodes to offer a redundant fault tolerant path for traffic between Access PEs and remote PE devices. 
In CST 3.5 and above, anycast SID paths can either be manually configured on the head-end or computed by the SR-PCE. When SR-PCE computes a path it will 
inspect the topology database to ensure the next SID in the computed segment list is reachable from all anycast nodes. If not, the anycast SID will not be 
used. The same IP address and prefix-sid must be configured on all shared anycast nodes, with the n-flag clear option set.  Note when anycast SID path computation is 
used with SR-PCE, only IGP metrics are supported.   

**IS-IS Configuration for Anycast SID**
<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS 
 interface Loopback100
  ipv4 address 100.100.100.1 255.255.255.255
  address-family ipv4 unicast
   <b>prefix-sid absolute 16150 n-flag clear</b>
   tag 1000 
</pre>
</div>

**Conditional IGP Loopback advertisement** 
While not the only use case for conditional advertisement, it is a required component when using anycast SIDs with static segment list. Conditional advertisement will 
not advertise the Loopback interface if certain routes are not found in the RIB. If the anycast Loopback is withdrawn, the segment list will be considered invalid on the 
head-end node.  The conditional prefixes should be all or a subset of prefixes from the adjacent IGP domain.  

<div class="highlighter-rouge">
<pre class="highlight">
route-policy check
 if rib-has-route in async remote-prefixes
   pass 
   endif 
 end-policy

prefix-set remote-prefixes 
  100.0.2.52, 
  100.0.2.53
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS  
 interface Loopback100 
 address-family ipv4 unicast 
 <b>advertise prefix route-policy check</b>
</pre>
</div>


#### IS-IS logical interface configuration with TI-LFA

It is recommended to use manual adjacency SIDs. A _protected_ SID is eligible for backup path computation, meaning if a packet ingresses the node with the label a backup path will be provided in case of a link failure. In the case of having multiple adjacencies between the same two nodes, use the same adjacency-sid on each link. Unnumbered interfaces are configured using the same configuration.  

<div class="highlighter-rouge">
<pre class="highlight">
 interface TenGigE0/0/0/10
  point-to-point
  hello-password keychain ISIS-KEY
  address-family ipv4 unicast
   fast-reroute per-prefix
   fast-reroute per-prefix ti-lfa
   adjacency-sid absolute 15002 protected
   metric 100
  ! 
  address-family ipv6 unicast
   fast-reroute per-prefix 
   fast-reroute per-prefix ti-lfa 
   metric 100 
</pre>
</div>

### MPLS Segment Routing Traffic Engineering (SRTE) configuration
The following configuration is done at the global ISIS configuration level and should be performed for all IOS-XR nodes.   

<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
 address-family ipv4 unicast
  mpls traffic-eng level-2-only
  mpls traffic-eng router-id Loopback0
</pre>
</div>

#### MPLS Segment Routing Traffic Engineering (SRTE) TE metric configuration  

The TE metric is used when computing SR Policy paths with the "te" or "latency" constraint type.  The TE metric is carried as a TLV within the TE opaque LSA distributed across the IGP area and to the PCE via BGP-LS.  
The TE metric is used in the CST 5G Transport use case.  If no TE metric is defined the local CSPF or PCE will utilize the IGP metric.   

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  interface TenGigE0/0/0/6
   metric 1000
</pre>
</div>


## IOS-XE Nodes - SR-MPLS Transport 
    
### Segment Routing MPLS configuration

<div class="highlighter-rouge">
<pre class="highlight">
mpls label range 6001 32767 static 16 6000

segment-routing mpls
 !
 set-attributes
  address-family ipv4
   sr-label-preferred
  exit-address-family
 !
 global-block 16000 24999
 !
</pre> 
</div> 

### Prefix-SID assignment to loopback 0 configuration

<div class="highlighter-rouge">
<pre class="highlight">
 connected-prefix-sid-map
  address-family ipv4
   100.0.1.51/32 index 151 range 1
  exit-address-family
 !
</pre> 
</div> 

### IGP protocol (ISIS) with Segment Routing MPLS configuration

<div class="highlighter-rouge">
<pre class="highlight">
key chain ISIS-KEY
 key 1
  key-string cisco
   accept-lifetime 00:00:00 Jan 1 2018 infinite
   send-lifetime 00:00:00 Jan 1 2018 infinite
!
router isis ACCESS
 net 49.0001.0102.0000.0254.00
 is-type level-2-only
 authentication mode md5
 authentication key-chain ISIS-KEY
 metric-style wide
 fast-flood 10
 set-overload-bit on-startup 120
 max-lsp-lifetime 65535
 lsp-refresh-interval 65000
 spf-interval 5 50 200
 prc-interval 5 50 200
 lsp-gen-interval 5 5 200
 log-adjacency-changes
 segment-routing mpls
 segment-routing prefix-sid-map advertise-local
</pre> 
</div> 

#### TI-LFA FRR configuration

<div class="highlighter-rouge">
<pre class="highlight">
 fast-reroute per-prefix level-2 all
 fast-reroute ti-lfa level-2
 microloop avoidance protected
!

interface Loopback0
 ip address 100.0.1.51 255.255.255.255
 ip router isis ACCESS
 isis circuit-type level-2-only
end
</pre> 
</div> 

#### IS-IS and MPLS interface configuration

<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigabitEthernet0/0/12
 mtu 9216
 ip address 10.117.151.1 255.255.255.254
 ip router isis ACCESS
 mpls ip
 isis circuit-type level-2-only
 isis network point-to-point
 isis metric 100
end
</pre> 
</div> 

### MPLS Segment Routing Traffic Engineering (SRTE)

<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
 mpls traffic-eng router-id Loopback0
 mpls traffic-eng level-2

interface TenGigabitEthernet0/0/12
 mpls traffic-eng tunnels
</pre> 
</div> 

### Area Border Routers (ABRs) IGP-ISIS Redistribution configuration (IOS-XR)

The ABR nodes must provide IP reachability for RRs, SR-PCEs and NSO between 
ISIS-ACCESS and ISIS-CORE IGP domains. This is done by IP
prefix redistribution. The ABR nodes have static hold-down routes for the 
block of IP space used in each domain across the network, those static routes are then redistributed into the domains using the _redistribute static_ command with a route-policy. The distance command is used to ensure redistributed routes are not preferred over local IS-IS routes on the opposite ABR.  The distance command must be applied to both ABR nodes. 

<div class="highlighter-rouge">
<pre class="highlight">
router static
address-family ipv4 unicast
  100.0.0.0/24 Null0
  100.0.1.0/24 Null0
  100.1.0.0/24 Null0
  100.1.1.0/24 Null0
prefix-set ACCESS-PCE_SvRR-LOOPBACKS
  100.0.1.0/24,
  100.1.1.0/24
end-set
prefix-set RR-LOOPBACKS
    100.0.0.0/24,
  100.1.0.0/24
end-set
</pre> 
</div> 

#### Redistribute Core SvRR and TvRR loopback into Access domain

<div class="highlighter-rouge">
<pre class="highlight">
route-policy CORE-TO-ACCESS1
  if destination in RR-LOOPBACKS then
    pass
  else
    drop
  endif
end-policy
!
router isis ACCESS 
 address-family ipv4 unicast
  distance 254 0.0.0.0/0 RR-LOOPBACKS
  redistribute static route-policy CORE-TO-ACCESS1    
</pre> 
</div> 

#### Redistribute Access SR-PCE and SvRR loopbacks into CORE domain

<div class="highlighter-rouge">
<pre class="highlight">
route-policy ACCESS1-TO-CORE                                     
  if destination in ACCESS-PCE_SvRR-LOOPBACKS then               
    pass                                                         
  else                                                            
    drop                                                         
  endif                                                          
end-policy                                                       
! 
router isis CORE
 address-family ipv4 unicast
  distance 254 0.0.0.0/0 ACCESS-PCE_SvRR-LOOPBACKS
  redistribute static route-policy CORE-TO-ACCESS1
</pre> 
</div> 

## Multicast transport using mLDP 
### Overview 
This portion of the implementation guide instructs the user how to configure mLDP end to end across the multi-domain network. Multicast service examples are given in the "Services" section of the implementation guide. 

### mLDP core configuration 
In order to use mLDP across the Converged SDN Transport network LDP must first be enabled.  There are two mechanisms to enable LDP on physical interfaces across the network, LDP auto-configuration or manually under the MPLS LDP configuration context.  The capabilities statement will ensure LDP unicast FECs are not advertised, only mLDP FECs. Recursive forwarding is required in a multi-domain network. mLDP must be enabled on all participating A-PE, PE, AG, PA, and P routers.   

#### LDP base configuration with defined interfaces 
<div class="highlighter-rouge">
<pre class="highlight">
mpls ldp
 capabilities sac mldp-only
 mldp
  logging notifications
  address-family ipv4
   make-before-break delay 30
   forwarding recursive
   recursive-fec
  !
 !
 router-id 100.0.2.53
 session protection
 address-family ipv4
 !
 interface TenGigE0/0/0/6
 !
 interface TenGigE0/0/0/7
</pre> 
</div> 

#### LDP auto-configuration 
LDP can automatically be enabled on all IS-IS interfaces with the following configuration in the IS-IS configuration. It is recommended to do this only after configuring all MPLS LDP properties.   

<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
  address-family ipv4 unicast
    segment-routing mpls sr-prefer
    mpls ldp auto-config
</pre> 
</div>

## G.8275.1 and G.8275.2 PTP (1588v2) timing configuration 

### Summary 
This section contains the base configurations used for both G.8275.1 and G.8275.2 timing. Please see the CST 3.0 HLD for an overview on timing in general.  

![](http://xrdocs.io/design/images/cmfi/cmf-timing.png)

### Enable frequency synchronization
In order to lock the internal oscillator to a PTP source, frequency synchronization must first be enabled globally.  

<div class="highlighter-rouge">
<pre class="highlight">
frequency synchronization
 quality itu-t option 1
 clock-interface timing-mode system
 log selection changes
!
</pre> 
</div>

### Optional Synchronous Ethernet configuration (PTP hybrid mode) 
If the end-to-end devices support SyncE it should be enabled. SyncE will allow much faster frequency sync and maintain integrity for long periods of time during holdover events. Using SyncE for frequency and PTP for phase is known as "Hybrid" mode. A lower priority is used on the SyncE input (50 for SyncE vs. 100 for PTP). 

<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/10
 frequency synchronization
  selection input
  priority 50
 !
!
</pre> 
</div>

### PTP G.8275.2 global timing configuration 
As of CST 3.0, IOS-XR supports a single PTP timing profile and single clock type in the global PTP configuration. The clock domain should follow the ITU-T guidelines for specific profiles using a domain >44 for G.8275.2 clocks.   
<div class="highlighter-rouge">
<pre class="highlight">
ptp
 clock
  domain 60
  profile g.8275.2 clock-type T-BC 
  ! 
 frequency priority 100  
 time-of-day priority 50 
 log
  servo events
  best-master-clock changes
 !
</pre> 
</div>

### PTP G.8275.2 interface profile definitions 
It is recommended to use "profiles" defined globally which are then applied to interfaces participating in timing. This helps minimize per-interface timing configuration. It is also recommended to define different profiles for "master" and "slave" interfaces.   

#### IPv4 G.8275.2 master profile 
The master profile is assigned to interfaces for which the router is acting as a boundary clock
<div class="highlighter-rouge">
<pre class="highlight">
ptp
 profile g82752_master_v4
  transport ipv4
  port state master-only
  sync frequency 16
  clock operation one-step <-- Note the NCS series should be configured with one-step, ASR9000 with two-step 
  announce timeout 5 
  announce interval 1
  unicast-grant invalid-request deny
  delay-request frequency 16
 !
!
</pre> 
</div>

#### IPv6 G.8275.2 master profile 
The master profile is assigned to interfaces for which the router is acting as a boundary clock
<div class="highlighter-rouge">
<pre class="highlight">
ptp
 profile g82752_master_v6
  transport ipv6
  port state master-only
  sync frequency 16
  clock operation one-step
  announce timeout 10
  announce interval 1
  unicast-grant invalid-request deny
  delay-request frequency 16
 !
!
</pre> 
</div>

#### IPv4 G.8275.2 slave profile 
The slave profile is assigned to interfaces for which the router is acting as a slave to another master clock  
<div class="highlighter-rouge">
<pre class="highlight">
ptp
 profile g82752_master_v4
  transport ipv4
  port state slave-only 
  sync frequency 16
  clock operation one-step <-- Note the NCS series should be configured with one-step, ASR9000 with two-step
  announce timeout 10
  announce interval 1
  unicast-grant invalid-request deny
  delay-request frequency 16
 !
!
</pre> 
</div>

#### IPv6 G.8275.2 slave profile 
The slave profile is assigned to interfaces for which the router is acting as a slave to another master clock  
<div class="highlighter-rouge">
<pre class="highlight">
ptp
 profile g82752_master_v6
  transport ipv6
  port state slave-only 
  sync frequency 16
  clock operation one-step <-- Note the NCS series should be configured with one-step, ASR9000 with two-step
  announce timeout 10
  announce interval 1
  unicast-grant invalid-request deny
  delay-request frequency 16
 !
!
</pre> 
</div>

### PTP G.8275.1 global timing configuration 
As of CST 3.0, IOS-XR supports a single PTP timing profile and single clock type in the global PTP configuration. The clock domain should follow the ITU-T guidelines for specific profiles using a domain <44 for G.8275.1 clocks.   
<div class="highlighter-rouge">
<pre class="highlight">
ptp
clock domain 24
  operation one-step <b>Use one-step for NCS series, two-step for ASR 9000</b>  
  physical-layer-frequency
  frequency priority 100 
  profile g.8275.1 clock-type T-BC
log
  servo events
  best-master-clock changes
</pre> 
</div>

#### IPv6 G.8275.1 slave profile 
The slave profile is assigned to interfaces for which the router is acting as a slave to another master clock  
<div class="highlighter-rouge">
<pre class="highlight">
ptp
 profile g82751_slave
  port state slave-only 
  clock operation one-step <b><-- Note the NCS series should be configured with one-step, ASR9000 with two-step</b> 
  announce timeout 10
  announce interval 1
  delay-request frequency 16
  multicast transport ethernet
 !
!
</pre> 
</div>

#### IPv6 G.8275.1 master profile 
The master profile is assigned to interfaces for which the router is acting as a master to slave devices 
<div class="highlighter-rouge">
<pre class="highlight">
ptp
 profile g82751_slave
  port state master-only  
  clock operation one-step <b><-- Note the NCS series should be configured with one-step, ASR9000 with two-step</b>
  sync frequency 16 
  announce timeout 10
  announce interval 1
  delay-request frequency 16
  multicast transport ethernet
 !
!
</pre> 
</div>

### Application of PTP profile to physical interface 
**Note:** In CST 3.0 PTP may only be enabled on physical interfaces. G.8275.1 operates at L2 and supports PTP across Bundle member links and interfaces part of a bridge domain. G.8275.2 operates at L3 and does not support Bundle interfaces or BVI interfaces.   

#### G.8275.2 interface configuration 
This example is of a slave device using a master of 2405:10:23:253::0.  
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/6
 ptp
  profile g82752_slave_v6
  master ipv6 2405:10:23:253::
  !
 !
</pre> 
</div>

#### G.8275.1 interface configuration 
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/6
 ptp
  profile g82751_slave
  !
 !
</pre> 
</div>

## Segment Routing Path Computation Element (SR-PCE) configuration

<div class="highlighter-rouge">
<pre class="highlight">
router static
 address-family ipv4 unicast
  0.0.0.0/1 Null0

router bgp 100
 nsr
 bgp router-id 100.0.0.100
 bgp graceful-restart graceful-reset
 bgp graceful-restart
 ibgp policy out enforce-modifications
 address-family link-state link-state
 !
 neighbor-group TvRR
  remote-as 100
  update-source Loopback0
  address-family link-state link-state
  !
 !
 neighbor 100.0.0.10
  use neighbor-group TvRR
 !
 neighbor 100.1.0.10
  use neighbor-group TvRR
 !
!
pce
 address ipv4 100.100.100.1
 rest
  user rest_user
   password encrypted 00141215174C04140B
  !
  authentication basic
 !
 state-sync ipv4 100.100.100.2
 peer-filter ipv4 access-list pe-routers
!
</pre> 
</div> 

## BGP - Services (sRR) and Transport (tRR) route reflector configuration 

### Services Route Reflector (sRR) configuration 
In the CST validation a sRR is used to reflect all service routes. In a production network each service could be allocated its own sRR based on resiliency and scale demands.  

<div class="highlighter-rouge">
<pre class="highlight">
router static
 address-family ipv4 unicast
  0.0.0.0/1 Null0

router bgp 100
 nsr
 bgp router-id 100.0.0.200
 bgp graceful-restart
 ibgp policy out enforce-modifications
 address-family vpnv4 unicast
  nexthop trigger-delay critical 10
  additional-paths receive
  additional-paths send
 !
 address-family vpnv6 unicast
  nexthop trigger-delay critical 10
  additional-paths receive
  additional-paths send
  retain route-target all
 !
 address-family l2vpn evpn
  additional-paths receive
  additional-paths send
 !
 address-family ipv4 mvpn
  nexthop trigger-delay critical 10
  soft-reconfiguration inbound always
  !
 address-family ipv6 mvpn
  nexthop trigger-delay critical 10
  soft-reconfiguration inbound always
  !
 neighbor-group SvRR-Client
  remote-as 100
  bfd fast-detect 
  bfd minimum-interval 3 
  update-source Loopback0
  address-family l2vpn evpn
   route-reflector-client
   !
  address-family vpnv4 unicast 
   route-reflector-client
   !
  address-family vpnv6 unicast  
   route-reflector-client
   !
  address-family ipv4 mvpn
   route-reflector-client
   !
  address-family ipv6 mvpn
   route-reflector-client
  !
 !
 neighbor 100.0.0.1
  use neighbor-group SvRR-Client
 !
!
</pre> 
</div> 

### Transport Route Reflector (tRR) configuration

<div class="highlighter-rouge">
<pre class="highlight">
router static
 address-family ipv4 unicast
  0.0.0.0/1 Null0

router bgp 100
 nsr
 bgp router-id 100.0.0.10
 bgp graceful-restart
 ibgp policy out enforce-modifications
 address-family link-state link-state
  additional-paths receive
  additional-paths send
 !
 neighbor-group RRC
  remote-as 100
  update-source Loopback0
  address-family link-state link-state
   route-reflector-client
  !
 !
 neighbor 100.0.0.1
  use neighbor-group RRC
 !
 neighbor 100.0.0.2
  use neighbor-group RRC
!
</pre> 
</div> 


## BGP – Provider Edge Routers (A-PEx and PEx) to service RR 
Each PE router is configured with BGP sessions to service route-reflectors for advertising VPN service routes across the inter-domain network.  
    
### IOS-XR configuration 

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 nsr
 bgp router-id 100.0.1.50
 bgp graceful-restart graceful-reset
 bgp graceful-restart
 ibgp policy out enforce-modifications
 address-family vpnv4 unicast
 !
 address-family vpnv6 unicast
 !
 address-family ipv4 mvpn
 !
 address-family ipv6 mvpn
 !
 address-family l2vpn evpn
 !
 neighbor-group SvRR
  remote-as 100
  bfd fast-detect 
  bfd minimum-interval 3 
  update-source Loopback0
  address-family vpnv4 unicast
  soft-reconfiguration inbound always
  !
  address-family vpnv6 unicast
  soft-reconfiguration inbound always
  !
  address-family ipv4 mvpn
  soft-reconfiguration inbound always
  !
  address-family ipv6 mvpn
  soft-reconfiguration inbound always
  !
  address-family l2vpn evpn
  soft-reconfiguration inbound always
  !
 !
 neighbor 100.0.1.201
  use neighbor-group SvRR
  ! 
! 
</pre> 
</div> 

### IOS-XE configuration

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 bgp router-id 100.0.1.51
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor SvRR peer-group
 neighbor SvRR remote-as 100
 neighbor SvRR update-source Loopback0
 neighbor 100.0.1.201 peer-group SvRR
 !
 address-family ipv4
 exit-address-family
 !
 address-family vpnv4
  neighbor SvRR send-community both
  neighbor SvRR next-hop-self
  neighbor 100.0.1.201 activate
 exit-address-family
 !
 address-family l2vpn evpn
  neighbor SvRR send-community both
  neighbor SvRR next-hop-self
  neighbor 100.0.1.201 activate
 exit-address-family
 !
</pre> 
</div> 

## BGP-LU co-existence BGP configuration 
CST 3.0 introduced co-existence between services using BGP-LU and SR endpoints. If you are using SR and BGP-LU within the same domain it requires using BGP-SR in order to resolve prefixes correctly on the each ABR. BGP-SR uses a new BGP community attached to the BGP-LU prefix to convey the SR prefix-sid index end to end across the network. Using the same prefix-sid index both within the SR-MPLS IGP domain and across the BGP-LU network simplifies the network from an operational perspective since the path to an end node can always be identified by that SID.  

It is recommended to enable the BGP-SR configuration when enabling SR on the PE node. See the PE configuration below for an example of this configuration.   

#### Segment Routing Global Block Configuration 
The BGP process must know about the SRGB in order to properly allocate local BGP-SR labels when receiving a BGP-LU prefix with a BGP-SR index community.  This is done via the following configuration. If a SRGB is defined under the IGP it must match the global SRGB value. The IGP will inherit this SRGB value if none is previously defined.  

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 global-block 32000 64000
 !
! 
</pre> 
</div>


#### Boundary node configuration  
The following configuration is necessary on all domain boundary nodes. Note the _ibgp policy out enforce-modifications_ command is required to change the next-hop on reflected IBGP routes.  

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 ibgp policy out enforce-modifications
 neighbor-group BGP-LU-PE
  remote-as 100
  update-source Loopback0
  address-family ipv4 labeled-unicast
   soft-reconfiguration inbound always 
   route-reflector-client
   next-hop-self
  !
 !
 neighbor-group BGP-LU-PE
  remote-as 100
  update-source Loopback0
  address-family ipv4 labeled-unicast
   soft-reconfiguration inbound always 
   route-reflector-client
   next-hop-self
  !
 !
 neighbor 100.0.2.53
  use neighbor-group BGP-LU-PE
 !
 neighbor 100.0.2.52
  use neighbor-group BGP-LU-PE 
 !
 neighbor 100.0.0.1 
  use neighbor-group BGP-LU-BORDER
 !
 neighbor 100.0.0.2 
  use neighbor-group BGP-LU-BORDER
 ! 
! 
</pre> 
</div>


#### PE node configuration  
The following configuration is necessary on all domain PE nodes participating in BGP-LU/BGP-SR. The label-index set must match the index of the Loopback addresses being advertised into BGP. This example shows a single Loopback address being advertised into BGP.   

<div class="highlighter-rouge">
<pre class="highlight">
route-policy LOOPBACK-INTO-BGP-LU($SID-LOOPBACK0)
  set label-index $SID-LOOPBACK0
  set aigp-metric igp-cost
end-policy
!
router bgp 100 
  address-family ipv4 unicast
   network 100.0.2.53/32 route-policy LOOPBACK-INTO-BGP-LU(153)
 !
 neighbor-group BGP-LU-BORDER
  remote-as 100
  update-source Loopback0
  address-family ipv4 labeled-unicast
  !
 !
 neighbor 100.0.0.3 
  use neighbor-group BGP-LU-BORDER
 !
 neighbor 100.0.0.4 
  use neighbor-group BGP-LU-BORDER
 !
</pre> 
</div>

## Area Border Routers (ABRs) IGP topology distribution

Next network diagram: “BGP-LS Topology Distribution” shows how Area
Border Routers (ABRs) distribute IGP network topology from ISIS ACCESS
and ISIS CORE to Transport Route-Reflectors (tRRs). tRRs then reflect
topology to Segment Routing Path Computation Element (SR-PCEs). Each SR-PCE has full 
visibility of the entire inter-domain network. 

**Note:** Each IS-IS process in the network requires a unique instance-id to identify itself to the PCE.

![](http://xrdocs.io/design/images/cmfi/image5.png)

_Figure 5: BGP-LS Topology Distribution_

<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
 **distribute link-state instance-id 101**
 net 49.0001.0101.0000.0001.00
 address-family ipv4 unicast
  mpls traffic-eng router-id Loopback0
  !
! 
router isis CORE
 **distribute link-state instance-id 100**
 net 49.0001.0100.0000.0001.00
 address-family ipv4 unicast
  mpls traffic-eng router-id Loopback0
  !
!  
router bgp 100
 **address-family link-state link-state**
 !
 neighbor-group TvRR
  remote-as 100
  update-source Loopback0
  address-family link-state link-state
  !
  neighbor 100.0.0.10
  use neighbor-group TvRR
 !
 neighbor 100.1.0.10
  use neighbor-group TvRR
 !
</pre> 
</div> 


## Segment Routing Traffic Engineering (SRTE) and Services Integration

This section shows how to integrate Traffic Engineering (SRTE) with
services. ODN is configured by first defining a global ODN color associated with specific SR Policy constraints. The color and BGP next-hop address on the service route will be used to dynamically instantiate a SR Policy to the remote VPN endpoint.   

### On Demand Next-Hop (ODN) configuration – IOS-XR

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  logging
   policy status
  !
  on-demand color 100
   dynamic
    pce
    !
    metric
     type igp
    !
   !
  !
  pcc
   source-address ipv4 100.0.1.50
   pce address ipv4 100.0.1.101
   !
   pce address ipv4 100.1.1.101
   !
  !

extcommunity-set opaque BLUE
  100
end-set

route-policy ODN_EVPN
  set extcommunity color BLUE
end-policy

router bgp 100
  address-family l2vpn evpn
   route-policy ODN_EVPN out
  !
!
</pre> 
</div> 

### On Demand Next-Hop (ODN) configuration – IOS-XE

<div class="highlighter-rouge">
<pre class="highlight">
mpls traffic-eng tunnels
mpls traffic-eng pcc peer 100.0.1.101 source 100.0.1.51
mpls traffic-eng pcc peer 100.0.1.111 source 100.0.1.51
mpls traffic-eng pcc report-all
mpls traffic-eng auto-tunnel p2p config unnumbered-interface Loopback0
mpls traffic-eng auto-tunnel p2p tunnel-num min 1000 max 5000
!
mpls traffic-eng lsp attributes L3VPN-SRTE
 path-selection metric igp
 pce
!
ip community-list 1 permit 9999
!
route-map L3VPN-ODN-TE-INIT permit 10
 match community 1
 set attribute-set L3VPN-SRTE
!
route-map L3VPN-SR-ODN-Mark-Comm permit 10
 match ip address L3VPN-ODN-Prefixes
 set community 9999
 !
!
router bgp 100
 address-family vpnv4
  neighbor SvRR send-community both
  neighbor SvRR route-map L3VPN-ODN-TE-INIT in
  neighbor SvRR route-map L3VPN-SR-ODN-Mark-Comm out
</pre> 
</div> 

### SR-PCE configuration – IOS-XR

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  pcc
   source-address ipv4 100.0.1.50
   pce address ipv4 100.0.1.101
   !
   pce address ipv4 100.1.1.101
   !
  !
</pre> 
</div> 

### SR-PCE configuration – IOS-XE

<div class="highlighter-rouge">
<pre class="highlight">
mpls traffic-eng tunnels
mpls traffic-eng pcc peer 100.0.1.101 source 100.0.1.51
mpls traffic-eng pcc peer 100.0.1.111 source 100.0.1.51
mpls traffic-eng pcc report-all
</pre> 
</div> 

<br> 

## SR-TE Policy Configuration 
At the foundation of CST is the use of Segment Routing Traffic Engineering Policies.  SR-TE allow providers to create end to end traffic paths 
with engineered constraints to achieve a SLA objective.  SR-TE Policies are either dynamically created by ODN (see ODN section) or users can 
configure SR-TE Policies on the head-end node.   

### SR-TE Color and Endpoint 
The components uniquely identifying a SR-TE Policy to a destination PE node are its endpoint and color. 
* The endpoint is the destination node loopback address.  Note the endpoint address should not be an anycast address. 
* The color is a 32-bit value which should have a SLA meaning to the network. The color allows for multiple SR-TE Policies 
to exist between a pair of nodes, each one with its own set of metrics and constraints.  

### SR-TE Candidate Paths 
* Each SR-TE Policy configured on a node must have at least one candidate path defined. 
* If multiple candidate paths are defined, only one is active at any one time. 
* The candidate path with the higher preference value is preferred over candidate paths with a lower preference value. 
* The candidate path configuration specifies whether the path is dynamic or uses an explicit segment list. 
* Within the dynamic configuration one can specify whether to use a PCE or not, the metric type used in the path computation (IGP metric, latency, TE metric, hop count), and 
the additional constraints placed on the path (link affinities, flex-algo constraints, or a cumulative metric of type IGP metric, latency, TE Metric, or hop count)
* There is a default candidate path with a preference of 200 using head-end IGP path computation 
* Each candidate path can have multiple explicit segment lists defined with a bandwidth weight value to load balance traffic across multiple explicit paths 

### Service to SR-TE Policy Forwarding 
Service traffic is forwarded over SR-TE Policies in the CST design using per-destination automated steering. 
* Per-destination steering utilizes two BGP components of the service route to forward traffic to a matching SR Policy 
  * A color extended community attached to the service route matching the SR Policy color 
  * The BGP next-hop address of the service route to match the endpoint of the SR Policy   

### SR-TE Configuration Examples 

#### SR Policy using IGP computation, head-end computation  
The local PE device will compute a path using the lowest cumulative path to 100.0.1.50.  Note in the multi-domain CST design, this computation will fail to nodes not found within the same IS-IS domain as the PE.  

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  policy GREEN-PE3-24
   color 1024 end-point ipv4 100.0.1.50
   candidate-paths
    preference 1
     dynamic
      pcep
      !
      metric
       type igp
</pre>
</div>

#### SR Policy using lowest IGP metric computation and PCEP
This policy will request a path from the configured primary PCE with the lowest cumulative IGP metric to the endpoint 100.0.1.50 

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  policy GREEN-PE3-24
   color 1024 end-point ipv4 100.0.1.50
   candidate-paths
    preference 1
     dynamic
      pcep
      !
      metric
       type igp
</pre>
</div>

#### SR Policy using lowest latency metric and PCEP
This policy will request a path from the configured primary PCE with the lowest cumulative latency to the endpoint 100.0.1.50.  As covered in the performance-measurement section, 
the per-link latency metric value used will be the dynamic/static PM value, a configured TE metric value, or the IGP metric.   

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  policy GREEN-PE3-24
   color 1024 end-point ipv4 100.0.1.50
   candidate-paths
    preference 1
     dynamic
      pcep
      !
      metric
       type latency  
</pre>
</div>

#### SR Policy using explicit segment list  
This policy does not perform any path computation, it will utilize the statically defined segment lists as the forwarding path across the network. The node does however check the validity of the node segments in the list. 
Each node SID in the segment list can be defined by either IP address or SID.  The full path to the egress node must be defined in the list, but you do not need to define every node explicitly in the path. If you want the 
path to take a specific link the correct node and adjacency SID must be defined in the list.   

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  segment-list anycast-path 
   index 1 mpls label 17034
   index 2 mpls label 16150 
  !
  policy anycast-path-ape3 
   color 9999 end-point ipv4 100.0.1.50
   candidate-paths
    preference 1
     explicit segment-list anycast-path
</pre>
</div>

## QoS Implementation 

### Summary 
Please see the CST 3.0 HLD for in-depth information on design choices. 

### Core QoS configuration 
The core QoS policies defined for CST 3.0 utilize priority levels, with no bandwidth guarantees per traffic class. In a production network it is recommended to analyze traffic flows and determine an appropriate BW guarantee per traffic class. The core QoS uses four classes. Note the "video" class uses priority level 6 since only levels 6 and 7 are supported for high priority multicast.   

![](http://xrdocs.io/design/images/cmfi/cmf-qos-core.png)

 Traffic Type | Priority Level | Core EXP Marking  
| ----------|---------|----------|---------------|
| Network Control | 1 | 6 | 
| Voice |  2 | 5 | 
| High Priority |  3 | 4| 
| Video |  6 | 2 | 
| Default | 0 | 0 | 

#### Class maps used in QoS policies 
Class maps are used within a policy map to match packet criteria or internal QoS markings like traffic-class or qos-group 

<div class="highlighter-rouge">
<pre class="highlight">
class-map match-any match-ef-exp5
 description High priority, EF
 match dscp 46
 match mpls experimental topmost 5
 end-class-map
!
class-map match-any match-cs5-exp4
 description Second highest priority
 match dscp 40
 match mpls experimental topmost 4
 end-class-map
!
class-map match-any match-video-cs4-exp2
 description Video
 match dscp 32
 match mpls experimental topmost 2
 end-class-map
!
class-map match-any match-cs6-exp6
 description Highest priority control-plane traffic
 match dscp cs6
 match mpls experimental topmost 6
 end-class-map
!
class-map match-any match-qos-group-1
 match qos-group 1
 end-class-map
!
class-map match-any match-qos-group-2
 match qos-group 2
 end-class-map
!
class-map match-any match-qos-group-3
 match qos-group 3
 end-class-map
!
class-map match-any match-qos-group-6
 match qos-group 3
 end-class-map
!
class-map match-any match-traffic-class-1
 description "Match highest priority traffic-class 1"
 match traffic-class 1
 end-class-map
!
class-map match-any match-traffic-class-2
 description "Match high priority traffic-class 2"
 match traffic-class 2
 end-class-map
!
class-map match-any match-traffic-class-3
 description "Match medium traffic-class 3"
 match traffic-class 3
 end-class-map
!
class-map match-any match-traffic-class-6
 description "Match video traffic-class 6"
 match traffic-class 6
 end-class-map
</pre> 
</div>

#### Core ingress classifier policy 

<div class="highlighter-rouge">
<pre class="highlight">
policy-map core-ingress-classifier
 class match-cs6-exp6
  set traffic-class 1
 !
 class match-ef-exp5
  set traffic-class 2
 !
 class match-cs5-exp4
  set traffic-class 3
 !
 class match-video-cs4-exp2
  set traffic-class 6
 !
 class class-default
  set mpls experimental topmost 0
  set traffic-class 0
  set dscp 0
 !
 end-policy-map
!
</pre> 
</div>

#### Core egress queueing map 

<div class="highlighter-rouge">
<pre class="highlight">
policy-map core-egress-queuing
 class match-traffic-class-2
  priority level 2
  queue-limit 100 us
 !
 class match-traffic-class-3
  priority level 3
  queue-limit 500 us
 !
 class match-traffic-class-6
  priority level 6
  queue-limit 500 us
 !
 class match-traffic-class-1
  priority level 1
  queue-limit 500 us
 !
 class class-default
  queue-limit 250 ms
 !
 end-policy-map
!
</pre> 
</div>

#### Core egress MPLS EXP marking map 
The following policy must be applied for PE devices with MPLS-based VPN services in order for service traffic classified in a specific QoS Group to be marked. VLAN-based P2P L2VPN services will by default inspect the incoming 802.1p bits and copy those the egress MPLS EXP if no specific ingress policy overrides that behavior. Note the EXP can be set in either an ingress or egress QoS policy. This QoS example sets the EXP via the egress map.   

<div class="highlighter-rouge">
<pre class="highlight">
policy-map core-egress-exp-marking
 class match-qos-group-1
  set mpls experimental imposition 6
 !
 class match-qos-group-2
  set mpls experimental imposition 5
 !
 class match-qos-group-3
  set mpls experimental imposition 4
 !
 class match-qos-group-6
  set mpls experimental imposition 2
 !
 class class-default
  set mpls experimental imposition 0
 !
 end-policy-map
!
</pre> 
</div>


### H-QoS configuration  

#### Enabling H-QoS on NCS 540 and NCS 5500 
Enabling H-QoS on the NCS platforms requires the following global command and requires a reload of the device. 

<div class="highlighter-rouge">
<pre class="highlight">
hw-module profile qos hqos-enable
</pre> 
</div>

#### Example H-QoS policy for 5G services 
The following H-QoS policy represents an example QoS policy reserving 5Gbps on a sub-interface. On ingress each child class is policed to a certain percentage of the 5Gbps policer.  In the egress queuing policy, shaping is used with guaranteed each class a certain amount of egress bandwidth, with high priority traffic being serviced in a low-latency queue (LLQ).  

#### Class maps used in ingress H-QoS policies 

<div class="highlighter-rouge">
<pre class="highlight">
class-map match-any edge-hqos-2-in
 match dscp 46
 end-class-map
!
class-map match-any edge-hqos-3-in
 match dscp 40
 end-class-map
!
class-map match-any edge-hqos-6-in
 match dscp 32
 end-class-map
</pre> 
</div>

#### Parent ingress QoS policy 

<div class="highlighter-rouge">
<pre class="highlight">
policy-map hqos-ingress-parent-5g
 class class-default
  service-policy hqos-ingress-child-policer
  police rate 5 gbps
  !
 !
 end-policy-map
</pre> 
</div>

#### H-QoS ingress child policies  

<div class="highlighter-rouge">
<pre class="highlight">
policy-map hqos-ingress-child-policer
 class edge-hqos-2-in
  set traffic-class 2
  police rate percent 10
  !
 !
 class edge-hqos-3-in
  set traffic-class 3
  police rate percent 30
  !
 !
 class edge-hqos-6-in
  set traffic-class 6
  police rate percent 30
  !
 !
 class class-default
  set traffic-class 0
  set dscp 0
  police rate percent 100 
  !
 !
 end-policy-map
</pre> 
</div>

#### Egress H-QoS parent policy (Priority levels) 

<div class="highlighter-rouge">
<pre class="highlight">
policy-map hqos-egress-parent-4g-priority
 class class-default
  service-policy hqos-egress-child-priority
  shape average 4 gbps
 !
 end-policy-map
!
</pre> 
</div>

#### Egress H-QoS child using priority only 
In this policy all classes can access 100% of the bandwidth, queues are services based on priority level. The lower priority level has preference.   

<div class="highlighter-rouge">
<pre class="highlight">
policy-map hqos-egress-child-priority
 class match-traffic-class-2
  shape average percent 100
  priority level 2
 !
 class match-traffic-class-3
  shape average percent 100
  priority level 3
 !
 class match-traffic-class-6
  priority level 4
  shape average percent 100
 !
 class class-default
 !
 end-policy-map
</pre> 
</div>

#### Egress H-QoS child using reserved bandwidth
In this policy each class is reserved a certain percentage of bandwidth. Each class may utilize up to 100% of the bandwidth, if traffic exceeds the guaranteed bandwidth it is eligible for drop.   

<div class="highlighter-rouge">
<pre class="highlight">
policy-map hqos-egress-child-bw
 class match-traffic-class-2
  bandwidth remaining percent 30
 !
 class match-traffic-class-3
  bandwidth remaining percent 30
 !
 class match-traffic-class-6
  bandwidth remaining percent 30
 !
 class class-default
  bandwidth remaining percent 10
 !
 end-policy-map
</pre> 
</div>

#### Egress H-QoS child using shaping  
In this policy each class is shaped to a defined amount and cannot exceed the defined bandwidth. 

<div class="highlighter-rouge">
<pre class="highlight">
policy-map hqos-egress-child-shaping
 class match-traffic-class-2
  shape average percent 30
 !
 class match-traffic-class-3
  shape average percent 30
 !
 class match-traffic-class-6
  shape average percent 30
 !
 class class-default
  shape average percent 10
 !
 end-policy-map
!
</pre> 
</div>

# Services
    
## End-To-End VPN Services

![](http://xrdocs.io/design/images/cmfi/image6.png)

_Figure 6: End-To-End Services Table_

### L3VPN MP-BGP VPNv4 On-Demand Next-Hop

![](http://xrdocs.io/design/images/cmfi/image7.png)

_Figure 7: L3VPN MP-BGP VPNv4 On-Demand Next-Hop Control Plane_

**Access Routers:** **Cisco ASR920 IOS-XE and NCS540 IOS-XR**

1.  **Operator:** New VPNv4 instance via CLI or NSO

2.  **Access Router:** Advertises/receives VPNv4 routes to/from Services
    Route-Reflector (sRR)

3.  **Access Router**: Request SR-PCE to provide path (shortest IGP metric)
    to remote access router

4.  **SR-PCE:** Computes and provides the path to remote router(s)

5.  **Access Router:** Programs Segment Routing Traffic Engineering
    (SRTE) Policy to reach remote access router

Please refer to “**On Demand Next-Hop (ODN)**” sections for initial ODN configuration.

#### Access Router Service Provisioning (IOS-XR)

**ODN route-policy configuration** 
<div class="highlighter-rouge">
<pre class="highlight">
extcommunity-set opaque ODN-GREEN
  100
end-set

route-policy ODN-L3VPN-OUT 
  set extcommunity color ODN-GREEN 
  pass
end-policy
</pre> 
</div> 

**VRF definition configuration**
<div class="highlighter-rouge">
<pre class="highlight">
vrf ODN-L3VPN
 rd 100:1
 address-family ipv4 unicast
  import route-target
   100:1
  !
  export route-target
  export route-policy ODN-L3VPN-OUT
   100:1
  !
 !
 address-family ipv6 unicast
  import route-target
   100:1
  !
  export route-target
  export route-policy ODN-L3VPN-OUT
   100:1
  !
 !
</pre> 
</div> 


**VRF Interface configuration**
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/23.2000
 mtu 9216 
 vrf ODN-L3VPN  
 ipv4 address 172.106.1.1 255.255.255.0
 encapsulation dot1q 2000
</pre> 
</div> 

**BGP VRF configuration with static/connected only**  

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 vrf VRF-MLDP
  rd auto
  address-family ipv4 unicast
   redistribute connected
   redistribute static
  !
  address-family ipv6 unicast 
   redistribute connected 
   redistribute static 
  !
</pre> 
</div> 

#### Access Router Service Provisioning (IOS-XE)

**VRF definition configuration**

<div class="highlighter-rouge">
<pre class="highlight">
vrf definition L3VPN-SRODN-1
 rd 100:100
 route-target export 100:100
 route-target import 100:100
 address-family ipv4
 exit-address-family
</pre> 
</div> 


**VRF Interface configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface GigabitEthernet0/0/2
 mtu 9216
 vrf forwarding L3VPN-SRODN-1
 ip address 10.5.1.1 255.255.255.0
 negotiation auto
end
</pre> 
</div> 

**BGP VRF configuration Static & BGP neighbor **

**Static routing configuration**

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 address-family ipv4 vrf L3VPN-SRODN-1
  redistribute connected
 exit-address-family
</pre> 
</div> 

**BGP neighbor configuration**

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 neighbor Customer-1 peer-group
 neighbor Customer-1 remote-as 200
 neighbor 10.10.10.1 peer-group Customer-1
 address-family ipv4 vrf L3VPN-SRODN-2
   neighbor 10.10.10.1 activate
 exit-address-family
</pre> 
</div> 

### L2VPN Single-Homed EVPN-VPWS On-Demand Next-Hop

![](http://xrdocs.io/design/images/cmfi/image8.png)

_Figure 8: L2VPN Single-Homed EVPN-VPWS On-Demand Next-Hop Control Plane_

**Access Routers:** **Cisco NCS5501-SE IOS-XR**

1.  **Operator:** New EVPN-VPWS instance via CLI or NSO

2.  **Access Router:** Advertises/receives EVPN-VPWS instance to/from
    Services Route-Reflector (sRR)

3.  **Access Router**: Request SR-PCE to provide path (shortest IGP metric)
    to remote access router

4.  **SR-PCE:** Computes and provides the path to remote router(s)

5.  **Access Router:** Programs Segment Routing Traffic Engineering
    (SRTE) Policy to reach remote access router

**Note:** Please refer to **On Demand Next-Hop (ODN) – IOS-XR** section for initial ODN configuration. The correct EVPN L2VPN routes must be advertised with a specific color ext-community to trigger dynamic SR Policy instantiation.   

#### Access Router Service Provisioning (IOS-XR):

**Port based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn                                                                                                                           xconnect group evpn_vpws                                                                                                        
 p2p odn-1                                                                                                                      
  interface TenGigE0/0/0/5                                                                                                      
   neighbor evpn evi 1000 target 1 source 1  

interface TenGigE0/0/0/5 
  l2transport
</pre> 
</div> 

**VLAN Based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group evpn_vpws
 p2p odn-1
  neighbor evpn evi 1000 target 1 source 1
  !
! 
interface TenGigE0/0/0/5.1 l2transport
 encapsulation dot1q 1
 rewrite ingress tag pop 1 symmetric
!
</pre> 
</div> 

### L2VPN Static Pseudowire (PW) – Preferred Path (PCEP)

![](http://xrdocs.io/design/images/cmfi/image9.png)

_Figure 9: L2VPN Static Pseudowire (PW) – Preferred Path (PCEP) Control
Plane_

**Access Routers:** **Cisco NCS5501-SE IOS-XR or Cisco ASR920 IOS-XE**

1.  **Operator:** New Static Pseudowire (PW) instance via CLI or NSO

2.  **Access Router**: Request SR-PCE to provide path (shortest IGP metric)
    to remote access router

3.  **SR-PCE:** Computes and provides the path to remote router(s)

4.  **Access Router:** Programs Segment Routing Traffic Engineering
    (SRTE) Policy to reach remote access router
    
#### Access Router Service Provisioning (IOS-XR):

**Note:** EVPN VPWS dual homing is not supported when using an SR-TE preferred path.  

**Note:** In IOS-XR 6.6.3 the SR Policy used as the preferred path must be referenced by its generated name and not the configured policy name.  This requires first issuing the command   

**Define SR Policy** 

<div class="highlighter-rouge">
<pre class="highlight">
 traffic-eng
  policy GREEN-PE3-1
   color 1001 end-point ipv4 100.0.1.50
   candidate-paths
    preference 1
     dynamic
      pcep
      !
      metric
       type igp
</pre> 
</div> 

**Determine auto-configured policy name** 
The auto-configured policy name will be persistant and must be used as a reference in the L2VPN preferred-path configuration. 

<div class="highlighter-rouge">
<pre class="highlight">
RP/0/RP0/CPU0:A-PE8#show segment-routing traffic-eng policy candidate-path name GREEN-PE3-1 
&nbsp; 
SR-TE policy database  
Color: 1001, End-point: 100.0.1.50
  Name: <b>srte_c_1001_ep_100.0.1.50</b> 
</pre> 
</div> 

**Port Based Service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/15 
  l2transport
  ! 
! 
l2vpn 
 pw-class static-pw-class-PE3
  encapsulation mpls
   control-word
   preferred-path sr-te policy srte_c_1001_ep_100.0.1.50
   ! 
  !
 ! 
 p2p Static-PW-to-PE3-1
  interface TenGigE0/0/0/15
   neighbor ipv4 100.0.0.3 pw-id 1000                      
    mpls static label local 1000 remote 1000 pw-class static-pw-class-PE3   
</pre> 
</div> 


**VLAN Based Service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/5.1001 l2transport
 encapsulation dot1q 1001
 rewrite ingress tag pop 1 symmetric
 ! 
!  
l2vpn 
 pw-class static-pw-class-PE3
  encapsulation mpls
   control-word
   preferred-path sr-te policy srte_c_1001_ep_100.0.1.50 
  p2p Static-PW-to-PE7-2                                                                                                                                      
   interface TenGigE0/0/0/5.1001
    neighbor ipv4 100.0.0.3 pw-id 1001                      
     mpls static label local 1001 remote 1001 pw-class static-pw-class-PE3 
</pre> 
</div> 

#### Access Router Service Provisioning (IOS-XE):

**Port Based service with Static OAM configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface GigabitEthernet0/0/1
 mtu 9216
 no ip address
 negotiation auto
 no keepalive
 service instance 10 ethernet
  encapsulation default
  xconnect 100.0.2.54 100 encapsulation mpls manual pw-class mpls
   mpls label 100 100
   no mpls control-word
 !
 pseudowire-static-oam class static-oam                        
  timeout refresh send 10                                      
  ttl 255                     
  ! 
 ! 
!  
pseudowire-class mpls                                                     
 encapsulation mpls                                                       
 no control-word                                                          
 protocol none                                                            
 preferred-path interface Tunnel1                                         
 status protocol notification static static-oam                           
!           
</pre> 
</div> 

**VLAN Based Service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface GigabitEthernet0/0/1
 no ip address
 negotiation auto
 service instance 1 ethernet Static-VPWS-EVC
  encapsulation dot1q 10
  rewrite ingress tag pop 1 symmetric
  xconnect 100.0.2.54 100 encapsulation mpls manual pw-class mpls
   mpls label 100 100
   no mpls control-word
   !
  ! 
! 
pseudowire-class mpls                                                     
 encapsulation mpls                                                       
 no control-word                                                          
 protocol none                                                            
 preferred-path interface Tunnel1  
</pre> 
</div>

## Ethernet CFM for L2VPN service assurance 
Ethernet Connectivity Fault Management is an Ethernet OAM component used to validate end-to-end connectivity between service endpoints. Ethernet CFM is defined by two standards, 802.1ag and Y.1731. Within an SP network, Maintenance Domains are created based on service scope. Domains are typically separated by operator boundaries and may be nested but cannot overlap.  Within each service, maintenance points can be created to verify bi-directional end to end connectivity. These are known as MEPs (Maintenance End-Point) and MIPs (Maintenance Intermediate Points). These maintenance points process CFM messages. A MEP is configured at service endpoints and has directionality where an "up" MEP faces the core of the network and a "down" MEP faces a CE device or NNI port. MIPs are optional and are created dynamically. Detailed information on Ethernet CFM configuration and operation can be found at https://www.cisco.com/c/en/us/td/docs/routers/ncs5500/software/interfaces/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-66x/b-interfaces-hardware-component-cg-ncs5500-66x_chapter_0101.html 

### Maintenance Domain configuration 
A Maintenance Domain is defined by a unique name and associated level. The level can be 0-7. The numerical identifier usually corresponds to the scope of the MD, where 7 is associated with CE endpoints, 6 associated with PE devices connected to a CE. Additional levels may be required based on the topology and service boundaries which occur along the end-to-end service.  In this example we only a single domain and utilize level 0 for all MEPs.   
<div class="highlighter-rouge">
<pre class="highlight">
ethernet cfm
 domain EVPN-VPWS-PE3-PE8 level 0
</pre> 
</div>

### MEP configuration for EVPN-VPWS services
For L2VPN xconnect services, each service must have a MEP created on the end PE device. There are two components to defining a MEP, first defining the Ethernet CFM "service" and then defining the MEP on the physical or logical interface participating in the L2VPN xconnect service. In the following configuration the xconnect group "EVPN-VPWS-ODN-PE3" and P2P EVPN VPWS service odn-8 are already defined. The Ethernet CFM service of "odn-8" does NOT have to match the xconnect service name. The MEP crosscheck defines a remote MEP to listen for Continuity Check messages from.  It does not have to be the same as the local MEP defined on the physical sub-interface (103), but for P2P services it is best practice to make them identical.  This configuration will send Ethernet CFM Continuity Check (CC) messages every 1 minute to verify end to end reachability.   

**L2VPN configuration**
<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group EVPN-VPWS-ODN-PE3
  p2p odn-8
   interface TenGigE0/0/0/23.8
   neighbor evpn evi 1318 target 8 source 8
   !
  !
 !
!
</pre> 
</div>

**Physical sub-interface configuration** 
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/23.8 l2transport
 encapsulation dot1q 8
 rewrite ingress tag pop 1 symmetric
 ethernet cfm
  mep domain EVPN-VPWS-PE3-PE8 service odn-8 mep-id 103
  !
 !
!
</pre> 
</div>

**Ethernet CFM service configuration** 
<div class="highlighter-rouge">
<pre class="highlight">
ethernet cfm
 domain EVPN-VPWS-PE3-PE8
  service odn-8 xconnect group EVPN-VPWS-ODN-PE3 p2p odn-8
   mip auto-create all
   continuity-check interval 1m
   mep crosscheck
    mep-id 103
   !
   log crosscheck errors
   log continuity-check errors
   log continuity-check mep changes
  !
 !
!
</pre> 
</div>

## Multicast NG-MVPN Profile 14 using mLDP and ODN L3VPN
In ths service example we will implement multicast delivery across the CST network using mLDP transport for multicast and SR-MPLS for unicast traffic. L3VPN SR paths will be dynamically created using ODN. Multicast profile 14 is the "Partitioned MDT - MLDP P2MP - BGP-AD - BGP C-Mcast Signaling"  Using this profile each mVPN will use a dedicated P2MP tree, endpoints will be auto-discovered using NG-MVPN BGP NLRI, and customer multicast state such as source streams, PIM, and IGMP membership data will be signaled using BGP. Profile 14 is the recommended profile for high scale and utilizing label-switched multicast (LSM) across the core.      

**Please note that mLDP requires an IGP path to the source PE loopback address.  The CST design utilizes a multi-domain approach which normally does not advertise IGP routes across domain boundaries. If mLDP is being utilized across domains, controlled redistribution should be used to advertise the source PE loopback addresses to receiver PEs**  

### Multicast core configuration 
The multicast "core" includes transit endpoints participating in mLDP only. See the mLDP core configuration section for details on end-to-end mLDP configuration.  

### Unicast L3VPN PE configuration 
In order to complete an RPF check for SSM sources, unicast L3VPN configuration is required. Additionally the VRF must be defined under the BGP configuration with the NG-MVPN address families configured. In our use case we are utilizing ODN for creating the paths between L3VPN endpoints with a route-policy attached to the mVPN VRF to set a specific color on advertised routes.   

**ODN opaque ext-community set** 
<div class="highlighter-rouge">
<pre class="highlight">
extcommunity-set opaque MLDP
  1000
end-set
</pre> 
</div>

**ODN route-policy** 
<div class="highlighter-rouge">
<pre class="highlight">
route-policy ODN-MVPN
  set extcommunity color MLDP
  pass
end-policy
</pre> 
</div>


**Global L3VPN VRF definition** 
<div class="highlighter-rouge">
<pre class="highlight">
vrf VRF-MLDP
 address-family ipv4 unicast
  import route-target
   100:38
  !
  export route-policy ODN-MVPN
  export route-target
   100:38
  !
 !
 address-family ipv6 unicast
  import route-target
   100:38
  !
  export route-policy ODN-MVPN
  export route-target
   100:38
  !
 !
!
</pre> 
</div>

**BGP configuration** 
<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 vrf VRF-MLDP
  rd auto
  address-family ipv4 unicast
   redistribute connected
   redistribute static
  !
  address-family ipv6 unicast
   redistribute connected
   redistribute static
  ! 
  address-family ipv4 mvpn
  !
  address-family ipv6 mvpn
  !
 !
!
</pre> 
</div>

### Multicast PE configuration
The multicast "edge" includes all endpoints connected to native multicast sources or receivers. 

**Define RPF policy** 

<div class="highlighter-rouge">
<pre class="highlight">
route-policy mldp-partitioned-p2mp
  set core-tree mldp-partitioned-p2mp
end-policy
!
</pre> 
</div>

**Enable Multicast and define mVPN VRF** 

<div class="highlighter-rouge">
<pre class="highlight">
multicast-routing
 address-family ipv4
  interface Loopback0
   enable
  !
 !
 vrf VRF-MLDP
  address-family ipv4
   mdt source Loopback0
   rate-per-route
   interface all enable
   accounting per-prefix
   bgp auto-discovery mldp
   !
   mdt partitioned mldp ipv4 p2mp
   mdt data 100
  !
 !
!
</pre> 
</div>

**Enable PIM for mVPN VRF** 
In this instance there is an interface TenGigE0/0/0/23.2000 which is using PIM within the VRF 

<div class="highlighter-rouge">
<pre class="highlight">
router pim
 address-family ipv4
  rp-address 100.0.1.50
 !
 vrf VRF-MLDP
  address-family ipv4
   rpf topology route-policy mldp-partitioned-p2mp
   mdt c-multicast-routing bgp
   !
   interface TenGigE0/0/0/23.2000
    enable
   !
  !
</pre> 
</div>

**Enable IGMP for mVPN VRF interface** 
To discover listeners for a specific group, enable IGMP on interfaces within the VRF. These interested receivers will be advertised via BGP to establish end to end P2MP trees from the source.   

<div class="highlighter-rouge">
<pre class="highlight">
router igmp
 vrf VRF-MLDP
  interface TenGigE0/0/0/23.2001
  !
  version 3
 !
!
</pre> 
</div>

## Multicast distribution using TreeSID with static S,G Mapping 
TreeSID utilizes only Segment Routing to create and forward multicast traffic across an optimized tree. The TreeSID tree is configured on the SR-PCE for deployment to the 
network. PCEP is used to instantiate the correct computed segments end to end.  On the head-end source node, 

**Note TreeSID requires all nodes in the multicast distribution network to have connections to the same SR-PCE instances, please see the PCEP configuration section of the Implmentation Guide**  

### TreeSID SR-PCE Configuration 

### TreeSID Source Node Configuration 

### TreeSID Receiver Node Configuration 



### End-To-End VPN Services Data Plane

![](http://xrdocs.io/design/images/cmfi/image10.png)

_Figure 10: End-To-End Services Data Plane_

## Hierarchical Services

![](http://xrdocs.io/design/images/cmfi/image11.png)

_Figure 11: Hierarchical Services Table_

### L3VPN – Single-Homed EVPN-VPWS, MP-BGP VPNv4/6 with Pseudowire-Headend (PWHE)

![](http://xrdocs.io/design/images/cmfi/image12.png)

_Figure 12: L3VPN – Single-Homed EVPN-VPWS, MP-BGP VPNv4/6 with Pseudowire-Headend (PWHE) Control Plane_

**Access Routers:** **Cisco NCS5501-SE IOS-XR or Cisco ASR920 IOS-XE**

1.  **Operator:** New EVPN-VPWS instance via CLI or NSO

2.  **Access Router:** Path to PE Router is known via ACCESS-ISIS IGP.

**Provider Edge Routers:** **Cisco ASR9000 IOS-XR**

1.  **Operator:** New EVPN-VPWS instance via CLI or NSO

2.  **Provider Edge Router:** Path to Access Router is known via
    ACCESS-ISIS IGP.

3.  **Operator:** New L3VPN instance (VPNv4/6) together with
    Pseudowire-Headend (PWHE) via CLI or NSO

4.  **Provider Edge Router:** Path to remote PE is known via CORE-ISIS
    IGP.
    
#### Access Router Service Provisioning (IOS-XR):

**VLAN based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group evpn-vpws-l3vpn-PE1
  p2p L3VPN-VRF1
   interface TenGigE0/0/0/5.501
   neighbor evpn evi 13 target 501 source 501
   !
  !
 !
interface TenGigE0/0/0/5.501 l2transport
 encapsulation dot1q 501
 rewrite ingress tag pop 1 symmetric
</pre> 
</div> 

**Port based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn                                                                                                                                                            
 xconnect group evpn-vpws-l3vpn-PE1                                                                                           
 p2p odn-1                                                                                                                                                      
  interface TenGigE0/0/0/5                                                                                                                                
   neighbor evpn evi 13 target 502 source 502  
   !
  ! 
 !
! 
interface TenGigE0/0/0/5 
  l2transport
</pre> 
</div> 

#### Access Router Service Provisioning (IOS-XE):

**VLAN based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn evpn instance 14 point-to-point
 vpws context evpn-pe4-pe1
  service target 501 source 501
  member GigabitEthernet0/0/1 service-instance 501
 !
interface GigabitEthernet0/0/1
 service instance 501 ethernet
  encapsulation dot1q 501
  rewrite ingress tag pop 1 symmetric
 !
 </pre> 
</div> 

**Port based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn evpn instance 14 point-to-point
 vpws context evpn-pe4-pe1
  service target 501 source 501
  member GigabitEthernet0/0/1 service-instance 501
 !
interface GigabitEthernet0/0/1
 service instance 501 ethernet
  encapsulation default
</pre> 
</div> 

#### Provider Edge Router Service Provisioning (IOS-XR):

**VRF configuration**  

<div class="highlighter-rouge">
<pre class="highlight">
vrf L3VPN-ODNTE-VRF1                                                                                                          
 address-family ipv4 unicast                                                                                                  
  import route-target                                                                                                                  
   100:501                                                                                                                   
  !                                                                                                                                    
  export route-target                                                                                                                  
   100:501                                                                                                                             
  !                                                                                                                                    
 !                                                                                                                                     
 address-family ipv6 unicast                                                                                                           
  import route-target                                                                                                                  
   100:501                                                                                                                             
  !                                                                                                                                    
  export route-target                                                                                                                  
   100:501                                                                                                                             
  !
 !
</pre> 
</div> 

**BGP configuration**

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100                                                                                                                         
 vrf L3VPN-ODNTE-VRF1
  rd 100:501
  address-family ipv4 unicast
   redistribute connected
  !
  address-family ipv6 unicast
   redistribute connected
  !
 !
</pre> 
</div> 

**PWHE configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface PW-Ether1
 vrf L3VPN-ODNTE-VRF1
 ipv4 address 10.13.1.1 255.255.255.0
 ipv6 address 1000:10:13::1/126
 attach generic-interface-list PWHE
!
</pre> 
</div> 

**EVPN VPWS configuration towards Access PE**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group evpn-vpws-l3vpn-A-PE3
  p2p L3VPN-ODNTE-VRF1
   interface PW-Ether1
   neighbor evpn evi 13 target 501 source 501
   !
</pre> 
</div> 

![](http://xrdocs.io/design/images/cmfi/image13.png)

_Figure 13: L3VPN – Single-Homed EVPN-VPWS, MP-BGP VPNv4/6 with
Pseudowire-Headend (PWHE) Data Plane_

### L3VPN – Anycast Static Pseudowire (PW), MP-BGP VPNv4 with Anycast IRB

![](http://xrdocs.io/design/images/cmfi/image14.png)

_Figure 14: L3VPN – Anycast Static Pseudowire (PW), MP-BGP VPNv4 with
Anycast IRB Control Plane_

**Access Routers:** **Cisco NCS5501-SE IOS-XR or Cisco ASR920 IOS-XE**

3.  **Operator:** New Static Pseudowire (PW) instance via CLI or NSO

4.  **Access Router:** Path to PE Router is known via ACCESS-ISIS IGP.

**Provider Edge Routers:** **Cisco ASR9000 IOS-XR (Same on both PE
routers in same location PE1/2 and PE3/4)**

5.  **Operator:** New Static Pseudowire (PW) instance via CLI or NSO

6.  **Provider Edge Routers:** Path to Access Router is known via
    ACCESS-ISIS IGP.

7.  **Operator:** New L3VPN instance (VPNv4/6) together with Anycast IRB
    via CLI or NSO

8.  **Provider Edge Routers:** Path to remote PEs is known via CORE-ISIS
    IGP.
    
#### Access Router Service Provisioning (IOS-XR):

**VLAN based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group Static-VPWS-PE12-H-L3VPN-AnyCast
  p2p L3VPN-VRF1
   interface TenGigE0/0/0/2.1
   neighbor ipv4 100.100.100.12 pw-id 5001
    mpls static label local 5001 remote 5001
    pw-class static-pw-h-l3vpn-class
   !
  !
interface TenGigE0/0/0/2.1 l2transport
 encapsulation dot1q 1
 rewrite ingress tag pop 1 symmetric
 !
! 
l2vpn
 pw-class static-pw-h-l3vpn-class
  encapsulation mpls
   control-word
  !
</pre> 
</div> 

**Port based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group Static-VPWS-PE12-H-L3VPN-AnyCast
  p2p L3VPN-VRF1
   interface TenGigE0/0/0/2
   neighbor ipv4 100.100.100.12 pw-id 5001
    mpls static label local 5001 remote 5001
    pw-class static-pw-h-l3vpn-class
   !
  !
interface TenGigE0/0/0/2 
 l2transport
 !
! 
l2vpn
 pw-class static-pw-h-l3vpn-class
  encapsulation mpls
   control-word
  !
</pre> 
</div> 

#### Access Router Service Provisioning (IOS-XE):

**VLAN based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface GigabitEthernet0/0/5
 no ip address
 media-type auto-select
 negotiation auto
 service instance 1 ethernet
  encapsulation dot1q 1
  rewrite ingress tag pop 1 symmetric
  xconnect 100.100.100.12 4001 encapsulation mpls manual
   mpls label 4001 4001
   mpls control-word
 !
</pre> 
</div> 

**Port based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface GigabitEthernet0/0/5
 no ip address
 media-type auto-select
 negotiation auto
 service instance 1 ethernet
  encapsulation default
  xconnect 100.100.100.12 4001 encapsulation mpls manual
   mpls label 4001 4001
   mpls control-word
 !
</pre> 
</div> 

#### Provider Edge Routers Service Provisioning (IOS-XR):

<div class="highlighter-rouge">
<pre class="highlight">
cef adjacency route override rib
</pre> 
</div> 

**AnyCast Loopback configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface Loopback100
 description Anycast
 ipv4 address 100.100.100.12 255.255.255.255
!
router isis ACCESS
 interface Loopback100
 address-family ipv4 unicast
  prefix-sid index 1012 n-flag-clear 
</pre> 
</div> 

**L2VPN configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn                                                             
 bridge group Static-VPWS-H-L3VPN-IRB                             
  bridge-domain VRF1                                              
   neighbor 100.0.1.50 pw-id 5001                                 
    mpls static label local 5001 remote 5001                      
    pw-class static-pw-h-l3vpn-class                              
   !                                                              
   neighbor 100.0.1.51 pw-id 4001                                 
    mpls static label local 4001 remote 4001                      
    pw-class static-pw-h-l3vpn-class                              
   !                                                              
   routed interface BVI1                                          
    split-horizon group core                                      
   !                                                              
   evi 12001
   !
  !
</pre> 
</div> 

**EVPN configuration**

<div class="highlighter-rouge">
<pre class="highlight">
evpn
 evi 12001
 !
  advertise-mac
 !
 virtual neighbor 100.0.1.50 pw-id 5001
  ethernet-segment
   identifier type 0 12.00.00.00.00.00.50.00.01
</pre> 
</div> 

**Anycast IRB configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface BVI1
 host-routing
 vrf L3VPN-AnyCast-ODNTE-VRF1
 ipv4 address 12.0.1.1 255.255.255.0
 mac-address 12.0.1
 load-interval 30
</pre> 
</div> 

**VRF configuration**

<div class="highlighter-rouge">
<pre class="highlight">
vrf L3VPN-AnyCast-ODNTE-VRF1
 address-family ipv4 unicast
  import route-target
   100:10001
  !
  export route-target
   100:10001
  !
 !
!
</pre> 
</div> 

**BGP configuration**

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 vrf L3VPN-AnyCast-ODNTE-VRF1
  rd auto
  address-family ipv4 unicast
   redistribute connected
  !
 !
</pre> 
</div> 

![](http://xrdocs.io/design/images/cmfi/image15.png)

_Figure 15: L3VPN – Anycast Static Pseudowire (PW), MP-BGP VPNv4/6 with
Anycast IRB Datal Plane_

### L2/L3VPN – Anycast Static Pseudowire (PW), Multipoint EVPN with Anycast IRB

![](http://xrdocs.io/design/images/cmfi/image16.png)

_Figure 16: L2/L3VPN – Anycast Static Pseudowire (PW), Multipoint EVPN
with Anycast IRB Control Plane_

**Access Routers:** **Cisco NCS5501-SE IOS-XR or Cisco ASR920 IOS-XE**

5.  **Operator:** New Static Pseudowire (PW) instance via CLI or NSO

6.  **Access Router:** Path to PE Router is known via ACCESS-ISIS IGP.

**Provider Edge Routers:** **Cisco ASR9000 IOS-XR (Same on both PE
routers in same location PE1/2 and PE3/4)**

7.  **Operator:** New Static Pseudowire (PW) instance via CLI or NSO

8.  **Provider Edge Routers:** Path to Access Router is known via
    ACCESS-ISIS IGP.


9.  **Operator:** New L2VPN Multipoint EVPN instance together with
    Anycast IRB via CLI or NSO (Anycast IRB is optional when L2 and L3
    is required in same service instance)

10. **Provider Edge Routers:** Path to remote PEs is known via CORE-ISIS
    IGP.

**Please note that provisioning on Access and Provider Edge routers is
same as in “L3VPN – Anycast Static Pseudowire (PW), MP-BGP VPNv4/6 with
Anycast IRB”. In this use case there is BGP EVPN instead of MP-BGP
VPNv4/6 in the core.**

#### Access Router Service Provisioning (IOS-XR):

**VLAN based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group Static-VPWS-PE12-H-L3VPN-AnyCast
  p2p L3VPN-VRF1
   interface TenGigE0/0/0/2.1
   neighbor ipv4 100.100.100.12 pw-id 5001
    mpls static label local 5001 remote 5001
    pw-class static-pw-h-l3vpn-class
   !
  !
interface TenGigE0/0/0/2.1 l2transport
 encapsulation dot1q 1
 rewrite ingress tag pop 1 symmetric
!
l2vpn
 pw-class static-pw-h-l3vpn-class
  encapsulation mpls
   control-word
  !
</pre> 
</div> 

**Port based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 xconnect group Static-VPWS-PE12-H-L3VPN-AnyCast
  p2p L3VPN-VRF1
   interface TenGigE0/0/0/2
   neighbor ipv4 100.100.100.12 pw-id 5001
    mpls static label local 5001 remote 5001
    pw-class static-pw-h-l3vpn-class
   !
  !
!
interface TenGigE0/0/0/2 
 l2transport
!
l2vpn
 pw-class static-pw-h-l3vpn-class
  encapsulation mpls
   control-word
</pre> 
</div> 

#### Access Router Service Provisioning (IOS-XE):

**VLAN based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface GigabitEthernet0/0/5
 no ip address
 media-type auto-select
 negotiation auto
 service instance 1 ethernet
  encapsulation dot1q 1
  rewrite ingress tag pop 1 symmetric
  xconnect 100.100.100.12 4001 encapsulation mpls manual
   mpls label 4001 4001
   mpls control-word
 !
</pre> 
</div> 

**Port based service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface GigabitEthernet0/0/5
 no ip address
 media-type auto-select
 negotiation auto
 service instance 1 ethernet
  encapsulation default
  xconnect 100.100.100.12 4001 encapsulation mpls manual
   mpls label 4001 4001
   mpls control-word
 !
</pre> 
</div> 

#### Provider Edge Routers Service Provisioning (IOS-XR):

<div class="highlighter-rouge">
<pre class="highlight">
cef adjacency route override rib
</pre> 
</div> 

**AnyCast Loopback configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface Loopback100
 description Anycast
 ipv4 address 100.100.100.12 255.255.255.255
!
router isis ACCESS
 interface Loopback100
  address-family ipv4 unicast
   prefix-sid index 1012
</pre> 
</div> 

**L2VPN Configuration**

<div class="highlighter-rouge">
<pre class="highlight">
l2vpn                                                             
 bridge group Static-VPWS-H-L3VPN-IRB                             
  bridge-domain VRF1                                              
   neighbor 100.0.1.50 pw-id 5001                                 
    mpls static label local 5001 remote 5001                      
    pw-class static-pw-h-l3vpn-class                              
   !                                                              
   neighbor 100.0.1.51 pw-id 4001                                 
    mpls static label local 4001 remote 4001                      
    pw-class static-pw-h-l3vpn-class                              
   !                                                              
   routed interface BVI1                                          
    split-horizon group core                                      
   !                                                              
   evi 12001
   !
  !
</pre> 
</div> 

**EVPN configuration**

<div class="highlighter-rouge">
<pre class="highlight">
evpn
 evi 12001
  !
  advertise-mac
  !
 !
 virtual neighbor 100.0.1.50 pw-id 5001
  ethernet-segment
   identifier type 0 12.00.00.00.00.00.50.00.01
</pre> 
</div> 

**Anycast IRB configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface BVI1
 host-routing
 vrf L3VPN-AnyCast-ODNTE-VRF1
 ipv4 address 12.0.1.1 255.255.255.0
 mac-address 12.0.1
 load-interval 30
!
</pre> 
</div> 

**VRF configuration**

<div class="highlighter-rouge">
<pre class="highlight">
vrf L3VPN-AnyCast-ODNTE-VRF1
 address-family ipv4 unicast
  import route-target
   100:10001
  !
  export route-target
   100:10001
  !
 !
!
</pre> 
</div> 

**BGP configuration**

<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 vrf L3VPN-AnyCast-ODNTE-VRF1
  rd auto
  address-family ipv4 unicast
   redistribute connected
  !
 !
 
</pre> 
</div> 

![](http://xrdocs.io/design/images/cmfi/image17.png)

_Figure 17: L2/L3VPN – Anycast Static Pseudowire (PW), Multipoint EVPN
with Anycast IRB Data Plane_

## Remote PHY CIN Implementation 

### Summary 
Detail can be found in the CST 3.0 high-level design guide for design decisions, this section will provide sample configurations. 

### Sample QoS Policies 
The following are usable policies but policies should be tailored for specific network deployments.  

#### Class maps 
Class maps are used within a policy map to match packet criteria for further treatment 
<div class="highlighter-rouge">
<pre class="highlight">
class-map match-any match-ef-exp5
 description High priority, EF
 match dscp 46
 match mpls experimental topmost 5
 end-class-map
!
class-map match-any match-cs5-exp4
 description Second highest priority
 match dscp 40
 match mpls experimental topmost 4
 end-class-map
!
class-map match-any match-video-cs4-exp2
 description Video
 match dscp 32
 match mpls experimental topmost 2
 end-class-map
!
class-map match-any match-cs6-exp6
 description Highest priority control-plane traffic
 match dscp cs6
 match mpls experimental topmost 6
 end-class-map
!
class-map match-any match-qos-group-1
 match qos-group 1
 end-class-map
!
class-map match-any match-qos-group-2
 match qos-group 2
 end-class-map
!
class-map match-any match-qos-group-3
 match qos-group 3
 end-class-map
!
class-map match-any match-qos-group-6
 match qos-group 3
 end-class-map
!
class-map match-any match-traffic-class-1
 description "Match highest priority traffic-class 1"
 match traffic-class 1
 end-class-map
!
class-map match-any match-traffic-class-2
 description "Match high priority traffic-class 2"
 match traffic-class 2
 end-class-map
!
class-map match-any match-traffic-class-3
 description "Match medium traffic-class 3"
 match traffic-class 3
 end-class-map
!
class-map match-any match-traffic-class-6
 description "Match video traffic-class 6"
 match traffic-class 6
 end-class-map
</pre> 
</div>

#### RPD and DPIC interface policy maps
These are applied to all interfaces connected to cBR-8 DPIC and RPD devices. 

**Note:** Egress queueing maps are not supported on L3 BVI interfaces

**RPD/DPIC ingress classifier policy map** 

<div class="highlighter-rouge">
<pre class="highlight">
policy-map rpd-dpic-ingress-classifier
 class match-cs6-exp6
  set traffic-class 1
  set qos-group 1
 !
 class match-ef-exp5
  set traffic-class 2
  set qos-group 2
 !
 class match-cs5-exp4
  set traffic-class 3
  set qos-group 3
 !
 class match-video-cs4-exp2
  set traffic-class 6
  set qos-group 6
 !
 class class-default
  set traffic-class 0
  set dscp 0
  set qos-group 0
 !
 end-policy-map
!
</pre> 
</div>

**P2P RPD and DPIC egress queueing policy map** 

<div class="highlighter-rouge">
<pre class="highlight">
policy-map rpd-dpic-egress-queuing
 class match-traffic-class-1
  priority level 1
  queue-limit 500 us
 !
 class match-traffic-class-2
  priority level 2
  queue-limit 100 us
 !
 class match-traffic-class-3
  priority level 3
  queue-limit 500 us
 !
 class match-traffic-class-6
  priority level 6
  queue-limit 500 us
 !
 class class-default
  queue-limit 250 ms
 !
 end-policy-map
!
</pre> 
</div>

#### Core QoS 
Please see the general QoS section for core-facing QoS configuration  

### CIN Timing Configuration 
Please see the G.8275.2 timing configuration guide in this document for details on timing configuration. The following values should be used for PTP configuration attributes.  Please note in CST 3.0 the use of an IOS-XR router as a Boundary Clock is only supported on P2P L3 interfaces. The use of a BVI for RPD aggregation requires the BC used for RPD nodes be located upstream, or alternatively a physical loopback cable may be used to provide timing off the IOS-XR based RPD leaf device.   

| PTP variable | IOS-XR configuration value | IOS-XE value | 
| ---------- | --------- | -------------- | 
| Announce Interval | 1 | 1 | 
| Announce Timeout  | 5 | 5 | 
| Sync Frequency | 16 | -4 | 
| Delay Request Frequency | 16 | -4 | 

#### Example CBR-8 RPD DTI Profile 
<div class="highlighter-rouge">
<pre class="highlight">
ptp r-dti 4
 profile G.8275.2
 ptp-domain 60
 clock-port 1
   clock source ip 192.168.3.1
   sync interval -4
   announce timeout 5
   delay-req interval -4
</pre> 
</div> 

### Multicast configuration 

#### Summary 
We present two different configuration options based on either native multicast deployment or the use of a L3VPN to carry Remote PHY traffic. The L3VPN option shown uses Label Switched Multicast profile 14 (partitioned mLDP) however profile 6 could also be utilized. 

#### Global multicast configuration - Native multicast 
On CIN aggregation nodes all interfaces should have multicast enabled.  
<div class="highlighter-rouge">
<pre class="highlight">
multicast-routing
 address-family ipv4
  interface all enable
 !
 address-family ipv6
  interface all enable  
   enable
 !
</pre> 
</div>

#### Global multicast configuration - LSM using profile 14 
On CIN aggregation nodes all interfaces should have multicast enabled.  
<div class="highlighter-rouge">
<pre class="highlight">
vrf VRF-MLDP
  address-family ipv4
   mdt source Loopback0
   rate-per-route
   interface all enable
   accounting per-prefix
   bgp auto-discovery mldp
   !
   mdt partitioned mldp ipv4 p2mp
   mdt data 100
  !
 !
</pre> 
</div>

#### PIM configuration - Native multicast
PIM should be enabled for IPv4/IPv6 on all core facing interfaces 
<div class="highlighter-rouge">
<pre class="highlight">
router pim
 address-family ipv4
  interface Loopback0
   enable
  !
  interface TenGigE0/0/0/6
   enable
  !
  interface TenGigE0/0/0/7
   enable
  !
 !
</pre> 
</div>

#### PIM configuration - LSM using profile 14
The PIM configuration is utilized even though no PIM neighbors may be connected.   
<div class="highlighter-rouge">
<pre class="highlight">
route-policy mldp-partitioned-p2mp
  set core-tree mldp-partitioned-p2mp
end-policy
!
router pim
 address-family ipv4
  interface Loopback0
   enable
 vrf rphy-vrf
  address-family ipv4
   rpf topology route-policy mldp-partitioned-p2mp  
   mdt c-multicast-routing bgp
   !
  !
</pre> 
</div>

#### IGMPv3/MLDv2 configuration - Native multicast 
Interfaces connected to RPD and DPIC interfaces should have IGMPv3 and MLDv2 enabled 
<div class="highlighter-rouge">
<pre class="highlight">
router igmp
 interface BVI100
  version 3
 !
 interface TenGigE0/0/0/25  
  version 3
 !
!
router mld
 interface BVI100
  version 2
 interface TenGigE0/0/0/25  
  version 3
 !
 !
</pre> 
</div>

#### IGMPv3/MLDv2 configuration - LSM profile 14  
Interfaces connected to RPD and DPIC interfaces should have IGMPv3 and MLDv2 enabled as needed  
<div class="highlighter-rouge">
<pre class="highlight">
router igmp
 vrf rphy-vrf
  interface BVI101
   version 3
  !
  interface TenGigE0/0/0/15
  !
 !
!
router mld
 vrf rphy-vrf
  interface TenGigE0/0/0/15
   version 2
  !
 !
!
</pre> 
</div>

#### IGMPv3 / MLDv2 snooping profile configuration (BVI aggregation)
In order to limit L2 multicast replication for specific groups to only interfaces with interested receivers, IGMP and MLD snooping must be enabled.  

<div class="highlighter-rouge">
<pre class="highlight">
igmp snooping profile igmp-snoop-1
!
mld snooping profile mld-snoop-1
!
</pre> 
</div>

### RPD DHCPv4/v6 relay configuration 
In order for RPDs to self-provision DHCP relay must be enabled on all RPD-facing L3 interfaces. In IOS-XR the DHCP relay configuration is done in its own configuration context without any configuration on the interface itself. 

#### Native IP / Default VRF  
<div class="highlighter-rouge">
<pre class="highlight">
dhcp ipv4
 profile rpd-dhcpv4 relay
  helper-address vrf default 10.0.2.3
 !
 interface BVI100 relay profile rpd-dhcpv4
!
dhcp ipv6
 profile rpd-dhcpv6 relay
  helper-address vrf default 2001:10:0:2::3
  iana-route-add
  source-interface BVI100
 !
 interface BVI100 relay profile rpd-dhcpv6
</pre> 
</div>

#### RPHY L3VPN  
In this example it is assumed the DHCP server exists within the rphy-vrf VRF, if it does not then additional routing may be necessary to forward packets between VRFs. 

<div class="highlighter-rouge">
<pre class="highlight">
dhcp ipv4
 vrf rphy-vrf relay profile rpd-dhcpv4-vrf
 profile rpd-dhcpv4-vrf relay
  helper-address vrf rphy-vrf 10.0.2.3
  relay information option allow-untrusted
 !
 inner-cos 5
 outer-cos 5
 interface BVI101 relay profile rpd-dhcpv4-vrf
 interface TenGigE0/0/0/15 relay profile rpd-dhcpv4-vrf
!
</pre> 
</div>

### cBR-8 DPIC interface configuration without Link HA
Without link HA the DPIC port is configured as a normal physical interface   
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/25
 description .. Connected to cbr8 port te1/1/0
 service-policy input rpd-dpic-ingress-classifier
 service-policy output rpd-dpic-egress-queuing
 ipv4 address 4.4.9.101 255.255.255.0
 ipv6 address 2001:4:4:9::101/64
 carrier-delay up 0 down 0
 load-interval 30
</pre> 
</div>

### cBR-8 DPIC interface configuration with Link HA
When using Link HA faster convergence is achieved when each DPIC interface is placed into a BVI with a statically assigned MAC address. Each DPIC interface is placed into a separate bridge-domain with a unique BVI L3 interface. The same MAC address should be utilized on all BVI interfaces.  Convergence using BVI interfaces is <50ms, L3 physical interfaces is 1-2s.   

**Even DPIC port CIN interface configuration** 
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/25
 description "Connected to cBR8 port Te1/1/0" 
 lldp
 !
 carrier-delay up 0 down 0
 load-interval 30
 l2transport
 !
!
l2vpn
 bridge group cbr8
  bridge-domain port-ha-0
   interface TenGigE0/0/0/25
   !
   routed interface BVI500
   !
  !
 !
 interface BVI500
 description "BVI for cBR8 port HA, requires static MAC"
 service-policy input rpd-dpic-ingress-classifier
 ipv4 address 4.4.9.101 255.255.255.0
 ipv6 address 2001:4:4:9::101/64
 mac-address 8a.9698.64
 load-interval 30
!
</pre> 
</div>

**Odd DPIC port CIN interface configuration** 
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/26
 description "Connected to cBR8 port Te1/1/1" 
 lldp
 !
 carrier-delay up 0 down 0
 load-interval 30
 l2transport
 !
!
l2vpn
 bridge group cbr8
  bridge-domain port-ha-1 
   interface TenGigE0/0/0/26
   !
   routed interface BVI501
   !
  !
 !
 interface BVI501
 description "BVI for cBR8 port HA, requires static MAC"
 service-policy input rpd-dpic-ingress-classifier
 ipv4 address 4.4.9.101 255.255.255.0
 ipv6 address 2001:4:4:9::101/64
 mac-address 8a.9698.64
 load-interval 30
!
</pre> 
</div>

### cBR-8 Digital PIC Interface Configuration  
<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/25
 description .. Connected to cbr8 port te1/1/0
 service-policy input rpd-dpic-ingress-classifier
 service-policy output rpd-dpic-egress-queuing
 ipv4 address 4.4.9.101 255.255.255.0
 ipv6 address 2001:4:4:9::101/64
 carrier-delay up 0 down 0
 load-interval 30
</pre> 
</div>

### RPD interface configuration

#### P2P L3
In this example the interface has PTP enabled towards the RPD  
<div class="highlighter-rouge">
<pre class="highlight">
interface TeGigE0/0/0/15  
 description To RPD-1
 mtu 9200
 ptp
  profile g82752_master_v4
 !  
 service-policy input rpd-dpic-ingress-classifier
 service-policy output rpd-dpic-egress-queuing 
 ipv4 address 192.168.2.0 255.255.255.254 
 ipv6 address 2001:192:168:2::0/127 
 ipv6 enable
 !
</pre> 
</div>

#### BVI
<div class="highlighter-rouge">
<pre class="highlight">
l2vpn
 bridge group rpd
  bridge-domain rpd-1
   mld snooping profile mld-snoop-1
   igmp snooping profile igmp-snoop-1
   interface TenGigE0/0/0/15
   !
   interface TenGigE0/0/0/16
   !
   interface TenGigE0/0/0/17
   !
   routed interface BVI100
   !
   !
  !
 !
!
interface BVI100
 description ... to downstream RPD hosts  
 service-policy input rpd-dpic-ingress-classifier
 ipv4 address 192.168.2.1 255.255.255.0
 ipv6 address 2001:192:168:2::1/64
 ipv6 enable
 !
</pre> 
</div>

### RPD/DPIC agg device IS-IS configuration 
The standard IS-IS configuration should be used on all core interfaces with the addition of specifying all DPIC and RPD connected as IS-IS passive interfaces. Using passive interfaces is preferred over redistributing connected routes. This configuration is needed for reachability between DPIC and RPDs across the CIN network.   

<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
 interface TenGigE0/0/0/25
  passive
  address-family ipv4 unicast
  !
  address-family ipv6 unicast
</pre> 
</div>

### Additional configuration for L3VPN Design 

#### Global VRF Configuration 
This configuration is required on all DPIC and RPD connected routers as well as ancillary elements communicating with Remote PHY elements   
<div class="highlighter-rouge">
<pre class="highlight">
vrf rphy-vrf
 address-family ipv4 unicast
  import route-target
   100:5000
  !
  export route-target
   100:5000
  !
 !
 address-family ipv6 unicast
  import route-target
   100:5000
  !
  export route-target
   100:5000
  !
 !
</pre> 
</div>

#### BGP Configuration
This configuration is required on all DPIC and RPD connected routers as well as ancillary elements communicating with Remote PHY elements   
<div class="highlighter-rouge">
<pre class="highlight">
router bgp 100
 vrf rphy-vrf
  rd auto
  address-family ipv4 unicast
   label mode per-vrf 
   redistribute connected
  !
  address-family ipv6 unicast
   label mode per-vrf 
   redistribute connected
  !
  address-family ipv4 mvpn
  !
  address-family ipv6 mvpn
  !
 !
</pre> 
</div>

# Model-Driven Telemetry Configuration 

## Summary 
This is not an exhaustive list of IOS-XR model-driven telemetry sensor paths, but gives some basic paths used to monitor a Converged SDN Transport deployment. Each sensor path may have its own 
cadence of collection and transmission, but it's recommended to not use values less than 60s when using many sensor paths.  

<br> 

## Device inventory and monitoring
| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
| Full inventory via OpenConfig model| openconfig-platform:components|
| NCS 540/5500 NPU resources |cisco-ios-xr-fretta-bcm-dpa-hw-resources-oper/dpa/stats/nodes/node/hw-resources-datas/hw-resources-data|
| Optics information |cisco-ios-xr-controller-optics-oper:optics-oper/optics-ports/optics-port/optics-info|
| System uptime| cisco-ios-xr-shellutil-oper:system-time/uptime|
| System CPU utilization| cisco-ios-xr-wdsysmon-fd-oper:system-monitoring/cpu-utilization|
<br> 

## Interface Data

| Metric | Sensor path | 
-----------------------| ---------------------------------------------------|
|Interface optics state:  Cisco-IOS-XR-controller-optics-oper:optics-oper/optics-ports/optics-port/optics-info/transport-admin-state|
|OpenConfig interface stats|openconfig-interfaces:interfaces|
|Interface data rates, based on load-interval|Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/data-rate| 
|Interface counters similar to "show int"|Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters|
|Full interface information|Cisco-IOS-XR-pfi-im-cmd-oper:interfaces/interface-xr/interface|
|Interface stats|Cisco-IOS-XR-pfi-im-cmd-oper:interfaces/interface-xr/interface/interface-statistics|
|Subset of interface stats|Cisco-IOS-XR-pfi-im-cmd-oper:interfaces/interface-xr/interface/interface-statistics/basic-interface-stats|
<br> 

## LLDP Monitoring 

| Metric | Sensor path | 
-----------------------| ---------------------------------------------------|
|All LLDP Info| Cisco-IOS-XR-ethernet-lldp-oper:lldp|
|LLDP neighbor info|Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors|
<br> 

## Aggregate bundle information (use interface models for interface counters)

| Metric | Sensor path | 
-----------------------| ---------------------------------------------------|
|OpenConfig LAG information|sensor-group openconfig-if-aggregate:aggregate|
|OpenConfig LAG state only|sensor-group openconfig-if-aggregate:aggregate/state|
|OpenConfig LACP information|sensor-group openconfig-lacp:lacp|
|Cisco full bundle information|sensor-group Cisco-IOS-XR-bundlemgr-oper:bundles|
|Cisco BFD over Bundle stats|sensor-group Cisco-IOS-XR-bundlemgr-oper:bundle-information/bfd-counters|
<br> 

## PTP and SyncE Information 

| Metric | Sensor path | 
-----------------------| ---------------------------------------------------|
|PTP servo status |  Cisco-IOS-XR-ptp-oper:ptp/platform/servo/device-status |
|PTP servo statistics | Cisco-IOS-XR-ptp-oper:ptp/platform/servo |
|PTP foreign master information |  Cisco-IOS-XR-ptp-oper:ptp/interface-foreign-masters |
|PTP interface counters, key is interface name | Cisco-IOS-XR-ptp-oper:ptp/interface-packet-counters | 
|Frequency sync info |  Cisco-IOS-XR-freqsync-oper:frequency-synchronization/summary/frequency-summary |
|SyncE interface information, key is interface name | Cisco-IOS-XR-freqsync-oper:frequency-synchronization/interface-datas/interface-data |
<br> 

## BGP Information  

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|BGP established neighbor count across all AF | Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/vrfs/vrf/process-info/global/established-neighbors-count-total|
|BGP total neighbor count|  Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/vrfs/vrf/process-info/global/neighbors-count-total|
|BGP prefix SID count| Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/vrfs/vrf/process-info/global/prefix-sid-label-index-count|
|BGP total VRF count including default VRF| Cisco-IOS-XR-ipv4-bgp-oper:process-info/ipv4-bgp-oper:global/ipv4-bgp-oper:total-vrf-count|
|BGP convergence|Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/default-vrf/afs/af/af-process-info/performance-statistics/global/|has-converged|
|BGP IPv4 route count| Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-bgp-ext/active-routes-count|
|OpenConfig BGP information|openconfig-bgp:bgp|
|OpenConfig BGP neighbor info only| openconfig-bgp:bgp/neighbors|
<br> 

## IS-IS Information

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|IS-IS neighbor info| sensor-path Cisco-IOS-XR-clns-isis-oper:isis/instances/instance/neighbors|
|IS-IS interface info| sensor-path Cisco-IOS-XR-clns-isis-oper:isis/instances/instance/levels/interfaces|
|IS-IS adj information| sensor-path Cisco-IOS-XR-clns-isis-oper:isis/instances/instance/levels/adjacencies|
|IS-IS neighbor summary| sensor-path Cisco-IOS-XR-clns-isis-oper:isis/instances/instance/neighbor-summaries|
|IS-IS node count|  Cisco-IOS-XR-clns-isis-oper:isis/instances/instance/topologies/topology/topology-levels/topology-level/topology-summary/router-node-count/reachable-node-count|
|IS-IS adj state| Cisco-IOS-XR-clns-isis-oper:isis/instances/instance/levels/level/adjacencies/adjacency/adjacency-state|
|IS-IS neighbor count|  Cisco-IOS-XR-clns-isis-oper:isis/instances/instance/neighbor-summaries/neighbor-summary/level2-neighbors/neighbor-up-count|
|IS-IS total route count|  Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-isis-l2/active-routes-count|
<br> 

## Routing protocol RIB information 

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|IS-IS L1 Info|Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-isis-l1|
|IS-IS L2 Info|Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-isis-l2
|IS-IS Summary|Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-isis-sum|
|Total route count per protocol|Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count|
|IPv6 IS-IS L1 info|Cisco-IOS-XR-ip-rib-ipv6-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-isis-l1|
|IPv6 IS-IS L2 info|Cisco-IOS-XR-ip-rib-ipv6-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-isis-l2|
|IPv6 IS-IS summary|Cisco-IOS-XR-ip-rib-ipv6-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-isis-sum|
|IPv6 total route count per protocol|Cisco-IOS-XR-ip-rib-ipv6-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count|
<br> 

## BGP RIB information
<b>It is not recommended to monitor these paths using MDT with large tables<b> 

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|openconfig-rib-bgp:bgp-rib|
|Cisco-IOS-XR-ip-rib-ipv6-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-bgp-ext|
|Cisco-IOS-XR-ip-rib-ipv6-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-bgp-int|
|Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-bgp-ext|
|Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/rtype-bgp-int|
<br>

## Routing policy Information 

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|Routing policy information | Cisco-IOS-XR-policy-repository-oper:routing-policy/policies|
<br>

## EVPN Information 

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|EVPN information|  Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/evpn-summary|
|Total EVPN|  Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/evpn-summary/total-count|
|EVPN total ES entries| Cisco-IOS-XR-evpn-oper:evpn/active/summary/es-entries|
|EVPN local Eth Auto Discovery routes| Cisco-IOS-XR-evpn-oper:evpn/active/summary/local-ead-routes|
|EVPN remote Eth Auto Discovery routes| Cisco-IOS-XR-evpn-oper:evpn/active/summary/remote-ead-routes|
<br> 

## Per-Interface QoS Statistics Information 

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|Input stats | Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/statistics/|
|General QoS Stats | Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/statistics/class-stats/general-stats|
|Per-queue stats | Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/statistics/class-stats/queue-stats-array|
|General service policy information, keys are policy name and interface applied|  Cisco-IOS-XR-qos-ma-oper:qos/interface-table/interface/input/service-policy-names|
<br> 

## Per-Policy, Per-Interface, Per-Class statistics 

<b>See sensor path name for detailed information on data leafs</b>  

|  Sensor path | 
|-----------------------| 
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/match-data-rate|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/pre-policy-matched-bytes|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/pre-policy-matched-packets|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/total-drop-bytes|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/total-drop-packets|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/total-drop-rate|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/total-transmit-rate|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/general-stats/transmit-bytes|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/queue-stats-array/queue-instance-length/value|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/queue-stats-array/queue-max-length/unit|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/queue-stats-array/queue-max-length/value|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/queue-stats-array/random-drop-bytes|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/queue-stats-array/random-drop-packets|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/queue-stats-array/tail-drop-bytes|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/queue-stats-array/tail-drop-packets|
|Cisco-IOS-XR-qos-ma-oper:qos/nodes/node/policy-map/interface-table/interface/input/service-policy-names/service-policy-instance/statistics/class-stats/shared-queue-id|
<br> 

## L2VPN Information  

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|L2VPN general forwarding information including EVPN and Bridge Domains| Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary|
|Bridge domain information| Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/bridge-domain-summary| 
|Total BDs active|  Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/bridge-domain-summary/bridge-domain-count|
|Total BDs using EVPN| Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/bridge-domain-summary/bridge-domain-with-evpn-enabled|
|Total MAC count (Local+remote)| Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/mac-summary/mac-count|
|L2VPN xconnect Forwarding information| Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/xconnect-summary|
|Xconnect total count|  Cisco-IOS-XR-l2vpn-oper:l2vpnv2/active/xconnect-summary/number-xconnects|
|Xconnect down count|  Cisco-IOS-XR-l2vpn-oper:l2vpnv2/active/xconnect-summary/number-xconnects-down|
|Xconnect up count| Cisco-IOS-XR-l2vpn-oper:l2vpnv2/active/xconnect-summary/number-xconnects-up|
|Xconnect unresolved| Cisco-IOS-XR-l2vpn-oper:l2vpnv2/active/xconnect-summary/number-xconnects-unresolved|
|Xconnect with down attachment circuits|  Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-summary/xconnect-summary/ac-down-count-l2vpn|
|Per-xconnect detailed information including state| xconnect group and name are keys:  Cisco-IOS-XR-l2vpn-oper:l2vpnv2/active/xconnects/xconnect|
|L2VPN bridge domain specific information, will have the BD name as a key|  Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-bridge-domains/l2fib-bridge-domain|
|L2VPN EVPN IPv4 MAC/IP information|  Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-evpn-ip4macs|
|L2VPN EVPN IPv6 MAC/IP information|  Cisco-IOS-XR-l2vpn-oper:l2vpn-forwarding/nodes/node/l2fib-evpn-ip6macs|
<br> 

## SR-PCE PCC and SR Policy Information 

| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|PCC to PCE peer information|  Cisco-IOS-XR-infra-xtc-agent-oper:pcc/peers|
|SR policy summary info|  Cisco-IOS-XR-infra-xtc-agent-oper:xtc/policy-summary|
|Specific SR policy information| Cisco-IOS-XR-infra-xtc-agent-oper:xtc/policy-summary/configured-down-policy-count|
|Specific SR policy information| Cisco-IOS-XR-infra-xtc-agent-oper:xtc/policy-summary/configured-total-policy-count|
|Specific SR policy information| Cisco-IOS-XR-infra-xtc-agent-oper:xtc/policy-summary/configured-up-policy-count|
|SR policy information, key is SR policy name|  Cisco-IOS-XR-infra-xtc-agent-oper:xtc/policies/policy|
|SR policy forwarding info including packet and byte stats per candidate path, key is policy name and candidate path|  Cisco-IOS-XR-infra-xtc-agent-oper:xtc/policy-forwardings|
<br> 

## MPLS performance measurement  
| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|Summary info|  Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/summary|
|Interface stats for delay measurements|  Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/summary/delay-summary/interface-delay-summary/delay-transport-counters/generic-counters|
|Interface stats for loss measurement|   Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/summary/loss-summary/interface-loss-summary|
|SR policy PM statistics|  Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/sr-policies/sr-policy-delay |
|Parent interface oper data sensor path|  Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/interfaces |
|Delay values for each probe measurement| Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/interfaces/delay/interface-last-probes|
|Delay values aggregated at computation interval| Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/interfaces/delay/interface-last-aggregations|
|Delay values aggregated at advertisement interval| Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/interfaces/delay/interface-last-advertisements|
|SR Policy measurement information|  Cisco-IOS-XR-perf-meas-oper:performance-measurement/nodes/node/sr-policies|
<br> 

## mLDP Information 
| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|mLDP LSP count|  Cisco-IOS-XR-mpls-ldp-mldp-oper:mpls-mldp/active/default-context/context/lsp-count|
|mLDP peer count|  Cisco-IOS-XR-mpls-ldp-mldp-oper:mpls-mldp/active/default-context/context/peer-count|
|mLDP database info, where specific LSP information is stored|  Cisco-IOS-XR-mpls-ldp-mldp-oper:mpls-mldp/active/default-context/databases/database|
<br> 

## ACL Information
| Metric | Sensor path | 
|-----------------------| ---------------------------------------------------|
|Details on ACL resource consumption|  Cisco-IOS-XR-ipv4-acl-oper:ipv4-acl-and-prefix-list/oor/access-list-summary/details/current-configured-ac-es|
|OpenConfig full ACL information | openconfig-acl:acl|
<br> 



```