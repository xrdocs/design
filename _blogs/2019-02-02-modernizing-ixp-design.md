---
published: false 
date: '2019-02-02 11:00-0400'
title: 'Modernizing IX Fabric Design Using Segment Routing and EVPN' 
excerpt: IX fabrics began as very simple L2 switching designs but have evolved to worldwide interconnection networks supporting Terabits of traffic. SR and EVPN transform simple IX networks into flexible and resilient fabrics support any service type at any location in the fabric.'    
author: Phil Bedard
tags:
  - iosxr
  - Peering
  - Design
  - Internet Exchange
position: hidden 
---

{% include toc %}

Modern IX Fabric Design 
============================

## Internet Exchange History 
-------------------------
### Initial Exchanges 
The Internet was founded on a loosely coupled open interconnectivity model. The ability for two networks to use a simple protocol to exchange IP routing data was essential in the growth of the initial research-focused Internet as well as what it has become today. As the Internet grew it made sense to create locations where multiple networks could connect to each other over a common multi-access network.  The initial exchanges connected to NFSNet in the 1980s were located in San Francisco, New York, and Chicago, with an additional European exchange in Stockholm, Sweden. During these days the connectivity was between universities and other government research institutions, but that would soon change as commercial interest in the Internet grew.   

### Growth through Internet Privatization 
In the early 1990s those running the Internet saw the first commercial companies join. The ANS Internet backbone was built by a consortium of commercial companies (PSI, MCI, and Merit) and those companies wanted to offer commercial services such as Email across the infrastructure and do other entities already connected to the Internet. One of the first commercial exchanges created was simply called the "Commercial Internet Exchange" or CIX, located in Reston, Virginia.  

One issue encountered in the initial exchange which still happens today is disagreements in connectivity. The original ANS backbone network, the most widely used network for connecting to the Internet, refused to connect to the CIX. As the Internet transitioned to a commercial network as opposed to a research network, the role of ANS diminished as exchanges like CIX become more important.  

### Dawn of the NAP and Rapid IX Expansion  

In the mid 1990s as the Internet became more of a privatized public network, the need arose to create a number of public Internet peering exchange locations to help universities, enterprises, and service providers connect to each other.  The original NAPs in the United States were run by either regional or national telecommunications companies.  Figure 1 and the table below lists the original five US NAP locations. There were other exchange locations other than the NAPs, but these were the main peering points in the US, and at one point 90% of the worldwide Internet traffic flowed through these five locations.  

| NAP Name | Location  |Operator| 
| ---------------- | ---------------------- |-----|
| AADS             | Chicago, IL            | Ameritech |
| MAE-EAST         | Washington DC, DC             | MFN, Pre-Existing Consortium |  
| NYIX            | New York, NY           | Sprint | 
| *MAE-WEST         | San Jose, CA      | MCI| 
| PAIX             | Palo Alto, CA          | PacBell | 

**MAE-WEST was not one of the original four NAPs awarded by NFSNET but was already established as a west coast IX prior to 1993 when those NAPs were awarded.  

The United States was not the only location in the world seeing the formation of Internet exchanges. The Amsterdam Internet Exchange, AMS-IX was formed in 1994 and is still the largest Internet Exchange in Europe.  



## IX Design Evolution 
-------------------
As the Internet has evolved so has IX design, driven by bandwidth growth and the need for more flexible interconnection as the scope of traffic and who connects to the Internet evolves.  

### Initial Exchange Design
The initial Internet exchanges were built to be multi-access networks where a participant could use a single physical connection for both private point to point and public connections. These exchanges used either IP over FDDI or IP over ATM (over TDM) as the transport between peers, as well as switched Ethernet in some cases. FDDI and ATM allowed the use of virtual circuits to provide point to point and multipoint connections over a common fabric. Internet in FDDI quickly waned and some IXs like MAE-EAST created a second fabric using ATM due to its popularity in the late 1990s and early 2000s.  

### Ethernet Takes Over 
In the late 1990s Ethernet was becoming popular to build Local Area Networks due to its simplicity and cost. The use of VLANs to segment Ethernet traffic into a number of virtual networks also gave it the same flexibility of ATM and FDDI.  The original NAPs began to transition to Ethernet at this point. One hurdle however was Ethernet circuits for WAN connectivity had not become popular yet, so it took some time for Ethernet to overtake ATM and FDDI completely.  

After Ethernet took over as the main transport for IX fabrics, they were primarily built using simple L2 switch fabrics. These switch fabrics do not have the loop prevention of IP networks, so protocols like STP or MST must be used for loop prevention. Ethernet fabrics are still in widespread use with IX networks, especially those with smaller scale requirements. 

