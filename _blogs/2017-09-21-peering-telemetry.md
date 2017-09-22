---
published: true
date: '2017-09-21 11:00-0400'
title: Peering Telemetry 
excerpt: Peering Telemetry - Best Practices and Use Cases  
author: Phil Bedard
tags:
  - iosxr
  - Peering
  - Design
position: hidden 
---

{% include toc %}

Introduction
============

Telemetry is the process of measuring the state of the components in a system and transmitting it to a remote location for further processing and analysis. Telemetry is important in any system whether it be a car, water treatment facility, or IP network. The loosely coupled distributed nature of IP networks where the state of a single node can affect an entire set of interconnected elements makes telemetry especially important. Peering adds an additional dimension of a node not under your administrative control, requiring additional telemetry data and analysis to detect network anomalies and provide meaningful insight to network behavior. Analysis of peering telemetry data also enhances operations and network efficiency. In this paper, we’ll explore IP network telemetry data types, how they are used, and their configuration in IOS-XR.

Telemetry Data Types
====================

It’s important to make a distinction between the two major telemetry types we’ll be collecting and analyzing for peering networks. Each data types is important for operations and planning and often combined to fulfill complex use cases.

Metric Data
-----------

Metric data refers to the measurement of quantitative data which normally changes over time. Metric telemetry data is usually found in the form of a counter, Boolean value, gauge, rate, or histogram. A counter is a monotonically increasing measurement, interface received bits is an example of a counter. A Boolean value is used to periodically transmit a state value. These are typically used in the absence of an event data type covering the state change. Gauges are used to record instantaneous values in time, such as the number of prefixes received from a BGP peer. Rate data is the rate of change in a counter or gauge over a period of time. Rate data being received from a remote device requires some processing of data on the device itself since it must record historical values and average them over a time period. Interface bits per second is an example of a typical rate data type. Histograms are an additional more complex data type requiring the most processing on a device. Histograms store the frequency of occurrence over a period of time and typically use ranges to group similar values. Histograms are not widely used in IP networks, but may have more applicability in the future.

Event Data 
-----------

Event data is the recording of data at specific times triggered by a specific monitored state change. The state change could be a boolean value such as an interface being up or down, or an exception triggered by exceeding a limit on a metric like a BGP peer exceeding its received prefix limit. Event data always has a timestamp and then a series of other schema-less data points carrying additional information. Event data often contains metric data to supply context around the event. A prefix limit violation event may include a peer IP, peer ASN, and currently configured limit.

Peering Metric Data
===================

Peering Metric Data Protocols 
------------------------------

Metric data fields can be collected in two different ways on XR platforms. They can either be streamed from the device at regular time intervals (push), or retrieved from the device at regular intervals (pull). IOS-XR on the NCS5500 series supports MDT, or Model-Driven Telemetry over gRPC as an efficient method to stream statistics to a collector. The MDT fields are accessed by a specific telemetry sensor path defined in native IOS-XR, OpenConfig, or standard IETF YANG models. NETCONF, defined in RFCXXXX can also be used to retrieve metric data from a device over SSH or gRPC using the operational state paths associated with the same YANG models.

SNMP, the de facto protocol for pulling data from a device, is also supported via native IOS-XR or standard IETF MIBs.

Peer Physical and Logical Interface Statistics 
-----------------------------------------------

The most basic information needed on peering connections is interface statistics. Collected via SNMP or newer methods like Model-Driven Telemetry, having insight into both real-time traffic statistics and historical trends is a necessary component for operating and planning peering networks. A list of recommended interface counters, their related MDT sensor paths, and SNMP OIDs can be found in Appendix A.1.

BGP Operational State 
----------------------

There is a variety of BGP operational state data to be mined for information. Doing so can lead to enhanced peering operations. A wealth of information on global and per-peer BGP state is available via OpenConfig and native IOS-XR YANG models. This includes data such as per-AFI and per-neighbor prefix counts, update message counts, and associated configuration data. Using the OpenConfig BGP RIB modesl, you can retrieve the global best-path Loc-RIB and per-neighbor adj-RIB-in and adj-RIB-out pre and post policy along with the reason why a given route was not selected as best-path. This gives additional operational insight through automation which would normally require one to login to various routers, issue show commands, and parse the verbose output.

A list of recommended BGP OpState YANG Paths can be found in Appendix A.2

Sampled Netflow / IPFIX 
------------------------

Netflow was invented by Cisco due to requirements for traffic visibility and accounting. Netflow in its simplest form exports 5-tuple data for each flow traversing a Netflow-enabled interface. Netflow data is further enhanced with the inclusion of BGP information in the exported Netflow data, namely AS\_PATH and destination prefix. This inclusion makes it possible to see where traffic originated by ASN and derive the destination for the traffic per BGP prefix. The latest iteration of Cisco Netflow is Netflow v9, with the next-generation IETF standardized version called IPFIX (IP Flow Information Export). IPFIX has expanded on Netflow’s capabilities by introducing hundreds of entities.

Netflow is traditionally partially processed telemetry data. The device itself keeps a running cache table of flow entries and counters associated with packets, bytes, and flow duration. At certain time intervals or event triggered, the flow entries are exported to a collector for further processing. The type 315 extension to IPFIX, supported on the NCS5500, does not process flow data on the device, but sends the raw sampled packet header to an external collector for all processing. Due to the high bandwidth, PPS rate, and large number of simultaneous flows on Internet routers, Netflow samples packets at a pre-configured rate for processing. Typical sampling values on peering routers are 1:4000 or 1:8000 packets.

