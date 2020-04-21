# Example: Juniper ScreenOS Device<a name="Juniper-with-screenos"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

In this section, you get an example of the configuration information provided by your integration team if your customer gateway device is a Juniper SSG or Netscreen series device running Juniper ScreenOS software\.

Two diagrams illustrate the example configuration\. The first diagram shows the high\-level layout of the customer gateway device, and the second diagram shows details from the example configuration\. You should use the real configuration information that you receive from your integration team and apply it to your customer gateway device\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#CustomerGatewayView3)
+ [A Detailed View of the Customer Gateway Device and an Example Configuration](#CustomerGatewayDetailedView3)
+ [How to Test the Customer Gateway Configuration](#TestCustomerGatewayConfiguration3)

## A High\-Level View of the Customer Gateway Device<a name="CustomerGatewayView3"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Juniper ScreenOS high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-screenos-diagram.png)

## A Detailed View of the Customer Gateway Device and an Example Configuration<a name="CustomerGatewayDetailedView3"></a>

The diagram in this section illustrates an example Juniper ScreenOS customer gateway device\. Following the diagram, there is a corresponding example of the configuration information your integration team should provide\. The example configuration contains information for each of the tunnels that you must configure\.

In addition, the example configuration refers to these items that you must provide:
+ *YOUR\_UPLINK\_ADDRESS*—The IP address for the Internet\-routable external interface on the customer gateway device\. The address must be static, and may be behind a device performing network address translation \(NAT\)\. To ensure that NAT traversal \(NAT\-T\) can function, you must adjust your firewall rules to unblock UDP port 4500\.
+ *YOUR\_BGP\_ASN*—The customer gateway's BGP ASN \(we use 65000 by default\)

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-44a8938f\), virtual private gateway ID \(vgw\-8db04f81\), the IP addresses \(72\.21\.209\.\*, 169\.254\.255\.\*\), and the remote ASN \(7224\)\. Replace these example values with the actual values from the configuration information that you receive\.

In addition, you must:
+ Configure the outside interface \(referred to as *ethernet0/0* in the example configuration\)\.
+ Configure the tunnel interface IDs \(referred to as *tunnel\.1* and *tunnel\.2* in the example configuration\)\.
+ Configure all internal routing that moves traffic between the customer gateway and your local network\.

In the following diagram and example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

![\[Juniper ScreenOS detailed diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/detailed-screenos-diagram.png)

**Warning**  
The following configuration information is an example of what you can expect your integration team to provide\. Many of the values in the following example are different from the configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

**Important**  
The configuration below is appropriate for ScreenOS versions 6\.2 and later\. You can download a configuration that is specific to ScreenOS version 6\.1\. In the **Download Configuration** dialog box, select `Juniper Networks, Inc.` from the **Vendor** list, `SSG and ISG Series Routers` from the **Platform** list, and `ScreenOS 6.1` from the **Software** list\.

```
# Amazon Web Services
# Virtual Private Cloud
#
# AWS utilizes unique identifiers to manipulate the configuration of a VPN 
# Connection. Each VPN Connection is assigned a VPN Connection Identifier
# and is associated with two other identifiers, namely the Customer Gateway
# Identifier and the Virtual Private Gateway Identifier.
#
# Your VPN Connection ID          : vpn-44a8938f
# Your Virtual Private Gateway ID : vgw-8db04f81
# Your Customer Gateway ID        : cgw-b4dc3961
#
# This configuration consists of two tunnels. Both tunnels must be configured
# on your Customer Gateway.
#
# This configuration was tested on a Juniper SSG-5 running ScreenOS 6.3R2.
#
# --------------------------------------------------------------------------------
# IPsec Tunnel #1
# --------------------------------------------------------------------------------

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
# #1: Internet Key Exchange (IKE) Configuration
#
# A proposal is established for the supported IKE encryption, authentication,
# Diffie-Hellman, and lifetime parameters.
#
# Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
# You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
# The address of the external interface for your customer gateway must be a static address. 
# Your customer gateway may reside behind a device performing network address translation (NAT).
# To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
#
set ike p1-proposal ike-prop-vpn-44a8938f-1 preshare group2 esp aes128 sha-1 second 28800

# The IKE gateway is defined to be the Virtual Private Gateway. The gateway configuration
# associates a local interface, remote IP address, and IKE policy.
#
# This example shows the outside of the tunnel as interface ethernet0/0. This
# should be set to the interface that IP address YOUR_UPLINK_ADDRESS is
# associated with.
# This address is configured with the setup for your Customer Gateway.
#
#If the address changes, the Customer Gateway and VPN Connection must be recreated.
#

set ike gateway gw-vpn-44a8938f-1 address 72.21.209.225 id 72.21.209.225 main outgoing-interface ethernet0/0 preshare "plain-text-password1" proposal ike-prop-vpn-44a8938f-1

# Troubleshooting IKE connectivity can be aided by enabling IKE debugging.
# To do so, run the following commands:
# clear dbuf         -- Clear debug buffer
# debug ike all      -- Enable IKE debugging
# get dbuf stream    -- View debug messages
# undebug all        -- Turn off debugging

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  
# #2: IPsec Configuration
#
# The IPsec (Phase 2) proposal defines the protocol, authentication,
# encryption, and lifetime parameters for our IPsec security association.
# Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
#

set ike p2-proposal ipsec-prop-vpn-44a8938f-1 group2 esp aes128 sha-1 second 3600
set ike gateway gw-vpn-44a8938f-1 dpd-liveness interval 10
set vpn IPSEC-vpn-44a8938f-1 gateway gw-vpn-44a8938f-1 replay tunnel proposal ipsec-prop-vpn-44a8938f-1

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
# #3: Tunnel Interface Configuration
#
# The tunnel interface is configured with the internal IP address.
#
# To establish connectivity between your internal network and the VPC, you
# must have an interface facing your internal network in the "Trust" zone.
#

set interface tunnel.1 zone Trust
set interface tunnel.1 ip 169.254.255.2/30
set interface tunnel.1 mtu 1436
set vpn IPSEC-vpn-44a8938f-1 bind interface tunnel.1

# By default, the router will block asymmetric VPN traffic, which may occur
# with this VPN Connection. This occurs, for example, when routing policies
# cause traffic to sent from your router to VPC through one IPsec tunnel
# while traffic returns from VPC through the other.
#
# This command allows this traffic to be received by your device.

set zone Trust asymmetric-vpn


# This option causes the router to reduce the Maximum Segment Size of TCP
# packets to prevent packet fragmentation.
#

set flow vpn-tcp-mss 1387

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png) 
# #4: Border Gateway Protocol (BGP) Configuration
#
# BGP is used within the tunnel to exchange prefixes between the Virtual Private Gateway
# and your Customer Gateway. The Virtual Private Gateway will announce the prefix 
# corresponding to your VPC.
#
# Your Customer Gateway may announce a default route (0.0.0.0/0). 
#
# The BGP timers are adjusted to provide more rapid detection of outages.
#
# The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN) is configured
# as part of your Customer Gateway. If the ASN must be changed, the
# Customer Gateway and VPN Connection will need to be recreated with AWS.
#

set vrouter trust-vr
set max-ecmp-routes 2
set protocol bgp YOUR_BGP_ASN
set hold-time 30
set network 0.0.0.0/0
# To advertise additional prefixes to Amazon VPC, copy the 'network' statement and  
# identify the prefix you wish to advertise (set ipv4 network X.X.X.X/X). Make sure the  
# prefix is present in the routing table of the device with a valid next-hop. 

set enable
set neighbor 169.254.255.1 remote-as 7224
set neighbor 169.254.255.1 enable
exit
exit
set interface tunnel.1 protocol bgp			

# -------------------------------------------------------------------------
# IPsec Tunnel #2
# -------------------------------------------------------------------------

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
# #1: Internet Key Exchange (IKE) Configuration
#
# A proposal is established for the supported IKE encryption, authentication,
# Diffie-Hellman, and lifetime parameters.
# Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
# You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
# The address of the external interface for your customer gateway must be a static address. 
# Your customer gateway may reside behind a device performing network address translation (NAT).
# To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall !rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
#

set ike p1-proposal ike-prop-vpn-44a8938f-2 preshare group2 esp aes128 sha-1 second 28800

# The IKE gateway is defined to be the Virtual Private Gateway. The gateway configuration
# associates a local interface, remote IP address, and IKE policy.
#
# This example shows the outside of the tunnel as interface ethernet0/0. This
# should be set to the interface that IP address YOUR_UPLINK_ADDRESS is
# associated with.
#
# This address is configured with the setup for your Customer Gateway. If the
# address changes, the Customer Gateway and VPN Connection must be recreated.
#
set ike gateway gw-vpn-44a8938f-2 address 72.21.209.193 id 72.21.209.193 main outgoing-interface ethernet0/0 preshare "plain-text-password2" proposal ike-prop-vpn-44a8938f-2

# Troubleshooting IKE connectivity can be aided by enabling IKE debugging.
# To do so, run the following commands:
# clear dbuf         -- Clear debug buffer
# debug ike all      -- Enable IKE debugging
# get dbuf stream    -- View debug messages
# undebug all        -- Turn off debugging

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  
# #2: IPsec Configuration
#
# The IPsec (Phase 2) proposal defines the protocol, authentication,
# encryption, and lifetime parameters for our IPsec security association.
# Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
#

set ike p2-proposal ipsec-prop-vpn-44a8938f-2 group2 esp aes128 sha-1 second 3600
set ike gateway gw-vpn-44a8938f-2 dpd-liveness interval 10
set vpn IPSEC-vpn-44a8938f-2 gateway gw-vpn-44a8938f-2 replay tunnel proposal ipsec-prop-vpn-44a8938f-2

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
# #3: Tunnel Interface Configuration
#
# The tunnel interface is configured with the internal IP address.
#
# To establish connectivity between your internal network and the VPC, you
# must have an interface facing your internal network in the "Trust" zone.

set interface tunnel.2 zone Trust
set interface tunnel.2 ip 169.254.255.6/30
set interface tunnel.2 mtu 1436
set vpn IPSEC-vpn-44a8938f-2 bind interface tunnel.2

# By default, the router will block asymmetric VPN traffic, which may occur
# with this VPN Connection. This occurs, for example, when routing policies
# cause traffic to sent from your router to VPC through one IPsec tunnel
# while traffic returns from VPC through the other.
#
# This command allows this traffic to be received by your device.

set zone Trust asymmetric-vpn

# This option causes the router to reduce the Maximum Segment Size of TCP
# packets to prevent packet fragmentation.

set flow vpn-tcp-mss 1387

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  
# #4: Border Gateway Protocol (BGP) Configuration
#
# BGP is used within the tunnel to exchange prefixes between the Virtual Private Gateway
# and your Customer Gateway. The Virtual Private Gateway will announce the prefix 
# corresponding to your VPC.
#
# Your Customer Gateway may announce a default route (0.0.0.0/0).
#
# The BGP timers are adjusted to provide more rapid detection of outages.
#
# The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN) is configured
# as part of your Customer Gateway. If the ASN must be changed, the
# Customer Gateway and VPN Connection will need to be recreated with AWS.
#

set vrouter trust-vr
set max-ecmp-routes 2
set protocol bgp YOUR_BGP_ASN
set hold-time 30
set network 0.0.0.0/0
# To advertise additional prefixes to Amazon VPC, copy the 'network' statement and  
# identify the prefix you wish to advertise (set ipv4 network X.X.X.X/X). Make sure the  
# prefix is present in the routing table of the device with a valid next-hop. 
set enable
set neighbor 169.254.255.5 remote-as 7224
set neighbor 169.254.255.5 enable
exit
exit
set interface tunnel.2 protocol bgp
```

## How to Test the Customer Gateway Configuration<a name="TestCustomerGatewayConfiguration3"></a>

You can test the gateway configuration for each tunnel\.

**To test the customer gateway device configuration for each tunnel**

1. On your customer gateway device, determine whether the BGP status is `Established`\.

   It takes approximately 30 seconds for a BGP peering to be established\.

1. Ensure that the customer gateway device is advertising a route to the virtual private gateway\. The route may be the default route \(`0.0.0.0/0`\) or a more specific route you prefer\.

When properly established, your BGP peering should be receiving one route from the virtual private gateway corresponding to the prefix that your VPC integration team specified for the VPC \(for example, `10.0.0.0/24`\)\. If the BGP peering is established, you are receiving a prefix, and you are advertising a prefix, your tunnel is configured correctly\. Make sure that both tunnels are in this state\.

Next you must test the connectivity for each tunnel by launching an instance into your VPC, and pinging the instance from your home network\. Before you begin, make sure of the following:
+ Use an AMI that responds to ping requests\. We recommend that you use one of the Amazon Linux AMIs\.
+ Configure your instance's security group and network ACL to enable inbound ICMP traffic\.
+ Ensure that you have configured routing for your VPN connection: your subnet's route table must contain a route to the virtual private gateway\. For more information, see [Enable Route Propagation in Your Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_VPN.html#vpn-configure-routing) in the *Amazon VPC User Guide*\.

**To test the end\-to\-end connectivity of each tunnel**

1. Launch an instance of one of the Amazon Linux AMIs into your VPC\. The Amazon Linux AMIs are listed in the launch wizard when you launch an instance from the Amazon EC2 console\. For more information, see the [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/)\.

1. After the instance is running, get its private IP address \(for example, `10.0.0.4`\)\. The console displays the address as part of the instance's details\.

1. On a system in your home network, use the ping command with the instance's IP address\. Make sure that the computer you ping from is behind the customer gateway device\. A successful response should be similar to the following\.

   ```
   ping 10.0.0.4
   ```

   ```
   Pinging 10.0.0.4 with 32 bytes of data:
   
   Reply from 10.0.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.0.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.0.0.4: bytes=32 time<1ms TTL=128
   
   Ping statistics for 10.0.0.4:
   Packets: Sent = 3, Received = 3, Lost = 0 (0% loss),
   
   Approximate round trip times in milliseconds:
   Minimum = 0ms, Maximum = 0ms, Average = 0ms
   ```
**Note**  
If you ping an instance from your customer gateway device router, ensure that you are sourcing ping messages from an internal IP address, not a tunnel IP address\. Some AMIs don't respond to ping messages from tunnel IP addresses\.

1. \(Optional\) To test tunnel failover, you can temporarily disable one of the tunnels on your customer gateway device, and repeat the above step\. You cannot disable a tunnel on the AWS side of the VPN connection\.

If your tunnels don't test successfully, see [Troubleshooting Juniper ScreenOS Customer Gateway Connectivity](Juniper_ScreenOs_Troubleshooting.md)\.