### More Advanced Fabric Transport 
As IX fabrics began to grow, there arose a need for better control of traffic paths and more capabilities than what simple L2 fabrics could offer. This is not an exhaustive list of potential transport options, but lists a few popular ones for IX use.   

#### VPLS and P2P PW over MPLS 
MPLS (Multi-Protocol Label Switching) has been a popular data plane shim layer for providing virtual private networks over a common fabric for more than a decade now. Distribution of labels is done using LDP or RSVP-TE. RSVP-TE offer resilience through the use of fast-reroute and the ability to engineer traffic paths based on constraints. In the design section we will examine the benefits of Segment Routing over both LDP and RSVP-TE.   

MPLS itself is not enough to interconnect IX participants, it requires using services to do so. VPLS (Virtual Private Lan Service) is the service most widely deployed today, emulating the data plane of a L2 switch, but carrying the traffic as MPLS-encapsulated frames. Point to point services are commonly provisioned using point to point pseudowires signaled using either a BGP or LDP control-plane. Please see (RFC XXX and RFC XXX).    

#### EVPN over VXLAN 
We will speak more about EVPN in the design section, but it has become the modern way to deliver L2VPN services, using a control-plane similar to L3VPN. EVPN is a control-plane protocol that can utilize different underlying transport methods. VXLAN is one method which encapsulates Layer2 frames into a VXLAN packet carried over IP/UDP. VXLAN is considered an overlay and since it is carried in IP has no inherent ability to provide resiliency or traffic engineering capabilities. Gaining that functionality requires layering VXLAN on top of MPLS transport, adding complexity to the overall network.   

#### TRILL, PBB, and other L2 Fabric Technology 
At a point in the early 2010s there was industry momentum towards creating a more advanced network without introducing the complexity of L3 routing into the network. Two protocols with more widespread support were TRILL and PBB (Provider Backbone Bridging) and its TE addition PBB-TE. Proprietary fabrics were also proposed like Cisco FabricPath and Juniper QFabric. Ultimately with the cost reduction of full L3 routing devices, these technologies faded and did not see widespread industry adoption. 

Modern IX Fabric Requirements 
-------------------------
## Hardware
The heart of an IX is the connectivity to participants at the edge and the transport connecting them to other participants. There are several components that needs to be considered when looking at hardware to build a modern IX.   

### Speeds and Feeds 
Bandwidth growth today requires the proper density to support the needs of the specific facility. This can range from 10s of Gbps to 10s of Tbps.   

### Edge and WAN Interface Flexibility
Ethernet is the single type of connectivity used today for connecting to an IX, but an IX does need flexibility to peers and the connectivity between IX fabric elements. 10G connections have largely replaced 1G connections for peers due to cost reduction in both device ports and optics. However, there is still a need for 1G connectivity as well as different physical medium types such as 10GBaseT. In order to support connectivity such as dark fiber, IXs sometimes must also provide ZR, ER, or DWDM based Ethernet port types.  

Modern IXs are also not typically limited ot a single physical location, so WAN connectivity also becomes important. Using technologies like coherent CFP2-DCO may be a requirement to interconnect IX facilities via high-speed flexible links.  

### Power and Space 
It almost goes without saying devices in a IX location must have the lowest possible space, power, and cooling footprint per BPS of traffic delivered. As interconnection continues to grow technology must advance to support higher density devices without considerably higher power and cooling requirements.  

## Packet Transport Requirements 
Resiliency is key within any fabric, and the IX fabric must be able to withstand the failure of a link or node with minimal traffic disruption. Boundaries between different IX facilities must be redundantly connected.  


## Service Types and Requirements  
In a modern IX there can be both traditional L2 connectivity as well as L3 connectivity. The following outlines the different service types and their requirements.  One common requirement across all service types is redundant attachment. Each service type should support either active/active or active/standby connectivity from the participant.   

### Point to Point Connectivity 
The most basic service type an IX fabric must provide is point to point participant connectivity. The edge must be able to accept and map traffic to a specific service based on physical port or VLAN tagged frames. The ability to rewrite VLANs at the edge may also be a requirement.   

### Multi-point Connectivity (Multi-lateral Peering)
Multi-point connectivity is most commonly used for multi-lateral peering fabrics or "public" fabrics where all participants belong to the same bridge domain. This type of fabric requires less configuration since a single IP interface is used to connect to multiple peers. BGP configuration can also be greatly simplified if the IX provides a route-server to advertise routes to participant peers versus a full mesh of peering sessions.   