Peering Event Data 
===================

Peering event telemetry data is ideally sent when an event occurs on the device, as opposed to polling the state. The timestamped event is sent to a collection system which may simply log the event or the event may trigger the collection of additional data or remediation action.

Peering Event Data Protocols 
-----------------------------

### BGP Monitoring Protocol

BMP, defined in RFC7854, is a protocol to monitor BGP events as well as BGP related data and statistics. BMP has two primary modes, Route Monitoring mode and Route Mirroring mode. The monitoring mode will initially transmit the adj-rib-in contents per-peer to a monitoring station, and continue to send updates as they occur on the monitored device. Setting the L bits on the RM header to 1 will convey this is a post-policy route, 0 will indicate pre-policy. The mirroring mode simply reflects all received BGP messages to the monitoring host. IOS-XR supports sending pre and post policy routes to a station via the Route Monitoring mode. BMP can additionally send information on peer state change events, including why a peer went down in the case of a BGP event.

There are drafts in the IETF process to extend BMP to report additional routing data, such as the loc-RIB and per-peer adj-RIB-out. Local-RIB is the full device RIB including received BGP routes, routes from other protocols, and locally originated routes. Adj-RIB-out will add the ability to monitor routes advertised to peers pre and post policy.

### Syslog 

Syslog has long been used as a method for reporting event data from both host servers and network devices, and allows a severity to be transmitted along with a verbose log message. Syslog requires more complex post-processing on the receiver end since all the data is encoded within the text message itself, but in the absence of a standardized schema for certain events can be a useful option. Syslog is not typically encrypted which can be a security concern.

### SNMP Traps

SNMP traps are event-driven SNMP messages sent by a device following a well-defined SNMP OID schema. SNMP trap receivers can easily decode the message type by the OID and apply the appropriate policy. SNMP trap policies must be defined on the device itself to filter out unwanted messages.

Peering Related Events
----------------------

Monitoring all event data on a router can often overwhelm collectors, so prescriptive monitoring is needed to only ingest applicable events. Applicable SNMP trap OIDs can be found in Appendix A.3.

### Peer Interface State 

The most basic form of peer monitoring is physical and logical interface state. Whenever an interface goes down, it’s a traffic impacting event needing further investigation. Interface state can be polled at periodic intervals.

### Peer Session State 

Monitoring peer session state can be critical for detecting transient outages, traffic shifts, and performing root cause analysis on historical traffic impacting events. BGP peer sessions can transition from down to up in a short time period and are not always triggered by interface state changes.

### Max-Prefix Threshold Events

Setting a realistic max-prefix limit on peers is an important security mechanism. Most router operating systems support the ability to trigger an event based on a percentage threshold of this max prefix limit. This is an important event to monitor since reaching the limit generally results in a traffic-affecting session teardown event.

### Globla and Per-Peer RIB Changes

BMP allows one to monitor incoming advertisements on a per-peer basis and record them for historical purposes. Having a record of all changes allows one to playback updates to determine the past impact of peer advertisement changes. BMP is the preferred mechanism to stream BGP updates as they happen, but NETCONF can also be used to retrieve BGP RIB data globally and per-peer on IOS-XR.

Enabling Telemetry 
===================

Model Driven Streaming Telemetry 
---------------------------------

MDT is enabled on the node itself in three steps. 1) Grouping source data YANG paths, called sensors, into a sensor group. 2) Creating a destination group with the destination and data encoding method. 3) Creating a subscription grouping a sensor-group to a destination-group. This method of configuration is known as “dial-out” since the node itself initiates the streaming. Another method, called “dial-in” uses specific models to configure all of the above information from an external management application. The dial-in configuration is ephemeral, meaning it is not stored in the startup configuration. Configuration of MDT for IOS-XR on the NCS5500 can be found here: <https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/telemetry/b-telemetry-cg-ncs5500-62x/b-telemetry-cg-ncs5500-62x_chapter_011.html>

Also, <http://xrdocs.io> has a number of telemetry related blogs which go in depth on configuration and use cases for MDT.

On the collection side, Pipeline is a Cisco open-source project which can be used to collect streaming data and output it to several popular time-series databases. Pipeline can be located at <https://github.com/cisco/bigmuddy-network-telemetry-pipeline> and tutorial on using Pipeline at <https://xrdocs.github.io/telemetry/tutorials/2017-05-08-pipeline-with-grpc>

Netflow / IPFIX 
----------------

The Netflow and IPFIX configuration guide for the NCS5500 can be found here:

<https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/62x/b-ncs5500-netflow-configuration-guide-62x/b-ncs5500-netflow-configuration-guide-62x_chapter_010.html>

Tuning Netflow parameters is critical to extracting the most useful data from Netflow. It is recommended to contact your Cisco SE to work through optimizing Netflow for your platform. As a general guideline, for traffic and security analysis, a sampling interval of 8000:1 or 4000:1 is sufficient.

There are a wide range of Netflow collection engines on the market today as well as cloud-based solutions. PMACCT found at <http://www.pmacct.net> is a popular open-source Netflow and IPFIX collector.

