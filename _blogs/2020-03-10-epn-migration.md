# Overview  
As networks modernize, it's rarely done via a Greenfield network build. Greenfield networks are built from scratch, without the burdens of legacy network equipment,legacy network protocols, and existing services. It is much more common for modernization to happen in Brownfield networks.  Brownfield network builds are done by adding or replacing to an existing network, requiring both networks to be functional for some time period until full technology migration has occurred. In some networks, this may not happen for many years, if at all. It is for this reason we need to prescribe migration strategies minimizing network impact without significant operational difficulty. 

# Migration Options  
There are two possible migration techniques, each driven by specific network service requirements. Migrating networks may use one of these techniques or both of them.   

## Coexistence 
Coexistence is the ability for the transport and service control and forwarding planes to operate as "ships in the night." This popular term means each method works without being aware of the other, like ships passing each other in the night having no awareness of each other. In order for two PE devices to use the new transport it requires the end to end path support the new technology. Coexistence is preferable if possible due to  

## Interoperability  
Interoperability (interop) is the ability to use a different transport or service type on different routers, with some router in the network performing a translation function between technologies. At the MPLS transport layer this requires inspecting a label from one distribution protocol, and re-advertising it with another protocol when necessary, along with the logic to build a forwarding plane stitching together labels from different source protocols. Service interop is performed by having a parent service type which can translate service information between each service type.   

# Common Existing Network Deployments  
Through the Evolved Programmable Network validated designs, Cisco has assisted providers in building large scale networks supporting a variety of network services. This guide will focus on EPN 4.0/5.0, which utilize similar underlying MPLS transport technology and the same VPN service types. 

## EPN 4.0 Unified MPLS Transport 
Unified MPLS, or seamless MPLS, has been deployed by providers around the world to build networks supporting mobile, business, and residential services. Unified MPLS uses isolated IGP domains for specific areas of the network. Each IGP domain contains a relatively small number of routers, driven by IGP scaling issues especially on lower end access routers. This brings about a need for an additional MPLS overlay layer to provide inter-domain MPLS transport between nodes in different IGP domains. In a unified MPLS design each PE node originating a service advertised its Loopback address (main service termination IP) via BGP labeled unicast to create this overlay. BGP labeled unicast is defined in RFC3107 and uses specific BGP SAFI to attach an MPLS label to an associated prefix. Unified MPLS relies on recursive routing to scale networks to tens of thousands of nodes, with each ABR node between boundtargaries changing the BGP next-hop towards PEs in each IGP domain to its own Loopback when advertising inter-domain PE prefixes. Reachability between access PEs in each domain is achieved using a MPLS label stack with two labels. The original remote access PE label, and then the transport label to its local ABR. The ABR will have a BGP route with the next-hop of the next ABR in the path, and will swap the top label.   

### Intra-domain Label Distribution 
There must be an intra-domain MPLS LSP to/from an access PE to its local ABR as well as an LSP between ABR nodes across the core. In a traditional MPLS network either LDP or RSVP-TE is used for intra-domain connectivity. LDP and RSVP-TE both operate in a manner where the intra-domain transport label changes at each hop in the domain. LDP and RSVP-TE add an additional protocol across the network. In this document we will focus on LDP interop and RSVP-TE coexistence. As of this writing interop between Segment Routing and RSVP-TE is not supported.  

## EPN 4.0 Services 
The two service types we'll focus on are L3VPN and L2VPN. L3VPN is relatively unchanged from EPN 4.0 to CST. L2VPN services have changed however so we will focus on those. L2VPN pertains to two different service types, point to point and multi-point. Point-to-point is implemented using pseudowires, multi-point is traditionally implemented using VPLS. Virtual Private Lan Service emulates a L2 switch by performing data plane MAC learning along with other L2 functions. Signaling for pseudowires and VPLS can be done using targeted LDP (T-LDP) or BGP. BGP can either be auto-discovery, where endpoints are discovered using T-LDP pseudowires, or use BGP for both auto-discovery as well as MPLS label allocation. 

85K mac addresses 
EVPN MAC entries 
Access house is all L2 
4 MAC addresses per household 
85000 in the aggregation domain today 
IPv6 neighbor entries 
NCS 560 
