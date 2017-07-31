---
published: true
date: '2016-07-27 13:58 -0700'
title: iPXE Deep Dive
author: Patrick Warichet
excerpt: iPXE Deep Dive
tags:
  - iosxr
  - cisco
  - iPXE
---
{% include toc icon="table" title="IOS-XR: iPXE Deep Dive" %}

## Introduction

iPXE is an open source boot firmware (licensed under the GNU GPL with some portions under GPL-compatible licenses). It is fully backward compatible with PXE but include several enhancement. The enhancement that are important for IOS-XR are the following:

- Boot from a web server via HTTP
- control the boot process with scripts
- control the boot process with menus
- DNS support

iPXE is included in the network card of the management interfaces only and support for iPXE boot is included in the system firmware (UEFI) of the NCS1K, NCS5k and NCS5500 series routers. All these systems are equipped with a UEFI 64-bits firmware (aka BIOS).

iPXE can run on both IPv4 and IPv6 protocol but cannot use SLAAC for IPv6.

iPXE enumerates all Ethernet interfaces net0, net1, net2, ...

### Topology

In the following examples we will use a NCS-5001 router.  This device is equipped with 2 management interfaces but we will use only one of these two interfaces, It is recommended to place each interfaces in different subnet to facilitate the management process and improve redundancy.

The following diagram show the topology used for all examples. Both the DHCP and HTTP server are on a different subnet than the NCS-5001.
![topology.png]({{site.baseurl}}/images/topology.png)

## Boot Process

The IOS-XR 6.0 boot process is illustrated below, iPXE requires two external services, a DHCP server (e.g. isc-dhcpd) and a HTTP server (e.g. Apache)

It is important to note that a different dhcp client will start at the end of the boot process. This second dhcp client will facilitate auto provisioning the system.
![boot-process.png]({{site.baseurl}}/images/boot-process.png)

By default all NCS series router boot from the local disk, there are 2 options to force the system to boot using iPXE: If the device is already booted you can issue the command "hw-module location <location> bootmedia network reload" in admin mode to force the system to reboot in iPXE mode.

```
RP/0/RP0/CPU0:ios#admin
Sun Apr 10 00:56:08.037 UTC
root connected from 127.0.0.1 using console on xr-vm_node0_RP0_CPU0
sysadmin-vm:0_RP0# hw-module location all bootmedia network reload
Sun Apr  10 00:56:24.167 UTC
Reload hardware module ? [no,yes] yes
result Card reload request on all succeeded.
```

If the system is just being powered on, you can get to the device firmware by pressing `<ESC>` or `<DEL>` after it has completed the hardware diagnostic. You will be presented with the Boot selection menu. To force the device to boot using iPXE select the first entry "UEFI: Built-in EFI IPXE"
![bios.png]({{site.baseurl}}/images/bios.png)

Once the option is selected, iPXE will initialize the management interfaces, display the features options that were included in the iPXE firmware and propose you to jump into the iPXE prompt by pressing `<CTRL>B`

```
iPXE initialising devices...
ok
iPXE 1.0.0+ (aa070) -- Open Source Network Boot Firmware -- http://ipxe.org
Features: DNS HTTP TFTP VLAN EFI ISO9660 NBI Menu
Press Ctrl-B to drop to iPXE shell
iPXE>
```

When presented with the iPXE command line, you are in the iPXE environment, there are multiple commands that can be used for manual booting and for diagnosing problems. Commands can also be used as part of an iPXE script (see section iPXE with chain loading).