BMP 
----

BMP is easily configured in the following steps in IOS-XR.

Configure a BMP destination host using the global “bmp server <1-8>” command with its associated parameters. The minimum configuration is “bmp server <1-8> host <fqdn|ip> port <port>.” BMP uses TCP as its transport protocol, and has no standard port so a port must be specified. Additionally, in order to send periodic BGP statistics, a statistics interval must be configured via the “bmp server <1-8> stats-reporting-period <1-3600> command”

Once a destination BMP host is configured, BMP is activated on a per-peer basis (or all peers via a shared peer-group configuration) using the “bmp-activate server <1-8>” under the neighbor configuration with the BGP routing configuration.

Collecting BMP data is best done using the open source SNAS collector, formally known as OpenBMP. SNAS can be found at <http://snas.io>.

SNMP and SNMP Traps 
--------------------

The NCS5500 IOS-XR SNMP configuration guide can be found here: <https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/sysman/62x/b-system-management-cg-ncs5500-62x/b-system-management-cg-ncs5500-62x_chapter_0110.html>

It is recommended to use SNMPv3 for higher security. SNMP is supported by most traditional EMS/NMS systems.

Syslog
------

The IOS-XR Syslog configuration guide can be found at <https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/system-monitoring/62x/b-system-monitoring-cg-ncs5500-62x/b-system-monitoring-cg-ncs5500-62x_chapter_010.html>

Applications for Peering Telemetry
==================================

Enhancing Peering Operations 
-----------------------------

Successfully operating a peering edge router requires knowledge of a variety of telemetry data. As the connecting device is not under your administrative control, having information on the state of the connection at all times is critical. Mitigating issues like an ingress congestion event on a peering interface can be more difficult to troubleshoot and be in a degraded state longer due to multiple party involvement.

### Monitor Peering Router Stability and Resources

Global BGP statistics can be used to determine peering router health. Global BGP state values such as overall per-AFI RIB size, established peer session count, overall BGP update counts, and state transitions are used to determine instability either causing network issues or leading to them. When coupled with known resource limits, these values can also be used to monitor devices for exhaustion in resources like FIB or RIB size, and peer session limits.

### Monitor Peer Stability

Just as we monitor the global BGP state, we can monitor the same statistics on a per-peer basis. Detecting instability on peering edge routers is beneficial since off-net instability can be replicated across your entire network. Tracking per-peer BGP state values such as overall BGP message count, update and withdrawal count, update queue depth, per-AFI adj-RIB-in pre/post policy, and active prefix count can help discover peer stability issues very quickly.

### 

### Using Flow Data for Enhanced Troubleshooting

It is often difficult to troubleshoot exact network behavior across administrative boundaries. Flow data can assist by giving a view of traffic behavior at an application stream level. Tools like ping and traceroute do not give the same insight into service traffic behavior.

### Applicable Metric Telemetry

-   Peer logical and physical interface statistics

-   Peering router general health data (CPU, Memory, FIB resources, etc.)

-   BGP protocol statistics

    -   Global BGP session count

    -   Global BGP RIB size

    -   Global BGP table version (update count)

    -   Per-peer session state

    -   Per-peer prefix counts

    -   Per-peer message update counts

    -   Per-peer message queue depth

    -   Current dampened paths

### Applicable Event Telemetry

-   Peer logical and physical interface state

-   Per-peer BGP session state

Network Visibility 
-------------------

### Peer Traffic Anomaly Detection 

Simple interface stats are again the starting point for peering network visibility. Having accurate and timely logical and physical interface stats can quickly alert you to service-affecting anomalies for both ingress and egress traffic. Using the faster sampling frequency of MDT on IOS-XR decreases the time to catch events from what was typically 5+ minutes using SNMP to 30 seconds or less.

Historical flow data from peers can be used to store a network baseline used with real-time information to determine anomalous behavior. Examples include DNS-driven content peers shifting traffic sources causing sub-optimal traffic on your network, or peer BGP routes being withdrawn from optimal peer sessions. Some shifts are not detected by interface statistics alone and require the flow-level traffic view to detect. Grouping by constraints such as SRC/DST ASN or BGP prefix can help quickly determine large changes in traffic across an entire network.

Analyzing BGP updates on a global and per-peer basis via BMP data can also help detect traffic-affecting routing anomalies and be an important resource for root cause analysis of previous events.

### Applicable Metric Telemetry

-   Peer logical and physical interface statistics

-   BGP protocol statistics

    -   Global BGP RIB size

    -   Global BGP table version (update count)

    -   Per-peer session state

    -   Per-peer prefix counts

    -   Per-peer message update counts

    -   Per-peer message queue depth

-   Netflow / IPFIX

### Applicable Event Telemetry

-   Peer logical and physical interface state

-   Per-peer BGP session state

Capacity Planning 
------------------

### Peer Targeting 

The most common use of Netflow data for peering is to determine who you need your network to peer with or where to add additional peer connections. The traffic exchange rate between sources or destinations on your network and remote networks can easily be derived with flow information and the associated BGP ASN data. Analyzing flow data from transit connections or larger service provider peer connections can help determine new organizations to peer with. Analyzing flow data from existing peers helps determine if traffic to/from your network is taking the optimal path. Sub-optimal paths can be remedied by adding additional peer connections.

