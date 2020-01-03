---
published: true
date: '2018-05-09 16:54 -0500'
title: Metro Design Implementation Guide
position: hidden 
author: Phil Bedard 
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

![](http://xrdocs.io/design/images/cmf-rphy-dpic-redundancy.png)

_Figure 1: Compass Metro Fabric High Level Topology_

![](http://xrdocs.io/design/images/cmf-rphy-dpic-redundancy.png)

_Figure 2: Testbed Physical Topology_

![](http://xrdocs.io/design/images/cmf-rphy-dpic-redundancy.png)

_Figure 3: Testbed Route-Reflector and SR-PCE physical connectivity_

![]({{site.baseurl}}/images/cmfi/image4.png)

_Figure 4: Testbed IGP Domains_

## Devices

**Access PE (A-PE) Routers**

  - Cisco NCS5501-SE (IOS-XR) – A-PE7
  - Cisco NCS540 (IOS-XR) - A-PE1, A-PE2, A-PE3, A-PE8 
  - Cisco ASR920 (IOS-XE) – A-PE4, A-PE5, A-PE6, A-PE9

**Pre-Aggregation (PA) Routers**

  - Cisco NCS5501-SE (IOS-XR) – PA3, PA4  

**Aggregation (PA) Routers**

  - Cisco NCS5501-SE (IOS-XR) – AG1, AG2, AG3, AG4 

**High-scale Provider Edge Routers**

  - Cisco ASR9000 (IOS-XR) – PE1, PE2, PE3, PE4 

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
    
## IOS-XR Nodes - SR-MPLS Transport 

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
</div> 
</pre> 

### SRGB and SRLB Definition 
It's recommended to first configure the Segment Routing Global Block (SRGB) across all nodes needing connectivity between each other. In most instances a single SRGB will be used across the entire network. In a SR MPLS deployment the SRGB and SRLB correspond to the label blocks allocated to SR. IOS-XR has a maximum configurable SRGB limit of 512,000 labels, however please consult platform-specific documentation for maximum values. The SRLB corresponds to the labels allocated for SIDs local to the node, such as Adjacency-SIDs. It is recommended to configure the same SRLB block across all nodes. The SRLB must not overlap with the SRGB.  The SRGB and SRLB are configured in IOS-XR with the following configuration:   

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 global-block 16000 23999
 local-block 15000 15999
</pre> 
</div>

### IGP protocol (ISIS) and Segment Routing MPLS configuration

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

#### ISIS router configuration

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
  spf-interval maximum-wait 1000 initial-wait 5 secondary-wait 100
  segment-routing mpls
  spf prefix-priority high tag 1000
  maximum-redistributed-prefixes 100 level 2
 ! 
 address-family ipv6 unicast
  metric-style wide
  spf-interval maximum-wait 5000 initial-wait 50 secondary-wait 200
  maximum-redistributed-prefixes 100 level 2
</div> 
</pre> 

ABR Loopback 0 on domain boundary is part of both IGP processes together with same “prefix-sid absolute” value
{: .notice--success}

The prefix SID can be configured as either _absolute_ or _index_.  The _index_ configuration is required for interop with nodes using a different SRGB. 
{: .notice--success}

#### IS-IS Loopback and node SID configuration
<div class="highlighter-rouge">
<pre class="highlight">
 interface Loopback0
  ipv4 address 100.0.1.50 255.255.255.255
  address-family ipv4 unicast
   <b>prefix-sid absolute 16150</b>
   tag 1000 
</div> 
</pre> 


#### IS-IS interface configuration with TI-LFA

It is recommended to use manual adjacency SIDs. A _protected_ SID is eligible for backup path computation, meaning if a packet ingresses the node with the label a backup path will be provided in case of a failure. In the case of having multiple adjacencies between the same two nodes, use the same adjacency-sid on each link. 

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
</div> 
</pre> 


### MPLS Segment Routing Traffic Engineering (SRTE) configuration
The following configuration is done at the global ISIS configuration level and should be performed for all IOS-XR nodes.   

<div class="highlighter-rouge">
<pre class="highlight">
router isis ACCESS
 address-family ipv4 unicast
  mpls traffic-eng level-2-only
  mpls traffic-eng router-id Loopback0
</div> 
</pre> 

#### MPLS Segment Routing Traffic Engineering (SRTE) TE metric configuration  

The TE metric is used when computing SR Policy paths with the "te" or "latency" constraint type.  The TE metric is carried as a TLV within the TE opaque LSA distributed across the IGP area and to the PCE via BGP-LS.  
The TE metric is used in the CST 5G Transport use case.  If no TE metric is defined the local CSPF or PCE will utilize the IGP metric.   

<div class="highlighter-rouge">
<pre class="highlight">
segment-routing
 traffic-eng
  interface TenGigE0/0/0/6
   metric 1000
</div> 
</pre>

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
</div> 
</pre>

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

</div> 
</pre>
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
prefix redistribution between IS-IS processes. The ABR nodes have static hold-down routes for the 
block of IP space used in each domain across the network.  

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

router isis CORE
address-family ipv4 unicast
  distance 254 0.0.0.0/0 ACCESS-PCE_SvRR-LOOPBACKS
  redistribute static route-policy CORE-TO-ACCESS1
</pre> 
</div> 

## Multicast transport using mLDP 
### Overview 
This portion of the implementation guide instructs the user how to configure mLDP end to end across the multi-domain network. Multicast service examples are given in the "Services" section of the implementation guide. 

### mLDP base configuration 
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

## Area Border Routers (ABRs) IGP topology distribution

Next network diagram: “BGP-LS Topology Distribution” shows how Area
Border Routers (ABRs) distribute IGP network topology from ISIS ACCESS
and ISIS CORE to Transport Route-Reflectors (tRRs). tRRs then reflect
topology to Segment Routing Path Computation Element (SR-PCEs). Each SR-PCE has full 
visibility of the entire inter-domain network. 

Each IS-IS process in the network requires a unique instance-id to identify itself to the PCE.
{: .notice--warning}

![]({{site.baseurl}}/images/cmfi/image5.png)

_Figure 5: BGP-LS Topology Distribution_

<div class="highlighter-rouge">
<pre class="highlight">

router isis ACCESS
 **distribute link-state instance-id 101**
 net 49.0001.0101.0000.0001.00
 address-family ipv4 unicast
  mpls traffic-eng router-id Loopback0

router isis CORE
 **distribute link-state instance-id 100**
 net 49.0001.0100.0000.0001.00
 address-family ipv4 unicast
  mpls traffic-eng router-id Loopback0

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

route-map L3VPN-ODN-TE-INIT permit 10
 match community 1
 set attribute-set L3VPN-SRTE
!
route-map L3VPN-SR-ODN-Mark-Comm permit 10
 match ip address L3VPN-ODN-Prefixes
 set community 9999
!
!
end

router bgp 100
 address-family vpnv4
  neighbor SvRR send-community both
  neighbor SvRR route-map L3VPN-ODN-TE-INIT in
  neighbor SvRR route-map L3VPN-SR-ODN-Mark-Comm out
</pre> 
</div> 

### Preferred Path configuration – IOS-XR

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

### Preferred Path configuration – IOS-XE

</pre> 
</div> 
mpls traffic-eng tunnels
mpls traffic-eng pcc peer 100.0.1.101 source 100.0.1.51
mpls traffic-eng pcc peer 100.0.1.111 source 100.0.1.51
mpls traffic-eng pcc report-all
</pre> 
</div> 

# Services
    
## End-To-End Services

![]({{site.baseurl}}/images/cmfi/image6.png)

_Figure 6: End-To-End Services Table_

### L3VPN MP-BGP VPNv4 On-Demand Next-Hop

![]({{site.baseurl}}/images/cmfi/image7.png)

_Figure 7: L3VPN MP-BGP VPNv4 On-Demand Next-Hop Control Plane_

**Access Routers:** **Cisco ASR920 IOS-XE**

1.  **Operator:** New VPNv4 instance via CLI or NSO

2.  **Access Router:** Advertises/receives VPNv4 routes to/from Services
    Route-Reflector (sRR)

3.  **Access Router**: Request SR-PCE to provide path (shortest IGP metric)
    to remote access router

4.  **SR-PCE:** Computes and provides the path to remote router(s)

5.  **Access Router:** Programs Segment Routing Traffic Engineering
    (SRTE) Policy to reach remote access router

Please refer to “**On Demand Next-Hop (ODN) – IOS-XE**” section for
initial ODN configuration.

#### Access Router Service Provisioning (IOS-XE):

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

![]({{site.baseurl}}/images/cmfi/image8.png)

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

Please refer to “**On Demand Next-Hop (ODN) – IOS-XR**” section for
initial ODN configuration.

#### Access Router Service Provisioning (IOS-XR):

**PORT Based service configuration**

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

![]({{site.baseurl}}/images/cmfi/image9.png)

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

Note EVPN VPWS dual homing is not supported when using an SR-TE preferred path.  
{: .notice--warning}

In IOS-XR 6.6.3 the SR Policy used as the preferred path must be referenced by its generated name and not the configured policy name.  This requires first issuing the command   
{: .notice--info}

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

SR-TE policy database
---------------------

Color: 1001, End-point: 100.0.1.50
  Name: **srte_c_1001_ep_100.0.1.50**
</pre> 
</div> 

**Port Based Service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/15 
  l2transport

l2vpn 
 pw-class static-pw-class-PE7
  encapsulation mpls
   control-word
   preferred-path sr-te policy GREEN-PE7

 p2p Static-PW-to-PE7-1
  interface TenGigE0/0/0/15
   neighbor ipv4 100.0.2.52 pw-id 1000                      
    mpls static label local 1000 remote 1000 pw-class static-pw-class-PE7   
</pre> 
</div> 


**VLAN Based Service configuration**

<div class="highlighter-rouge">
<pre class="highlight">
interface TenGigE0/0/0/5.1001 l2transport
 encapsulation dot1q 1001
 rewrite ingress tag pop 1 symmetric

l2vpn 
 pw-class static-pw-class-PE7
  encapsulation mpls
   control-word
   preferred-path sr-te policy GREEN-PE7
  p2p Static-PW-to-PE7-2                                                                                                                                      
   interface TenGigE0/0/0/5.1001
    neighbor ipv4 100.0.2.52 pw-id 1001                      
     mpls static label local 1001 remote 1001 pw-class static-pw-class-PE7 
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

pseudowire-class mpls                                                     
 encapsulation mpls                                                       
 no control-word                                                          
 protocol none                                                            
 preferred-path interface Tunnel1  
</pre> 
</div> 

### End-To-End Services Data Plane

![]({{site.baseurl}}/images/cmfi/image10.png)

_Figure 10: End-To-End Services Data Plane_

## Hierarchical Services

![]({{site.baseurl}}/images/cmfi/image11.png)

_Figure 11: Hierarchical Services Table_

### L3VPN – Single-Homed EVPN-VPWS, MP-BGP VPNv4/6 with Pseudowire-Headend (PWHE)

![]({{site.baseurl}}/images/cmfi/image12.png)

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

![]({{site.baseurl}}/images/cmfi/image13.png)

_Figure 13: L3VPN – Single-Homed EVPN-VPWS, MP-BGP VPNv4/6 with
Pseudowire-Headend (PWHE) Data Plane_

### L3VPN – Anycast Static Pseudowire (PW), MP-BGP VPNv4 with Anycast IRB

![]({{site.baseurl}}/images/cmfi/image14.png)

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
 prefix-sid index 1012
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

![]({{site.baseurl}}/images/cmfi/image15.png)

_Figure 15: L3VPN – Anycast Static Pseudowire (PW), MP-BGP VPNv4/6 with
Anycast IRB Datal Plane_

### L2/L3VPN – Anycast Static Pseudowire (PW), Multipoint EVPN with Anycast IRB

![]({{site.baseurl}}/images/cmfi/image16.png)

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
  
interface TenGigE0/0/0/2 
 l2transport
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

![]({{site.baseurl}}/images/cmfi/image17.png)

_Figure 17: L2/L3VPN – Anycast Static Pseudowire (PW), Multipoint EVPN
with Anycast IRB Data Plane_