### Layer 3 Cloud or Transit Connectivity    
Depending on the IX provider, they may offer their own blended transit services or multi-cloud connections via L3 connectivity. In this service type the IX will peer directly with the participant or provide a L3 gateway if the participant is not using dynamic routing to the IX.  

### QoS Requirements 
Quality of Service or Class of Service (CoS) covers an array of traffic handling components. With the use of higher speed 10G and 100G interfaces as defacto physical connectivity, the most basic QoS needed is policing or shaping of ingress traffic at the edge to the contracted rate of the participant. 

An IX can also offer differentiated services for connectivity between participants requiring traffic marking at the edge and specific treatment across the core of the network. 

### Broadcast, Multicast, and Unknown Unicast Traffic 

## Security 
### Peer Connection Isolation
One of the key tenets of a multi-tenant network fabric is to provide secure traffic isolation between parties using the fabric. This can be done using "soft" or "hard" mechanisms. Soft isolation uses packet structure in order to isolate traffic between tenants, such as MPLS headers or specific service tags. As you move down the stack, the isolation becomes "harder", first using VLAN tags, channels in the case of Flex Ethernet, or separate physical medium to completely isolate tenants. Harder isolation is typically less efficient and more difficult to operate. In modern networks, isolation using VPN services is regarded as sufficient for an IX fabric and offers the greatest flexibility and scale. The isolation must be performed on the IX fabric itself and protect against users spoofing MPLS headers or VLANs tags at the attachment point in the network.  

### L2 Security 
The table below lists the more common L2 security features required by an IX network. Some of these should perform the action of disabling either permanently or temporarily a connected port.   

| Feature | Description | 
| ---------------- | ---------------------- |
| L2 ACL     | Ability to create filters based on L2 frame criteria (SRC/DST MAC, Ethertype, control BPDUs, etc)  |
| ARP/ND/RA policing | Police ARP/ND/RA requests |  
| MAC scale limits  | Limit MAC scale for specific Bridge Domain|
| Static ARP | Override dynamic ARP with static ARP entries |

### L3 Security 

| Feature | Description | 
| ---------------- | ---------------------- |
|    | Ability to create filters based on L2 frame criteria (SRC/DST MAC, Ethertype, control BPDUs, etc)  |
| ARP/ND/RA policing | Police ARP/ND/RA requests |  
| MAC scale limits  | Limit MAC scale for specific Bridge Domain|
| Static ARP | Override dynamic ARP with static ARP entries |



## Additional IX Components 
### Route Servers
A redundant set of route servers is used in many IX deployments to eliminate each peer having to configure a BGP session to every other peer. The route server is similar to a BGP route reflector with the main difference being a route server operates with EBGP peers and not IBGP peers. The route server also acts as a point of route security since the filters governing advertisements between participants is typically performed on the route server. Route server definition can be found in RFC 7947 and route server operations in RFC 7948.    

### Route Looking Glass 
Looking glasses allow an outside user or internal participant to view the current real-time routing for a specific prefix or set of prefixes. This is invaluable for troubleshooting routing issues.   

### Analytics and Monitoring
#### Fabric Telemetry  
Having accurate statistics on peer and fabric state is important for evaluating the current health of the fabric as well assist in capacity planning. Monitoring traffic utilization, device health, and protocol state using modern telemetry such as streaming telemetry can help rectify faults faster and improve reliability.  See the appendix for a list of common Cisco and OpenConfig models to be used with streaming telemetry.   
#### Route Update History  
Route update history is one area of IX operation that can assist not only with IX growth but also Internet health as a whole. Being able to trace the history of route updates coming through an IX helps both providers and enterprises determine root cause for traffic issues, identify the origin of Internet security events, and assist those researching Internet routing change over time. Route update history can be communicated by either BGP or using BGP Monitoring Protocol (BMP).  

Modern IXP Fabric Network Design  
------------------------------------------
## Topology Considerations 
### Scale Out Design 
We can learn from modern datacenter design in how we build a modern IX fabric, at least the network located within a single facility or group of locations in close proximity. The use of smaller functional building blocks increases operational efficiency and resiliency within the fabric. Connecting devices in a Clos (leaf/spine or fat-tree are other names) fabric seen in Figure XX versus a large modular chassis approach has a number of benefits. In the case of interconnecting remote datacenters using a more fabric based approach connecting border devices in a more mesh design also increases overall resiliency.   

## Segment Routing Underlay 
### Segment Routing and Segment Routing Traffic Engineering
Segment Routing is the modern simplified packet transport control-plane for multi-service networks. Segment Routing eliminates separate IGP and label distribution protocols running in parallel while providing built-in resilience and traffic engineering capabilities. Much more information on segment routing can be located at http://www.segment-routing.net  