Telemetry data can not only tell you where to augment existing peering, or connect to a new provider in an existing location, but help determine where to add additional peering facilities. Aggregated flow data along with topology data help determine the cost of carrying traffic across an internal network as well as paid peering connections. Eliminating network hops and augmenting paid peering or transit with settlement free peering connections can offset the location and network build costs in a short amount of time.

### Existing Peer Capacity Planning

Interface bandwidth statistics are necessary for accurate capacity planning. The most basic metric to trigger capacity upgrades is exceeding a traffic utilization threshold. Capacity planning can also be aided by flow data. Knowing the growth rates of specific types of peer traffic will aid in future overall network traffic projections. Flow data can also be used to derive growth rates between specific source/destination pairs, helping better predict growth across a network and not just at the peer interface boundary.

Security 
---------

Peering by definition is at the edge of the network, where security is mandatory. Telemetry data is critical to security applications and aids in quickly identifying potential threats and triggering mitigation activity.

### Attack Detection 

DDoS is a threat to all Internet-connected entities. Often times simple interface packet rates can be used to quickly identify that a DDoS attack is in progress. Flow data is then critical in determining the source, destination, volume, packet rate, and type of data associated to a DDoS attack. Coupled with a dynamic remediation system, traffic can be quickly blocked or diverted and alleviate congestion on downstream nodes.

Not all attacks have a signature of high packet rates. The attacks can also originate from within your own network, such as open DNS resolvers participating in an Internet-wide attack on a remote destination. Flow data becomes important in these instances, monitoring per-protocol or anomalous behavior on both ingress and egress peering traffic can quickly help identify and mitigate attacks.

### BGP Prefix Anomalies 

The pre-policy RIB information from BMP can be used for traffic simulation as well as detecting security issues such as prefix hijacking without the prefixes being active in the provider table. In certain cases, analyzing incoming NLRI for malformed attributes or excessive AS\_PATH lengths can help mitigate router security vulnerabilities as well.

Having historical information can also help troubleshoot traffic issues where a provider may be changing advertisement locations due to instability on their network, causing reachability issues from sources on your network.

### Applicable Metric Telemetry

-   Netflow / IPFIX

-   Peer logical and physical interface statistics

### Applicable Event Telemetry

-   Peer logical and physical interface state

-   BGP adj-RIB-in information and updates

Peering Traffic Engineering 
----------------------------

Peer traffic engineering in this context refers to shifting either ingress or egress traffic between peer connections. Peer TE may be performed on a variety of reasons such as capacity optimization, performance, or maintenance activity. Without more granular flow data to determine traffic per prefix, where a prefix may be recursively split for precision, accurate traffic placement cannot be achieved. Peer TE may be a manual operation, done by an offline planning tool, or a real-time network component.

### Ingress Peer Engineering 

Ingress peer engineering generally involves the manipulation of outbound BGP NLRI, either through prefix withdrawal, prefix disaggregation, or augmenting a transitive attribute such as MED or AS\_PATH. Targeting specific prefixes to manipulate requires telemetry data. IPE may be employed due to capacity, performance, or operations reasons. Accurate capacity augmentation requires interface statistics, BGP prefix information, and Netflow data to plan for traffic shifts and verify the network behaves as predicted after implementation.


Appendix A 
===========

A.1 Interface Metric SNMP OIDs and YANG Paths 
----------------------------------------------

### Yang Models 

ietf-interfaces
openconfig-interfaces
openconfig-if-ethernet
oc-platform
Cisco-IOS-XR-infra-statsd-oper
Cisco-IOS-XR-drivers-media-eth-oper

|                               |                                                                                |
|-------------------------------|--------------------------------------------------------------------------------|
| Logical Interface Admin State | Enum                                                                           |
| SNMP OID                      | IF-MIB:ifAdminStatus                                                           |
| OC YANG                       | oc-if:interfaces/interface/state/admin-status (see OC model, not just up/down) |
| Native YANG                   | Cisco-IOS-XR-pfi-im-cmd-oper:interfaces/interface-xr/interface/state           |
| MDT                           | Native                                                                         |

|                                     |                                                                               |
|-------------------------------------|-------------------------------------------------------------------------------|
| Logical Interface Operational State | Enum                                                                          |
| SNMP OID                            | IF-MIB:ifOperStatus                                                           |
| OC YANG                             | oc-if:interfaces/interface/state/oper-status (see OC model, not just up/down) |
| Native YANG                         | Cisco-IOS-XR-pfi-im-cmd-oper:interfaces/interface-xr/interface/state          |
| MDT                                 | Native                                                                        |

|                                     |                                                                                           |
|-------------------------------------|-------------------------------------------------------------------------------------------|
| Logical Last State Change (seconds) | Counter                                                                                   |
| SNMP OID                            | IF-MIB:ifLastChange                                                                       |
| OC YANG                             | oc-if:interfaces/interface/state/last-change                                              |
| Native YANG                         | Cisco-IOS-XR-pfi-im-cmd-oper:interfaces/interface-xr/interface/last-state-transition-time |
| MDT                                 | Native                                                                                    |