You can use the help command to show a list of all available commands. Full documentation for each command is provided in the iPXE command reference [http://ipxe.org/cmd](http://ipxe.org/cmd).

## iPXE DHCPv4 Request

After Initializing the management Interface Ethernet driver, iPXE will send a DHCP request, DHCP will send both an IPv6 and an IPv4 request, The capture below show the initial IPv4 DHCP request sent by the system.

```
    Bootstrap Protocol
        Message type: Boot Request (1)
        Hardware type: Ethernet (0x01)
        Hardware address length: 6
        Hops: 0
        Transaction ID: 0x20e69f64
        Seconds elapsed: 4
        Bootp flags: 0x8000 (Broadcast)
            1... .... .... .... = Broadcast flag: Broadcast
            .000 0000 0000 0000 = Reserved flags: 0x0000
        Client IP address: 0.0.0.0 (0.0.0.0)
        Your (client) IP address: 0.0.0.0 (0.0.0.0)
        Next server IP address: 0.0.0.0 (0.0.0.0)
        Relay agent IP address: 0.0.0.0 (0.0.0.0)
        Client MAC address: 52:46:27:70:1a:67 (52:46:27:70:1a:67)
        Client hardware address padding: 00000000000000000000
        Server host name not given
        Boot file name not given
        Magic cookie: DHCP
        Option: (53) DHCP Message Type
            Length: 1
            DHCP: Discover (1)
        Option: (57) Maximum DHCP Message Size
            Length: 2
            Maximum DHCP Message Size: 1472
        Option: (93) Client System Architecture
            Length: 2
            Client System Architecture: EFI x86-64 (9)
        Option: (94) Client Network Device Interface
            Length: 3
            Major Version: 3
            Minor Version: 10
        Option: (60) Vendor class identifier
            Length: 45
            Vendor class identifier: PXEClient:Arch:00009:UNDI:003010:PID:NCS-5001
        Option: (77) User Class Information
            Length: 4
            User Class identifier: iPXE
        Option: (55) Parameter Request List
            Length: 22
            Parameter Request List Item: (1) Subnet Mask
            Parameter Request List Item: (3) Router
            Parameter Request List Item: (6) Domain Name Server
            Parameter Request List Item: (7) Log Server
            Parameter Request List Item: (12) Host Name
            Parameter Request List Item: (15) Domain Name
            Parameter Request List Item: (17) Root Path
            Parameter Request List Item: (43) Vendor-Specific Information
            Parameter Request List Item: (60) Vendor class identifier
            Parameter Request List Item: (66) TFTP Server Name
            Parameter Request List Item: (67) Bootfile name
            Parameter Request List Item: (119) Domain Search
            Parameter Request List Item: (128) PXE - undefined (vendor specific)
            Parameter Request List Item: (129) PXE - undefined (vendor specific)
            Parameter Request List Item: (130) PXE - undefined (vendor specific)
            Parameter Request List Item: (131) PXE - undefined (vendor specific)
            Parameter Request List Item: (132) PXE - undefined (vendor specific)
            Parameter Request List Item: (133) PXE - undefined (vendor specific)
            Parameter Request List Item: (134) PXE - undefined (vendor specific)
            Parameter Request List Item: (135) PXE - undefined (vendor specific)
            Parameter Request List Item: (175) Etherboot
            Parameter Request List Item: (203) Unassigned
        Option: (175) Etherboot
            Length: 36
    Value:2969895296,2248423659,50397184,385941796,16847617,19529985,654377248,
    16848129,19267841
        Option: (61) Client identifier
            Length: 11
            Hardware type: DUID (0xFF)
            Client Identifier: 46:4f:43:31:39:34:37:52:31:34:33 (FOC1947R143)
        Option: (97) UUID/GUID-based Client Identifier
            Length: 17
            Client Identifier (UUID): 9ed138b2-dc55-42b6-9c56-2cf6f63921d9
        Option: (255) End
            Option End: 255
```

iPXE includes a number of options in the initial IPv4 DHCP request, the relevant ones are highlighted

**Option 60:** "vendor-class-identifier" Identify 4 elements separated by columns:
1 The type of client: e.g.: PXEClient
2 The architecture of The system (Arch): e.g.: 00009 Identify an EFI system using a x86-64 CPU
3 The Universal Network Driver Interface (UNDI): e.g.: 003010 (first 3 octets identify the major version and last 3 octets identify the minor version)
4 The Product Identifier (PID): e.g.: NCS-5001

**Option 61:** "dhcp-client-identifier" Identify the Serial Number of the system

**Option 66 and 67:** are used for TFTP, the first one request the TFTP server name while the second request the filename

**Option 77:** "user-class" Identify the mode of the system: e.g.: iPXE

**Option 97:** "uuid" Identify the Universally Unique Identifier a 128-bit value (not usable at this time)

**Option 128 - 135:** Reserved for PXE boot variables but not in use.

In is response the DHCP server will place the bootfile URI in option 67 "filename" or "bootfile-name" e.g.:[http://172.30.0.22/ncs5k/6.0.0/ncs5k-mini-x.iso-6.0.0](http://172.30.0.22/ncs5k/6.0.0/ncs5k-mini-x.iso-6.0.0)

## iPXE DHCPv6 Request

```
    DHCPv6
        Message type: Relay-forw (12)
        Hopcount: 0
        Link address: fd:30:12::1 (fd:30:12::1)
        Peer address: fe80::c672:95ff:fea7:efc0 (fe80::c672:95ff:fea7:efc0)
        Relay Message
            Option: Relay Message (9)
            Length: 88
            Value: 011ac36d00010012000200000009464f4331393437523134...
            DHCPv6
                Message type: Solicit (1)
                Transaction ID: 0x1ac36d
                Client Identifier
                    Option: Client Identifier (1)
                    Length: 18
                    Value: 000200000009464f43313934375231343300
                    DUID: 000200000009464f43313934375231343300
                    DUID Type: assigned by vendor based on Enterprise number (2)
                    Enterprise ID: ciscoSystems (9)
                    Identifier: 464f43313934375231343300 (FOC1947R143)
                Identity Association for Non-temporary Address
                    Option: Identity Association for Non-temporary Address (3)
                    Length: 12
                    Value: 1d4098ed0000000000000000
                    IAID: 1d4098ed
                    T1: 0
                    T2: 0
                Option Request
                    Option: Option Request (6)
                    Length: 8
                    Value: 00170018003b003c
                    Requested Option code: DNS recursive name server (23)
                    Requested Option code: Domain Search List (24)
                    Requested Option code: Bootfile URL(59)
                    Requested Option code: Bootfile Prameters (60)
                User Class
                    Option: User Class (15)
                    Length: 6
                    Value: 000469505845 : "iPXE"
                Vendor Class
                    Option: Vendor Class (16)
                    Length: 14
                    Value: 0000000900084e43532d35303031
                    Enterprise ID: ciscoSystems (9)
                    vendor-class-data: NCS-5001
                Elapsed time
                    Option: Elapsed time (8)
                    Length: 2
                    Value: 0000
                    Elapsed-time: 0 ms
        Interface-Id
            Option: Interface-Id (18)
            Length: 4
            Value: 0000001d
            Interface-ID: 
```

The initial DHCPv6 solicit has the relevant option highlighted

**Option 1:** "client-identifier" equivalent to DHCPv4 option 61 but with the following format:
DUID Type: integer 16 e.g.: 0002 (assigned by vendor)
Enterprise Id: integer 32 e.g.: 00000009 (Cisco Systems)
Client Identifier: string e.g.: FOC1947R143

**Option 15:** "dhcp6.user-class" equivalent to DHCPv4 option 77 but the first 2 Octets define the length of the string

**Option 16:** "vendor-class-identifier" equivalent to DHCPv4 option 60 but with the following format:
Enterprise Id: integer 32 e.g.:00000009 (Cisco Systems)
Length: integer 16
Vendor: string e.g.: NCS-5001

**Option 59:** "dhcp6.bootfile-url" equivalent to DHCPv4 option 67

**Option 60:** "dhcp6.bootfile-parameter"  required to be present but not in use.

The DHCPv6 server will include option 59 "dhcp6.bootfile-url" in its response containing the full URL of the boot image e.g.: [http://[fd:30::172:30:0:22]/ncs5k/6.0.0/ncs5k-mini-x.iso-6.0.0] (http://[fd:30::172:30:0:22]/ncs5k/6.0.0/ncs5k-mini-x.iso-6.0.0)

## iPXE without Chainloading

In this first examples, iPXE features are not used but the usage is similar to PXE boot. In the following examples we will rely solely on the DHCP server configuration to provide the elements necessary to identify the boot ISO for the device.

## DHCP Server Configuration

### DHCPv4

Using the options above we can configure isc-dhcpd to adequately provide the URI to boot the system, the common statements for the network and the pool are shown below:

```
######### Network 172.30.12.0/24 ################
shared-network 172-30-12-0 {
   subnet 172.30.12.0 netmask 255.255.255.0 {
      option subnet-mask 255.255.255.0;
      option broadcast-address 172.30.12.255;
      option routers 172.30.12.1;
      option domain-name-servers 172.30.0.25;
      option domain-name "cisco.local";
   }
   ####### Pool #########
        pool {
           range 172.30.12.10 172.30.12.100;
           next-server 172.30.0.22;
           if exists user-class and option user-class = "iPXE" {
              filename = "http://172.30.0.22/ncs5k-mini-4";
           } else if exists user-class and option user-class = "exr-config" {
              filename = "http://172.30.0.22/scripts/ncs-ztp.sh";
           }
```

In the example above option 77 is used to provide the bootfile to the system, the if-then-else statement is required to prevent the DHCP server to provide the (large) bootfile to the auto-configuration process. With this configuration all system in iPXE mode will receive a DHCP offer with identical bootfile URI.

If we want to add more granularity to the process we can define a class and using option 60 to only target a specific product or a family of products using the PID embedded in the the request. in the match statement we first verify that the system is in iPXE mode by matching the beginning of the vendor-class-identifier "PXEClient" than we match the first 6 octets of the PID portion "NCS-50", this will match all the NCS-5K routers (NCS-5001, NCS-5002, NCS-5011) and provide them the same bootfile URI, the "if" statement can be more specific to only match NCS-5001 product and additional "else-if" statement can be added to match other products.

```
######### Class #########
   class "ncs-5k" {
      match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
         if substring (option vendor-class-identifier, 37, 6) = "NCS-50" {
            filename = "http://172.30.0.22/ncs5k-mini-3";
         }
      }
```

Granularity of the boot image can be even more specific, the traditional approach is to use the mac address with a host definition inside the pool, as illustrated below

```
######## Hosts #########
host ncs-5001-a {
   hardware ethernet c4:72:95:a7:ef:c2;
   if exists user-class and option user-class = "iPXE" {
      filename = "http://172.30.0.22/ncs5k-mini-1";
   }
   fixed-address 172.30.12.50;
}
```

Using the host statement we provide a fixed address which can be useful for DNS, we still need to verify that option 77 is set to iPXE in the request to only provide the bootfile when required. The disadvantage of using the mac-address is that it is not necessary know in advance and is not written on the packaging box if this is the initial bootup of the system, another approach would be to use the uuid (option 97) or the serial number embedded in option 61.

```
######## Hosts #########
host ncs-5001-b {
   option dhcp-client-identifier "FOC1947R144";
   if exists user-class and option user-class = "iPXE" {
      filename = "http://172.30.0.22/ncs5k-mini-2";
   }
   fixed-address 172.30.12.52;
}
```

Using the different options and the flexibility of the ISC dhcp server we can achieve various degrees of granularity for the system we want to iPXE boot, but using DHCP options does not scale and each change require to restart the DHCP service. It offers the advantage to be identical to PXE and is easy to do for small to medium size network.

### DHCPv6

The ISC-DHCP service is mono stack, to support IPv6 a second instance of the service needs to be launched, both instances should use different configuration file. The common configuration statement for the DHCPv6 is as follow:

```
shared-network FD-30-12 {
   subnet6 fd:30:12::/64 {
      # Range for clients
      range6 fd:30:12::1024 fd:30:12::1124;
      # Range for clients requesting a temporary address
      range6 fd:30:12::/64 temporary;
      # Additional options
      option dhcp6.name-servers fd:30::172:30:0:25;
      option dhcp6.domain-search "cisco.local"; 
      if exists dhcp6.user-class and substring(option dhcp6.user-class, 2, 4) = "iPXE" {
         option dhcp6.bootfile-url = "http://[fd:30::172:30:0:22]/ncs5k-mini-4";
      } else if exists dhcp6.user-class and substring(option dhcp6.user-class, 0, 10) = "exr-config" {
         option dhcp6.bootfile-url = "http://[fd:30::172:30:0:22]/scripts/ncs-ztp.sh";
      }
   }
}
```

The DHCP configuration for IPv6 is similar to IPv4, the first 2 octets of the user-class define the length of the string, so we need to use the substring() statement to match "iPXE". Another difference is the square brackets used to represent the IPv6 address in isc-dhcp configuration file. iPXE cannot used SLAAC and you need to disable SLAAC on the first hop router and force statefull IPv6 address assignment on the segment. For reference here is a snippet of an Cisco IOS configuration. since the DHCP and HTTP server are on a different subnet a helper-address and a relay address have been configured for DHCPv4 and DHCPv6.

```
interface GigabitEthernet2/0
   description ** Management Network **
   ip dhcp relay information trusted
   ip address 172.30.12.1 255.255.255.0
   ip helper-address 172.30.0.25
   ip virtual-reassembly in
   ipv6 address FD:30:12::1/64
   ipv6 nd managed-config-flag
   ipv6 nd other-config-flag
   ipv6 nd router-preference High
   ipv6 dhcp relay destination FD:30::172:30:0:25 GigabitEthernet1/0
end
```

If there are no router present on the segment, you will have to launch the Router Advertisement Daemon (radvd) and force IPv6 routing on the DHCP server. An example of radvd.conf is as follow:

```
interface eth1
{
   MinRtrAdvInterval 5;
   MaxRtrAdvInterval 60;
   AdvSendAdvert on;
   AdvOtherConfigFlag on;
   IgnoreIfMissing off;
   prefix FD:30:12::/64 {
   };
};
```

Granularity in identifying the boot image is similar to IPv4, A class that encompass all NCS-5K series can be defined as follow (ipv6 class support is available starting isc-dhcp-server 4.3.4):

```
######### Class #########
class "ncs-5k" {
   match if exists vendor-class-identifier and substring(vendor-class-identifier, 6, 6) = "NCS-50";
   if exists dhcp6.user-class and substring(option dhcp6.user-class, 2, 4) = "iPXE" {
      filename = "http://[fd:30::172.30.0.22]/ncs5k-mini-3";
   }
}
```

Granularity to the host level can be achieve by using the serial number as identifier since the client sends the serial number as part of the client-id a simple solution is to match the complete hex data of the option.

```
######## Hosts #########
host ncs-5001-b {
   host-identifier option dhcp6.client-id 00:02:00:00:00:09:46:4f:43:31:39:34:37:52:31:34:33:00;
   if exists dhcp6.user-class and substring(option dhcp6.user-class, 2, 4) = "iPXE" {
      option dhcp6.bootfile-url = "http://[fd:30::172:30:0:22]/ncs5k-mini-2";
   }
   fixed-address6 fd:30:12::172.30.12.52;
}
```

Refer to the section iPXE DHCPv6 Request on how to decode the dhcp6.client-id or use xxd to translate it in ascii.

```bash
cisco@galaxy-42$ echo "00:02:00:00:00:09:46:4f:43:31:39:34:37:52:31:34:33" | xxd -pe -r && echo -e
        FOC1947R143
cisco@galaxy-42$ echo -n "FOC1947R143" | od -A n -t x1 | sed 's/^ /00:02:00:00:00:09:/' | sed 's/ /:/g'
    00:02:00:00:00:09:46:4f:43:31:39:34:37:52:31:34:33
```

### Dynamic Scripting - Embedding iPXE variables in URL

The URL provided by the DHCP server does not have to be a static. For example, you could direct iPXE to boot from the URL

http://172.30.0.22/boot.php?mac=${net0/mac}&product=${product:uristring}&serial=${serial:uristring}

Which would expand to a URL such as

http://172.30.0.22/boot.php?mac=c4:72:95:a7:ef:c0&product=NCS5001&serial=FOC1947R143

The boot.php program running on the web server could dynamically generate a script based on the information provided in the URL. For example, boot.php could look up the serial number in a MySQL database to determine the correct target to boot from, and then dynamically generate a script such as
    
```php
<?php
   header ( "Content-type: text/plain" );
   echo "#!ipxe \n";
   echo "set myURL http://172.30.0.22/Cisco/NCS/NCS5001/FOC1947R143 \n";
   echo "boot myURL \n";
?>
```

## iPXE with Chainloading

Chainloading is the capability to jump from one boot statement to another. Using chainloading and the embedded scripting capability of iPXE we can have a very detail and complex selection mechanism for the boot image. In the following example we will use the boot file structure illustrated below and we will use the initial DHCP configuration described earlier but in place of providing the URI for an ISO the DHCP server will provide the URI to a iPXE boot script (boot.ipxe).
![chainloading.png]({{site.baseurl}}/images/chainloading.png)

The file boot.ipxe file is a script that will identify the correct image based on available iPXE variable, it starts with the "!ipxe" statement and include statement like chain isset, etc.. All the iPXE statements are documented in the iPXE command section [open source boot firmware](http://ipxe.org/cmd)

The script is evaluated top to bottom and works for both IPv4 and IPv6

### iPXE Script

```
!ipxe
 
# Global variables used by all other iPXE scripts
chain --autofree boot.ipxe.cfg ||
 
# Boot <boot-url>/<boot-dir>/hostname-<hostname>.ipxe
# if hostname DHCP variable is set and script is present
isset ${hostname} && chain --replace --autofree ${boot-dir}hostname-${hostname}.ipxe ||
 
# Boot <boot-url>/<boot-dir>/uuid-<UUID>.ipxe
# if SMBIOS UUID variable is set and script is present (not usable see CSCuz28164)
isset ${uuid} && chain --replace --autofree ${boot-dir}uuid-${uuid}.ipxe ||
 
# Boot <boot-url>/<boot-dir>/mac-010203040506.ipxe if script is present
chain --replace --autofree ${boot-dir}mac-${mac:hexraw}.ipxe ||
 
# Boot <boot-url>/<boot-dir>/serial-FOC1947R143.ipxe if script is present
isset ${serial} && chain --replace --autofree ${boot-dir}serial-${serial}.ipxe ||
 
# Boot <boot-url>/<boot-dir>/pid-<product>.ipxe if script is present
isset ${product} && chain --replace --autofree ${boot-dir}pid-${product}.ipxe ||

# Boot <boot-url>/menu.ipxe script if all other options have been exhausted
chain --replace --autofree ${menu-url} ||
chain --replace --autofree ${menu-url6} ||
```

The first action of the script is to import a set of variables from boot.ipxe.cfg this will set ${boot-url} / ${boot-url6} and other variables.

The script verify if a specific variable has been set either in the SMBIOS of the system or in the DHCP response from the server.

If the variable has been set, the script attempts to jump to a secondary boot file. For example if the serial number is set "isset ${serial}", the script will attempt to jump to the file <boot-dir>/serial-FOC1947R144.ipxe if the file exists. If the file exist iPXE will start executing statement from this boot script. If the file does not exist the script continue to the next statement until it reaches the menu statement, the last statement of the list.

Here is an example of a secondary boot script based on the serial number of the device, as you can see this script points to the last element of the chain: the ISO boot file.

```
cisco@galaxy-42:/var/www/html/ipxe$ cat serial-FOC1947R143.ipxe
#!ipxe
echo
echo Booting NCS5K Mini ISO 6.0.0 from ISO for ${initiator}
chain --replace --autofree  ${boot-url}ncs5k-mini-x.iso-6.0.0 ||
chain --replace --autofree  ${boot-url6}ncs5k-mini-x.iso-6.0.0
```

Finally if all boot items have failed, the menu.ipxe script is executed and propose an interactive menu-driven list of boot options.

Below is the example script for the boot menu, this example is adapted from [https://gist.github.com/robinsmidsrod/2234639](https://gist.github.com/robinsmidsrod/2234639)

Each menu items can be associated with a shortcut key and navigation between items is done using the up and down arrows, for xrv9k image we have to use the sanboot option, for NCS-5K and NCS-5500 device we use the boot keyword.

**boot.ipxe.cfg**

```
#!ipxe

# Base URL used to resolve most resources
# Should always end with a slash
set boot-url http://172.30.0.22/
set boot-url6 http://[fd:30::172:30:0:22]/

# What URL to use when sanbooting
# Should always end with a slash
set sanboot-url http://172.30.0.22/
set sanboot-url6 http://[fd:30::172:30:0:22]/

# Relative directory to boot.ipxe used to
# override boot script for specific clients
set boot-dir ipxe/

# Absolute URL to the menu script, used by boot.ipxe
# and commonly used at the end of simple override scripts
# in ${boot-dir}.
set menu-url ${boot-url}menu.ipxe
set menu-url6 ${boot-url6}menu.ipxe

set initiator ${product} - ${serial}

```

**boot.ipxe**

```
!ipxe
# Variables are specified in boot.ipxe.cfg
# Some menu defaults
set menu-timeout 30000
set submenu-timeout ${menu-timeout}
isset ${menu-default} || set menu-default exit
###################### MAIN MENU ####################################
:start
menu iPXE boot menu for ${initiator}
item --gap --             ------------------------- XRV9K Boot Menu ------------------------------
item --key a sunstone-mini              Boot xrv9k Mini 6.0.0 ISO
item --key d sunstone-latest            Boot xrv9k Mini 6.1.1 ISO Latest build
item --key e sunstone-disk              Boot xrv9k from local disk
item --gap --             ------------------------ NCS5000 Boot Menu -----------------------------
item --key f ncs5000-6.0.0              Boot ncs-5000 Mini 6.0.0 ISO
item --key g ncs5000-6.1.1              Boot ncs-5000 Mini 6.1.1 ISO
item --gap --             ------------------------ NCS5500 Boot Menu -----------------------------
item --key h ncs5500-6.0.0              Boot ncs-5500 Mini 6.0.0 ISO
item --key i ncs5500-6.1.1              Boot ncs-5500 Mini 6.1.1. Latest ISO
item --gap --             ------------------------- Advanced options -----------------------------
item --key j config                     Configure settings
item shell                              Drop to iPXE shell
item reboot                             Reboot System
item
item --key x exit                       Exit iPXE and continue BIOS boot
choose --timeout ${menu-timeout} --default ${menu-default} selected || goto cancel
set menu-timeout 0
goto ${selected}
 
:cancel
echo You cancelled the menu, dropping you to a shell
 
:shell
echo Type 'exit' to get the back to the menu
shell
set menu-timeout 0
set submenu-timeout 0
goto start
 
:failed
echo Booting failed, dropping to shell
goto shell
 
:reboot
reboot
 
:exit
exit
    
:config
config
goto start
 
:back
set submenu-timeout 0
clear submenu-default
goto start
 
############ MAIN MENU ITEMS ############
 
:sunstone-mini
echo Booting XRV9K Mini 6.0.0 from ISO for ${initiator}
sanboot ${sanboot-url}xrv9k-mini-x.iso-6.0.0 ||
sanboot ${sanboot-url6}xrv9k-mini-x.iso-6.0.0 || goto failed
goto start
 
:sunstone-latest
echo Booting XRV9K Mini 6.1.1 latest developer release from ISO for ${initiator}
sanboot ${sanboot-url}xrv9k-mini-latest.iso ||
sanboot ${sanboot-url6}xrv9k-mini-latest.iso || goto failed
goto start
 
:ncs5000-6.0.0
echo
echo Booting NCS-5K Mini ISO 6.0.0 from ISO for ${initiator}
boot ${boot-url}ncs5000-mini.official ||
boot ${boot-url6}ncs5000-mini.official || goto failed
goto start
 
:ncs5000-6.1.1
echo
echo Booting NCS-5K Mini ISO 6.1.1 from ISO for ${initiator}
boot ${boot-url}ncs5000-mini.latest ||
boot ${boot-url6}ncs5000-mini.latest || goto failed
goto start
 
:ncs5500-6.0.0
echo
echo Booting NCS-5500 Mini ISO 6.0.0 from ISO for ${initiator}
boot ${boot-url}ncs5500-mini.official ||
boot ${boot-url6}ncs5500-mini.official || goto failed
goto start
 
:ncs5500-6.1.1
echo
echo Booting NCS-5500 Mini ISO 6.1.1 from ISO for ${initiator}
boot ${boot-url}ncs5500-mini.latest ||
boot ${boot-url6}ncs5500-mini.latest || goto failed
goto start
 
:sunstone-disk
echo Start XRV9K from disk
sanboot --no-describe --drive 0x80 || goto failed  
goto start
```

Here is a screenshots of the boot process with only the DHCPv6 service active and no valid boot file present.

```
iPXE> autoboot net0                                             <- autoboot from the mgmt interface 
net0: c4:72:95:a7:ef:c0 using dh8900cc on PCI01:00.1 (open)
  [Link:up, TX:108 TXE:0 RX:5188624 RXE:5186887]
Configuring (net0 c4:72:95:a7:ef:c0).......... ok
net0: fe80::c672:95ff:fea7:efc0/64
net0: fd:30:12::1124/64 gw fe80::fa72:eaff:fe8b:ce80            <- ipv6 statefull address assignment 
Filename: http://[fd:30::172:30:0:22]/boot.ipxe                 <- ipv6 boot URI from DHCPv6
http://[fd:30::172:30:0:22]/boot.ipxe... ok                     <- boot script is downloaded 
/boot.ipxe.cfg... ok                                            <- boot variable are chained
/ipxe/uuid-03000200-0400-0500-0006-000700080009.ipxe... No such file or directory (http://ipxe.org/2d0c618e)
/ipxe/mac-c47295a7efc0.ipxe... No such file or directory (http://ipxe.org/2d0c618e)
/ipxe/serial-FOC1947R143.ipxe... No such file or directory (http://ipxe.org/2d0c618e)
/ipxe/pid-NCS-5001.ipxe... No such file or directory (http://ipxe.org/2d0c618e)
http://172.30.0.22/menu.ipxe... Network unreachable (http://ipxe.org/280a6090)
http://[fd:30::172:30:0:22]/menu.ipxe... ok                      <- boot menu is executed

                 iPXE boot menu for NCS-5001 - FOC1947R143
 
------------------------- XRV9K Boot Menu ------------------------------ 
Boot xrv9k Mini 6.0.0 ISO
Boot xrv9k Mini 6.1.1 ISO Latest build
Boot xrv9k from local disk
------------------------ NCS5000 Boot Menu -----------------------------
Boot ncs-5000 Mini 6.0.0 ISO
Boot ncs-5000 Mini 6.1.1 ISO Latest build
------------------------ NCS5500 Boot Menu -----------------------------
Boot ncs-5500 Mini 6.0.0 ISO
Boot ncs-5500 Mini 6.1.1 Latest build
------------------------- Advanced options -----------------------------
Configure settings
Drop to iPXE shell
Reboot System
 
Exit iPXE and continue BIOS boot
```

If we select the entry "Boot ncs-5000 Mini 6.0.0 ISO", the script will first attempt to boot using the IPv4 address, since our device did not receive a valid IPv4 address it will attempt to use the IPv6 address and start the NOS installation.

```
Booting Skywarp Mini ISO 6.0.0 from ISO for NCS-5001 - FOC1947R143
http://172.30.0.22/ncs5000-mini.official... Network unreachable (http://ipxe.org/280a6090)
http://[fd:30::172:30:0:22]/ncs5000-mini.official... ok
Booting iso-image@0x42e2cb000(835930112), bzImage@0x42e2f7000(4473806)
```

## Conclusions

iPXE offers a wide variety of configuration paradigm that can be used in large deployment, with its scripting capability, iPXE is independent of DHCP configuration and can achieve very good granularity based on model number, serial number, mac-address, host name, etc.

Creation of boot file can be automated easily on the back-end side without restarting any services. On the HTTP server symbolic link can be used to move devices from one ISO to another without reconfiguration. with its backward compatibility with PXE and its low resources requirement, it is a very good alternative to ONIE.

Future enhancement to the boot process including secure boot will bring even more security to the iPXE without using HTTPS.