### SR-MPLS Data Plane
One important point with Segment Routing is that it is data plane agnostic, meaning the architecture is built to support multiple data plane types. The SR "SID" is an identifier expressed by the underlying data plane. The Segment Routing MPLS data plane uses standard MPLS headers to carry traffic end to end, with SR responsible for label distribution and path computation. In this design we will utilize the SR MPLS data plane, but as other data planes such as Segment Routing IPv6 become mature they could plugin in as well.   
### Multi-Plane Design using Flexible Algorithms 
An exciting develop in SR capabilities is through the use of "Flexible Algorithms."  Simply put Flex-Algo allows one to define multiple SIDs on a single device with each one representing a specific "algorithm."  Using the algorithm as a constraint in head-end path computation simplifies the path to a single label since the algorithm itself takes of pruning links not applicable to the topology.  See the figure below for an example of Flex-Algo using the initial topology definition to restrict the path to only encrypted links.  
### Constraint-based SR Policies  
Path computation on the head-end SR node or external PCE can include more advanced constraints such as latency, link affinity, hop count, or SRLG avoidance. Cisco supports a wide range of path constraints both within XR on the SR head-end node as well as through SR-PCE, Cisco's external Path Computation Element for Segment Routing.  
### On-Demand SR Policies 
Cisco has simplifies the control-plane even more with a feature called ODN (On Demand Next Hop) for SR-TE. When a head-end node receives an EVPN BGP route with a specific extended community, known as the color community, it instructs the head-end node to create an SR Policy to the BGP next-hop following the defined constraints for that community. The head-end node can compute the SR Policy path itself, or the ODN policy can instruct the head-end to consult a PCE for end-to-end path computation.   
## L2 IX Services using Segment Routing and EVPN
### EVPN Background 
EVPN is the next-generation service type for creating L2 VPN overlay services across a Segment Routing underlay network. EVPN replaces services like VPLS emulating a physical Ethernet switch with a scalable BGP based control-plane. MAC addresses are no longer learned across the network as part of traffic forwarded, they are learned at the edges and distributes as BGP VPN routes. Below is a list of just a few of the advantages of EVPN over legacy services types such as VPLS or LDP-signaled P2P pseudowires.  
- RFC 7432 is the initial RFC defining EVPN service types and operation covering both MP and P2P L2 services 
  - VPWS point to point Ethernet VPN  
  - ELAN multi-point Ethernet VPN 
- EVPN brings the paradigms of BGP L3VPN to Ethernet VPNs 
- MAC and ARP/ND (MAC+IP) information is advertised using BGP NLRI 
- EVPN signaling identifies the same ESI (Ethernet Segment) connected to multiple PE nodes, allowing active/active or active/standby multi-homing 
- EVPN has been extended to support IRB (Integrated Routing and Bridging) for inter-subnet routing 
### EVPN Benefits for IX Use 
- BGP-based control plane has obvious scaling and distribution benefits
- Elimates mesh of pseudowires between L2 endpoints 
- All-active per-flow multihoming 
- Reduced flooding scope for ARP traffic 
- BUM labels act as a way to control flooding without complex split-horizon configuration 
- Fast MAC withdrawal improves convergence vs. data plane learning 
- Filter ARP/MAC advertisements via common BGP route policy 
- Distributed MAC pinning 
   - Once MAC is learned on a CE interface, it is advertised with EVPN BD as “sticky” 
   - Remote PEs will drop traffic sourced from a MAC labeled as sticky from another PE 
   - Works in redundancy scenarios 
- Works seamlessly with existing L2 Ethernet fabrics without having to run L2 protocols such as STP within EVPN itself 

Modern IXP Deployment 
-----------------------------
## Background 
In the following section we will explore the deployment using IOS-XR devices and CLI. We will start with the most basic deployment and add additional components to enable features such as multi-plane design and L3 services.  
## Segment Routing Underlay Deployment 
In the simplest deployment example, Segment Routing is deployed by configuring either OSPF or IS-IS with SR MPLS extensions enabled. 

### SRGB and SRLB Definition 
It's recommended to first configure the Segment Routing Global Block (SRGB) across all nodes in a common SR IGP domain. This is done in IOS-XR with the following configuration: 

<pre>
segment-routing 
segment-routing
 global-block 16000 16999
 local-block 17000 17999
</pre>

### IGP / Segment Routing Configuration 

The following configuration example shows an example ISIS deployment with SR-MPLS extensiosn enabled for the IPv4 address family.  