|                                |                                                              |
|--------------------------------|--------------------------------------------------------------|
| Logical Interface SNMP ifIndex | Integer                                                      |
| SNMP OID                       | IF-MIB:ifIndex                                               |
| OC YANG                        | oc-if:interfaces/interface/state/if-index                    |
| Native YANG                    | Cisco-IOS-XR-snmp-agent-oper:snmp/interface-indexes/if-index |
| MDT                            | Native                                                       |

|                                   |                                                                                                             |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------|
| Logical Interface RX Bytes 64-bit | Counter                                                                                                     |
| SNMP OID                          | IF-MIB:ifHCInOctets                                                                                         |
| OC YANG                           | oc-if:/interfaces/interface/state/counters/in-octets                                                        |
| Native YANG                       | Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters/bytes-received |
| MDT                               | Native                                                                                                      |

|                                   |                                                                                                         |
|-----------------------------------|---------------------------------------------------------------------------------------------------------|
| Logical Interface TX Bytes 64-bit | Counter                                                                                                 |
| SNMP OID                          | IF-MIB:ifHCOutOctets                                                                                    |
| OC YANG                           | oc-if:/interfaces/interface/state/counters/out-octets                                                   |
| Native YANG                       | Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters/bytes-sent |
| MDT                               | Native                                                                                                  |

|                             |                                                                                                           |
|-----------------------------|-----------------------------------------------------------------------------------------------------------|
| Logical Interface RX Errors | Counter                                                                                                   |
| SNMP OID                    | IF-MIB:ifInErrors                                                                                         |
| OC YANG                     | oc-if:/interfaces/interface/state/counters/in-errors                                                      |
| Native YANG                 | Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters/input-errors |
| MDT                         | Native                                                                                                    |
|                             |                                                                                                            |
|-----------------------------|------------------------------------------------------------------------------------------------------------|
| Logical Interface TX Errors | Counter                                                                                                    |
| SNMP OID                    | IF-MIB:ifOutErrors                                                                                         |
| OC YANG                     | oc-if:/interfaces/interface/state/counters/out-errors                                                      |
| Native YANG                 | Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters/output-errors |
| MDT                         | Native                                                                                                     |

|                                      |                                                                   |
|--------------------------------------|-------------------------------------------------------------------|
| Logical Interface Unicast Packets RX | Counter                                                           |
| SNMP OID                             | IF-MIB:ifHCInUcastPkts                                            |
| OC YANG                              | oc-if:/interfaces/interface/state/counters/in-unicast-pkts        |
| Native YANG                          | Not explicitly supported, subtract multicast/broadcast from total |
| MDT                                  | Native                                                            |

|                                      |                                                                   |
|--------------------------------------|-------------------------------------------------------------------|
| Logical Interface Unicast Packets TX | Counter                                                           |
| SNMP OID                             | IF-MIB:ifHCOutUcastPkts                                           |
| OC YANG                              | oc-if:/interfaces/interface/state/counters/out-unicast-pkts       |
| Native YANG                          | Not explicitly supported, subtract multicast/broadcast from total |
| MDT                                  | Native                                                            |

|                               |                                                                                                          |
|-------------------------------|----------------------------------------------------------------------------------------------------------|
| Logical Interface Input Drops | Counter                                                                                                  |
| SNMP OID                      | IF-MIB:ifIntDiscards                                                                                     |
| OC YANG                       | oc-if:/interfaces/interface/state/counters/in-discards                                                   |
| Native YANG                   | Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters/input-drops |
| MDT                           | Native                                                                                                   |

|                                |                                                                                                           |
|--------------------------------|-----------------------------------------------------------------------------------------------------------|
| Logical Interface Output Drops | Counter                                                                                                   |
| SNMP OID                       | IF-MIB:ifOutDiscards                                                                                      |
| OC YANG                        | oc-if:/interfaces/interface/state/counters/out-discards                                                   |
| Native YANG                    | Cisco-IOS-XR-infra-statsd-oper:infra-statistics/interfaces/interface/latest/generic-counters/output-drops |
| MDT                            | Native                                                                                                    |

|                                       |                                                                   |
|---------------------------------------|-------------------------------------------------------------------|
| Ethernet Layer Stats – All Interfaces | Counters                                                          |
| SNMP OID                              | NA                                                                |
| OC YANG                               | oc-if:interfaces/interface/oc-eth:ethernet/oc-eth:state           |
| Native YANG                           | Cisco-IOS-XR-drivers-media-eth-oper/ethernet-interface/statistics |
| MDT                                   | Native                                                            |

|                                     |                                                                                      |
|-------------------------------------|--------------------------------------------------------------------------------------|
| Ethernet PHY State – All Interfaces | Counters                                                                             |
| SNMP OID                            | NA                                                                                   |
| OC YANG                             | oc-platform:components/component/oc-transceiver:transceiver                          |
| Native YANG                         | Cisco-IOS-XR-drivers-media-eth-oper/ethernet-interface/interfaces/interface/phy-info |
| MDT                                 | Native                                                                               |

|                           |                                                                                                                   |
|---------------------------|-------------------------------------------------------------------------------------------------------------------|
| Ethernet Input CRC Errors | Counter                                                                                                           |
| SNMP OID                  | NA                                                                                                                |
| OC YANG                   | oc-if:interfaces/interface/oc-eth:ethernet/oc-eth:state/oc-eth:counters/oc-eth:in-crc-errors                      |
| Native YANG               | Cisco-IOS-XR-drivers-media-eth-oper/ethernet-interface/statistics/statistic/dropped-packets-with-crc-align-errors |
| MDT                       | Native                                                                                                            |

**The following transceiver paths retrieve the total power for the transceiver, there are specific per-lane power levels which can be retrieved from both native and OC models, please refer to the model YANG file for additional information.**

|                               |                                                                                                                                                                     |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Ethernet Transceiver RX Power | Counter                                                                                                                                                             |
| SNMP OID                      | NA                                                                                                                                                                  |
| OC YANG                       | oc-platform:components/component/oc-transceiver:transceiver/oc-transceiver:physical-channels/oc-transceiver:channel/oc-transceiver:state/oc-transceiver:input-power |
| Native YANG                   | Cisco-IOS-XR-drivers-media-eth-oper/ethernet-interface/interfaces/interface/phy-info/phy-details/transceiver-rx-power                                               |
| MDT                           | Native                                                                                                                                                              |

|                               |                                                                                                                                                                     |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Ethernet Transceiver TX Power | Counter                                                                                                                                                             |
| SNMP OID                      | NA                                                                                                                                                                  |
| OC YANG                       | oc-platform:components/component/oc-transceiver:transceiver/oc-transceiver:physical-channels/oc-transceiver:channel/oc-transceiver:state/oc-transceiver:input-power |
| Native YANG                   | Cisco-IOS-XR-drivers-media-eth-oper/ethernet-interface/interfaces/interface/phy-info/phy-details/transceiver-tx-power                                               |
| MDT                           | Native                                                                                                                                                              |

A.2 BGP Operational State YANG Paths
------------------------------------

### Relevant YANG Models

openconfig-bgp.yang
openconfig-bgp-rib.yang
Cisco-IOS-XR-ipv4-bgp-oper
Cisco-IOS-XR-ipv6-bgp-oper
Cisco-IOS-XR-ip-rib-ipv4-oper
Cisco-IOS-XR-ip-rib-ipv6-oper

### Global BGP Protocol State

IOS-XR native models do not store route information in the BGP Oper model, they are stored in the IPv4/IPv6 RIB models. These models contain RIB information based on protocol, with a numeric identifier for each protocol with the BGP ProtoID being 5. The protoid must be specified or the YANG path will return data for all configured routing protocols.

|                                |                                                                                                                              |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| BGP Total Paths (all AFI/SAFI) | Counter                                                                                                                      |
| SNMP OID                       | NA                                                                                                                           |
| OC YANG                        | oc-bgp:bgp/global/state/total-paths                                                                                          |
| Native YANG                    | Cisco-IOS-XR-ip-rib-ipv4-oper/rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count/num-active-paths |
| MDT                            | Native                                                                                                                       |

|                                   |                                                                                                                                 |
|-----------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| BGP Total Prefixes (all AFI/SAFI) | Counter                                                                                                                         |
| SNMP OID                          | NA                                                                                                                              |
| OC YANG                           | oc-bgp:bgp/global/state/total-prefixes                                                                                          |
| Native YANG                       | Cisco-IOS-XR-ip-rib-ipv4-oper/rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count/active-routes-count |
| MDT                               | Native                                                                                                                          |

### BGP Neighbor State

#### Example Usage

**Due the construction of the YANG model, the neighbor-address key must be included as a container in all OC BGP state RPCs. The following RPC gets the session state for all configured peers:**

```
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get>
    <filter>
      <bgp xmlns="http://openconfig.net/yang/bgp">
        <neighbors>
          <neighbor>
            <neighbor-address/>
            <state>
              <session-state/>
            </state>
          </neighbor>
        </neighbors>
      </bgp>
    </filter>
  </get>
</rpc>	

<nc:rpc-reply message-id="urn:uuid:24db986f-de34-4c97-9b2f-ac99ab2501e3" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <nc:data>
    <bgp xmlns="http://openconfig.net/yang/bgp">
      <neighbors>
        <neighbor>
          <neighbor-address>172.16.0.2</neighbor-address>
          <state>
            <session-state>IDLE</session-state>
          </state>
        </neighbor>
        <neighbor>
          <neighbor-address>192.168.2.51</neighbor-address>
          <state>
            <session-state>IDLE</session-state>
          </state>
        </neighbor>
      </neighbors>
    </bgp>
  </nc:data>
</nc:rpc-reply>
```



|                                      |                                                                                         |
|--------------------------------------|-----------------------------------------------------------------------------------------|
| Complete State for all BGP neighbors | Mixed                                                                                   |
| SNMP OID                             | NA                                                                                      |
| OC YANG                              | oc-bgp:bgp/neighbors/neighbor/state                                                     |
| Native YANG                          | Cisco-IOS-XR-ipv4-bgp-oper/bgp/instances/instance/instance-active/default-vrf/neighbors |
| MDT                                  | Native                                                                                  |

|                                      |                                                                                         |
|--------------------------------------|-----------------------------------------------------------------------------------------|
| Complete State for all BGP neighbors | Mixed                                                                                   |
| SNMP OID                             | NA                                                                                      |
| OC YANG                              | oc-bgp:bgp/neighbors/neighbor/state                                                     |
| Native YANG                          | Cisco-IOS-XR-ipv4-bgp-oper/bgp/instances/instance/instance-active/default-vrf/neighbors |
| MDT                                  | Native                                                                                  |