<pre>router isis example 
 set-overload-bit on-startup wait-for-bgp
 is-type level-1
 net 49.0002.1921.6801.4003.00
 distribute link-state
 log adjacency changes
 log pdu drops
 lsp-refresh-interval 65000
 max-lsp-lifetime 65535
 lsp-password hmac-md5 encrypted 03276828295E731F70
 address-family ipv4 unicast
  metric-style wide
  mpls traffic-eng level-1-2
  mpls traffic-eng router-id Loopback0
  maximum-paths 32
  <b>segment-routing mpls</b>
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   metric 10
   <b>prefix-sid absolute 16431</b>
   !
  !
 !
 interface GigabitEthernet0/0/0/1
  circuit-type level-1
  point-to-point
  address-family ipv4 unicast
   fast-reroute per-prefix ti-lfa
  metric 10
</pre>

The two key elements to enable Segment Routing are `segment-routing mpls` under the ipv4 unicast address family and the node `prefix-sid absolute 16341` definition under the Loopback0 interface.  The prefix-sid can be defined as either an indexed value or absolute.  The index value is added to the SRGB start (16000 in our case) to derive the SID value. Using absolute SIDs is recommended where possible, but in a multi-vendor network where one vendor may not be able to use the same SRGB as the other, using an indexed value is necessary.   

### Enabling TI-LFA 
Topology-Independent Loop-Free Alternates is not enabled by default. The above configuration enables TI-LFA on the Gigabit0/0/0/1 interface for IPv4 prefixes. TI-LFA can be enabled for all interfaces by using this command under the address-family ipv4 unicast in the instance configuration. It's recommended to enable it at the interface to control other TI-LFA attributes such as node protection and SRLG support.   

This is all that is needed to enable Segment Routing, and you can already see the simplicity in its deployment vs additional label distribution protocols like LDP and RSVP-TE. 

## Basic EVPN Deployment 
We will first look at EVPN configuration for deploying basic point to point and multi-point L2 services.   

### BGP AFI/SAFI Configuration 
EVPN uses additional BGP address families in order to carry EVPN information across the network. EVPN uses the BGP L2VPN AFI of 25 and a SAFI of 70. In order to carry EVPN information between two peers, this AFI/SAFI must be enabled on all peers. The following shows the minimum BGP configuration to enable this at a global and peer level.  

<pre> 
router bgp 100
 bgp router-id 100.0.0.1
 address-family l2vpn evpn
 !
!
neighbor-group EVPN  
 remote-as 100
 update-source Loopback0
 address-family l2vpn evpn
 !
!
neighbor 100.0.0.2
 use neighbor-group EVPN 
</pre> 

At this point the two neighbors will become established over the EVPN AFI/SAFI.  The command to view the relationship in IOS-XR is `show bgp l2vpn evpn summary`  

### EVPN Service Configuration Elements
#### EVI - Ethernet Virtual Instance 
All EVPN services require the configuration of the Ethernet Virtual Instance identifier, used to advertise the existence of an EVPN service
endpoint to routers participating in the EVPN service network. 
#### RD - Route Distinguisher 
Similar to L3VPN and BGP-AD L2VPN, a Route Distinguisher is used to differentiate EVPN routes belonging to different EVIs. The RD is auto-generated based on the Loopack0 IP address as specified in the EVPN RFC.  
#### RT - Route Target 
Also similar to L3VPN and BGP-AD L2VPN, a Route Target extended community is defined so EVPN routes are imported into the correct EVI across the network. The RT is auto-generated based on the EVI ID, but can be manually configured. It is recommended to utilize the auto-generated RT value.   
#### ESI - Ethernet Segment Identifier 
The ESI is used to identify a particular Ethernet "segment" for the purpose of multi-homing. In single-homed scenarios, the ESI is set to 0 
by default. In multi-homing scenarios such all-active attachment, the ESI is configured the same on multiple routers. If it is known an attachment will never be multi-homed, using the default ESI of 0 is recommended, but if there is a chance it may be multi-homed in the future, using a unique ESI is recommended. The ESI is a 10-octet value with 1 byte used for the type and 9 octets for the value.  Values of 0 and the maximum value are reserved.   


### Topology Diagram for Services 
The following is a topology diagram to follow along with the service endpoints in the below service configuration examples.  

## EVPN Service Configuration Elements 



### P2P Peer Interconnect using EVPN-VPWS  
The following highlights a simple P2P transparent L2 interconnect using EVPN-VPWS 

## L3 Services using EVPN and L3VPN 

## Service and Transport Plane Coherence
Ideally the transport plane is service-aware, meaning the constraint and properties required by the service are expressed in the transport paths across the network.   