|                                     |                                                                                                                   |
|-------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| Session State for all BGP neighbors | Enum                                                                                                              |
| SNMP OID                            | NA                                                                                                                |
| OC YANG                             | oc-bgp:bgp/neighbors/neighbor/state/session-state                                                                 |
| Native YANG                         | Cisco-IOS-XR-ipv4-bgp-oper/bgp/instances/instance/instance-active/default-vrf/neighbors/neighbor/connection-state |
| MDT                                 | Native                                                                                                            |

|                                        |                                                                                                                     |
|----------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| Message counters for all BGP neighbors | Counter                                                                                                             |
| SNMP OID                               | NA                                                                                                                  |
| OC YANG                                | /oc-bgp:bgp/neighbors/neighbor/state/messages                                                                       |
| Native YANG                            | Cisco-IOS-XR-ipv4-bgp-oper/bgp/instances/instance/instance-active/default-vrf/neighbors/neighbor/message-statistics |
| MDT                                    | Native                                                                                                              |

|                                           |                                                                                                                    |
|-------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| Current queue depth for all BGP neighbors | Counter                                                                                                            |
| SNMP OID                                  | NA                                                                                                                 |
| OC YANG                                   | /oc-bgp:bgp/neighbors/neighbor/state/queues                                                                        |
| Native YANG                               | Cisco-IOS-XR-ipv4-bgp-oper/bgp/instances/instance/instance-active/default-vrf/sessions/session/messages-queued-out 
                                                                                                                      
  Cisco-IOS-XR-ipv4-bgp-oper/bgp/instances/instance/instance-active/default-vrf/sessions/session/messages-queued-in   |
| MDT                                       | Native                                                                                                             |

### BGP RIB Data

RIB data is retrieved per AFI/SAFI. To retrieve IPv6 unicast routes using OC models, replace “ipv4-unicast” with “ipv6-unicast”

IOS-XR native models do not have a BGP specific RIB, but a protocol of 'bgp' can be specified in the <protocol-name> field to filter out prefixes to those learned via BGP. 

#### Example Usage 

**The following OC YANG path retrieves a list of best-path IPv4 prefixes without attributes from the loc-RIB:**

```
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get>
    <filter>
      <bgp-rib xmlns="http://openconfig.net/yang/rib/bgp">
        <afi-safis>
          <afi-safi>
            <ipv4-unicast>
              <loc-rib>
                <routes>
                  <route>
                    <prefix/>
                    <best-path>true</best-path>
                  </route>
                </routes>
              </loc-rib>
            </ipv4-unicast>
          </afi-safi>
        </afi-safis>
      </bgp-rib>
    </filter>
  </get>
</rpc>   
```
 
The following native XR NETCONF RPC retrieves a list of BGP prefixes in its global (vrf default) IPv4 RIB, with only the prefix,prefix-length, source (route-path), and active attributes. Replacing &lt;active/&gt; with &lt;active&gt;true&lt;/active&gt; would only return active prefixes, removing the prefix,prefix-length-xr, and active leaf attributes under route will return all attributes for each route{: .notice--warning}

```
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get>
    <filter>
      <rib xmlns="http://cisco.com/ns/yang/Cisco-IOS-XR-ip-rib-ipv4-oper">
        <vrfs>
          <vrf>
            <afs>
              <af>
                <safs>
                  <saf>
                    <ip-rib-route-table-names>
                      <ip-rib-route-table-name>
                        <routes>
                          <route>
                            <route-path>
                                <ipv4-rib-edm-path>
                                    <address/>
                                </ipv4-rib-edm-path>
                            </route-path>
                            <prefix/>
                            <prefix-length-xr/>
                            <protocol-name>bgp</protocol-name>
                            <active/>
                          </route>
                        </routes>
                      </ip-rib-route-table-name>
                    </ip-rib-route-table-names>
                  </saf>
                </safs>
              </af>
            </afs>
            <vrf-name>default</vrf-name>
          </vrf>
        </vrfs>
      </rib>
    </filter>
  </get>
</rpc>
``` 

|                               |                                                                      |
|-------------------------------|----------------------------------------------------------------------|
| IPv4 RIB – Prefix Count | Counter                                                              |
| SNMP OID                      | NA                                                                     |
| OC YANG                       | oc-bgprib:bgp-rib/afi-safis/afi-safi/ipv4-unicast/loc-rib/num-routes |
| Native YANG                   | Cisco-IOS-XR-ip-rib-ipv4-oper/rib/vrfs/vrf/afs/af/safs/saf/ip-rib-route-table-names/ip-rib-route-table-name/routes/route| 
| MDT                           | Native                                                                     |

|                                               |                                                                               |
|-----------------------------------------------|-------------------------------------------------------------------------------|
| IPv4 RIB – IPv4 Prefixes w/o Attributes | List                                                                          |
| SNMP OID                                      | NA                                                                              |
| OC YANG                                       | oc-bgprib:bgp-rib/afi-safis/afi-safi/ipv4-unicast/loc-rib/routes/route/prefix |
| Native YANG                                   |Cisco-IOS-XR-ip-rib-ipv4-oper/rib/vrfs/vrf/afs/af/safs/saf/ip-rib-route-table-names/ip-rib-route-table-name/routes/route **(see example RPC)**|
| MDT                                           |Native                                                                               |

|                                             |                                                                  |
|---------------------------------------------|------------------------------------------------------------------|
| IPv4 Local RIB – IPv4 Prefixes w/Attributes | List                                                             |
| SNMP OID                                    | NA                                                                 |
| OC YANG                                     | oc-bgprib:bgp-rib/afi-safis/afi-safi/ipv4-unicast/loc-rib/routes |
| Native YANG                                 | Cisco-IOS-XR-ip-rib-ipv4-oper/rib/vrfs/vrf/afs/af/safs/saf/ip-rib-route-table-names/ip-rib-route-table-name/routes/route **(see example RPC)**|
| MDT                                         | Native                                                                 |

**The following per-neighbor RIB paths can be qualified with a specific neighbor address to retrieve RIB data for a specific peer. Below is an example of a NETCONF RPC to retrieve the number of post-policy routes from the 192.168.2.51 peer and the returned output.  Native IOS-XR models do not support per-neighbor RIBs, but using the above example with the route-path address leaf set to the neighbor address will filter prefixes to a specific neighbor.** 

```
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get>
    <filter>
      <bgp-rib xmlns="http://openconfig.net/yang/rib/bgp">
        <afi-safis>
          <afi-safi>
            <ipv4-unicast>
              <neighbors>
                <neighbor>
                  <neighbor-address>192.168.2.51</neighbor-address>
                  <adj-rib-in-post>
                    <num-routes/>
                  </adj-rib-in-post>
                </neighbor>
              </neighbors>
            </ipv4-unicast>
          </afi-safi>
        </afi-safis>
      </bgp-rib>
    </filter>
  </get>
</rpc>

<nc:rpc-reply message-id="urn:uuid:7d9a0468-4d8d-4008-972b-8e703241a8e9" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <nc:data>
    <bgp-rib xmlns="http://openconfig.net/yang/rib/bgp">
      <afi-safis>
        <afi-safi>
          <afi-safi-name xmlns:idx="http://openconfig.net/yang/rib/bgp-types">idx:IPV4_UNICAST</afi-safi-name>
          <ipv4-unicast>
            <neighbors>
              <neighbor>
                <neighbor-address>192.168.2.51</neighbor-address>
                <adj-rib-in-post>
                  <num-routes>3</num-routes>
                </adj-rib-in-post>
              </neighbor>
            </neighbors>
          </ipv4-unicast>
        </afi-safi>
      </afi-safis>
    </bgp-rib>
  </nc:data>
</nc:rpc-reply>
```


|                                     |                                                                                    |
|-------------------------------------|------------------------------------------------------------------------------------|
| IPv4 Neighbor adj-rib-in pre-policy | List                                                                               |
| SNMP OID                            | NA                                                                                 |
| OC YANG                             | oc-bgprib:bgp-rib/afi-safis/afi-safi/ipv4-unicast/neighbors/neighbor/adj-rib-in-re |
| MDT                                 | NA                                                                                   |

|                                      |                                                                                      |
|--------------------------------------|--------------------------------------------------------------------------------------|
| IPv4 Neighbor adj-rib-in post-policy | List                                                                                 |
| SNMP OID                             | NA                                                                                   |
| OC YANG                              | oc-bgprib:bgp-rib/afi-safis/afi-safi/ipv4-unicast/neighbors/neighbor/adj-rib-in-post |
| MDT                                  | NA                                                                                     |

|                                      |                                                                                      |
|--------------------------------------|--------------------------------------------------------------------------------------|
| IPv4 Neighbor adj-rib-out pre-policy | List                                                                                 |
| SNMP OID                             | NA                                                                                   |
| OC YANG                              | oc-bgprib:bgp-rib/afi-safis/afi-safi/ipv4-unicast/neighbors/neighbor/adj-rib-out-pre |
| MDT                                  | NA                                                                                     |

|                                       |                                                                                      |
|---------------------------------------|--------------------------------------------------------------------------------------|
| IPv4 Neighbor adj-rib-out post-policy | List                                                                                 |
| SNMP OID                              | NA                                                                                     |
| OC YANG                               | oc-bgprib:bgp-rib/afi-safis/afi-safi/ipv4-unicast/neighbors/neighbor/adj-rib-out-pre |
| MDT                                   | NA                                                                                   |

A.3 Device Resource YANG Paths
------------------------------

Cisco-IOS-XR-fretta-bcm-dpa-hw-resources-oper.yang
openconfig-platform

|                  |                         |
|------------------|-------------------------|
| Device Inventory | List                    |
| SNMP OID         |                         |
| OC YANG          |oc-platform:components |
| MDT              |                         |

|                             |                                                                                                         |
|-----------------------------|---------------------------------------------------------------------------------------------------------|
| NCS5500 Dataplane Resources | List                                                                                                    |
| SNMP OID                    | NA                                                                                                      |
| OC YANG                     | NA                                                                                                      |
| Native YANG                 | Cisco-IOS-XR-fretta-bcm-dpa-hw-resources-oper/dpa/stats/nodes/node/hw-resources-datas/hw-resources-data |
| MDT                         |                                                                                                         |


