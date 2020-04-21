# Example: Juniper SRX JunOS Device<a name="juniper-srx"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

In this section, you get an example of the configuration information provided by your integration team if your customer gateway device is a Juniper SRX router running JunOS 11\.0 \(or later\) software\.

Two diagrams illustrate the example configuration\. The first diagram shows the high\-level layout of the customer gateway device, and the second diagram shows details from the example configuration\. You should use the real configuration information that you receive from your integration team and apply it to your customer gateway device\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#juniper-srx-high-level)
+ [A Detailed View of the Customer Gateway Device and an Example Configuration](#juniper-srx-detailed)
+ [How to Test the Customer Gateway Configuration](#juniper-srx-test)

## A High\-Level View of the Customer Gateway Device<a name="juniper-srx-high-level"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Customer gateway high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-juniper-diagram.png)

## A Detailed View of the Customer Gateway Device and an Example Configuration<a name="juniper-srx-detailed"></a>

The diagram in this section illustrates an example Juniper JunOS 11\.0\+ customer gateway device\. Following the diagram, there is a corresponding example of the configuration information your integration team should provide\. The example configuration contains a set of information for each of the tunnels that you must configure\.

In addition, the example configuration refers to these items that you must provide:
+ *YOUR\_UPLINK\_ADDRESS*—The IP address for the Internet\-routable external interface on the customer gateway\. The address must be static, and may be behind a device performing network address translation \(NAT\)\. To ensure that NAT traversal \(NAT\-T\) can function, you must adjust your firewall rules to unblock UDP port 4500\.
+ *YOUR\_BGP\_ASN*—The customer gateway's BGP ASN \(we use 65000 by default\)

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-44a8938f\), virtual private gateway ID \(vgw\-8db04f81\), the IP addresses \(72\.21\.209\.\*, 169\.254\.255\.\*\), and the remote ASN \(7224\)\. Replace these example values with the actual values from the configuration information that you receive\.

In addition, you must:
+ Configure the outside interface \(referred to as *ge\-0/0/0\.0* in the example configuration\)\.
+ Configure the tunnel interface IDs \(referred to as *st0\.1* and *st0\.2* in the example configuration\)\.
+ Configure all internal routing that moves traffic between the customer gateway and your local network\.
+ Identify the security zone for the uplink interface \(the following configuration information uses the default "untrust" zone\)\.
+ Identify the security zone for the inside interface \(the following configuration information uses the default "trust" zone\)\.

In the following diagram and example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

![\[Juniper JunOS detailed diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/detailed-juniper-diagram.png)

**Warning**  
The following configuration information is an example of what you can expect your integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
# Amazon Web Services
# Virtual Private Cloud
#
# AWS utilizes unique identifiers to manipulate the configuration of 
# a VPN Connection. Each VPN Connection is assigned a VPN Connection 
# Identifier and is associated with two other identifiers, namely the 
# Customer Gateway Identifier and the Virtual Private Gateway Identifier.
#
# Your VPN Connection ID          : vpn-44a8938f
# Your Virtual Private Gateway ID : vgw-8db04f81
# Your Customer Gateway ID        : cgw-b4dc3961
#
# This configuration consists of two tunnels. Both tunnels must be 
# configured on your Customer Gateway.
#
# -------------------------------------------------------------------------
# IPsec Tunnel #1
# -------------------------------------------------------------------------
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
# #1: Internet Key Exchange (IKE) Configuration
#
# A proposal is established for the supported IKE encryption, 
# authentication, Diffie-Hellman, and lifetime parameters.
#
# Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
# You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
# The address of the external interface for your customer gateway must be a static address. 
# To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
#
set security ike proposal ike-prop-vpn-44a8938f-1 authentication-method pre-shared-keys 
set security ike proposal ike-prop-vpn-44a8938f-1 authentication-algorithm sha1
set security ike proposal ike-prop-vpn-44a8938f-1 encryption-algorithm aes-128-cbc
set security ike proposal ike-prop-vpn-44a8938f-1 lifetime-seconds 28800
set security ike proposal ike-prop-vpn-44a8938f-1 dh-group group2

# An IKE policy is established to associate a Pre Shared Key with the  
# defined proposal.
#
set security ike policy ike-pol-vpn-44a8938f-1 mode main 
set security ike policy ike-pol-vpn-44a8938f-1 proposals ike-prop-vpn-44a8938f-1
set security ike policy ike-pol-vpn-44a8938f-1 pre-shared-key ascii-text plain-text-password1

# The IKE gateway is defined to be the Virtual Private Gateway. The gateway 
# configuration associates a local interface, remote IP address, and
# IKE policy.
#
# This example shows the outside of the tunnel as interface ge-0/0/0.0.
# This should be set to the interface that IP address YOUR_UPLINK_ADDRESS is
# associated with.
# This address is configured with the setup for your Customer Gateway.
#
# If the address changes, the Customer Gateway and VPN Connection must 
# be recreated.
set security ike gateway gw-vpn-44a8938f-1 ike-policy ike-pol-vpn-44a8938f-1
set security ike gateway gw-vpn-44a8938f-1 external-interface ge-0/0/0.0
set security ike gateway gw-vpn-44a8938f-1 address 72.21.209.225
set security ike gateway gw-vpn-44a8938f-1 no-nat-traversal

# Troubleshooting IKE connectivity can be aided by enabling IKE tracing.
# The configuration below will cause the router to log IKE messages to
# the 'kmd' log. Run 'show messages kmd' to retrieve these logs.
# set security ike traceoptions file kmd
# set security ike traceoptions file size 1024768
# set security ike traceoptions file files 10
# set security ike traceoptions flag all

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  
# #2: IPsec Configuration
#
# The IPsec proposal defines the protocol, authentication, encryption, and
# lifetime parameters for our IPsec security association.
# Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.  
#
set security ipsec proposal ipsec-prop-vpn-44a8938f-1 protocol esp
set security ipsec proposal ipsec-prop-vpn-44a8938f-1 authentication-algorithm hmac-sha1-96
set security ipsec proposal ipsec-prop-vpn-44a8938f-1 encryption-algorithm aes-128-cbc
set security ipsec proposal ipsec-prop-vpn-44a8938f-1 lifetime-seconds 3600

# The IPsec policy incorporates the Diffie-Hellman group and the IPsec
# proposal.
#
set security ipsec policy ipsec-pol-vpn-44a8938f-1 perfect-forward-secrecy keys group2
set security ipsec policy ipsec-pol-vpn-44a8938f-1 proposals ipsec-prop-vpn-44a8938f-1

# A security association is defined here. The IPsec Policy and IKE gateways
# are associated with a tunnel interface (st0.1).
# The tunnel interface ID is assumed; if other tunnels are defined on
# your router, you will need to specify a unique interface name 
# (for example, st0.10).
#
set security ipsec vpn vpn-44a8938f-1 bind-interface st0.1
set security ipsec vpn vpn-44a8938f-1 ike gateway gw-vpn-44a8938f-1
set security ipsec vpn vpn-44a8938f-1 ike ipsec-policy ipsec-pol-vpn-44a8938f-1
set security ipsec vpn vpn-44a8938f-1 df-bit clear 

# This option enables IPsec Dead Peer Detection, which causes periodic
# messages to be sent to ensure a Security Association remains operational.
#
set security ike gateway gw-vpn-44a8938f-1 dead-peer-detection

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
# #3: Tunnel Interface Configuration
#

# The tunnel interface is configured with the internal IP address.
#
set interfaces st0.1 family inet address 169.254.255.2/30
set interfaces st0.1 family inet mtu 1436
set security zones security-zone trust interfaces st0.1

# The security zone protecting external interfaces of the router must be 
# configured to allow IKE traffic inbound.
#
set security zones security-zone untrust host-inbound-traffic system-services ike

# The security zone protecting internal interfaces (including the logical 
# tunnel interfaces) must be configured to allow BGP traffic inbound.
#
set security zones security-zone trust host-inbound-traffic protocols bgp

# This option causes the router to reduce the Maximum Segment Size of
# TCP packets to prevent packet fragmentation.
#
set security flow tcp-mss ipsec-vpn mss 1387

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  
# #4: Border Gateway Protocol (BGP) Configuration
#                                                                                     
# BGP is used within the tunnel to exchange prefixes between the
# Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway    
# will announce the prefix corresponding to your VPC.
#            
# Your Customer Gateway may announce a default route (0.0.0.0/0), 
# which can be done with the EXPORT-DEFAULT policy.
#
# To advertise additional prefixes to Amazon VPC, add additional prefixes to the "default" term
# EXPORT-DEFAULT policy. Make sure the prefix is present in the routing table of the device with 
# a valid next-hop.
#                                                                               
# The BGP timers are adjusted to provide more rapid detection of outages.       
# 
# The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN) is configured
# as part of your Customer Gateway. If the ASN must be changed, the 
# Customer Gateway and VPN Connection will need to be recreated with AWS.
#
# We establish a basic route policy to export a default route to the
# Virtual Private Gateway.       
#
set policy-options policy-statement EXPORT-DEFAULT term default from route-filter 0.0.0.0/0 exact                                                               
set policy-options policy-statement EXPORT-DEFAULT term default then accept     
set policy-options policy-statement EXPORT-DEFAULT term reject then reject

set protocols bgp group ebgp type external

set protocols bgp group ebgp neighbor 169.254.255.1 export EXPORT-DEFAULT 
set protocols bgp group ebgp neighbor 169.254.255.1 peer-as 7224
set protocols bgp group ebgp neighbor 169.254.255.1 hold-time 30
set protocols bgp group ebgp neighbor 169.254.255.1 local-as YOUR_BGP_ASN
			
# -------------------------------------------------------------------------
# IPsec Tunnel #2
# -------------------------------------------------------------------------
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
# #1: Internet Key Exchange (IKE) Configuration
#
# A proposal is established for the supported IKE encryption, 
# authentication, Diffie-Hellman, and lifetime parameters.
# Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
# You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
# The address of the external interface for your customer gateway must be a static address. 
# To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
#
set security ike proposal ike-prop-vpn-44a8938f-2 authentication-method pre-shared-keys 
set security ike proposal ike-prop-vpn-44a8938f-2 authentication-algorithm sha1
set security ike proposal ike-prop-vpn-44a8938f-2 encryption-algorithm aes-128-cbc
set security ike proposal ike-prop-vpn-44a8938f-2 lifetime-seconds 28800
set security ike proposal ike-prop-vpn-44a8938f-2 dh-group group2

# An IKE policy is established to associate a Pre Shared Key with the  
# defined proposal.
#
set security ike policy ike-pol-vpn-44a8938f-2 mode main 
set security ike policy ike-pol-vpn-44a8938f-2 proposals ike-prop-vpn-44a8938f-2
set security ike policy ike-pol-vpn-44a8938f-2 pre-shared-key ascii-text plain-text-password2

# The IKE gateway is defined to be the Virtual Private Gateway. The gateway 
# configuration associates a local interface, remote IP address, and
# IKE policy.
#
# This example shows the outside of the tunnel as interface ge-0/0/0.0.
# This should be set to the interface that IP address YOUR_UPLINK_ADDRESS is
# associated with.
# This address is configured with the setup for your Customer Gateway.
#
# If the address changes, the Customer Gateway and VPN Connection must be recreated.
#
set security ike gateway gw-vpn-44a8938f-2 ike-policy ike-pol-vpn-44a8938f-2
set security ike gateway gw-vpn-44a8938f-2 external-interface ge-0/0/0.0
set security ike gateway gw-vpn-44a8938f-2 address 72.21.209.193
set security ike gateway gw-vpn-44a8938f-2 no-nat-traversal

# Troubleshooting IKE connectivity can be aided by enabling IKE tracing.
# The configuration below will cause the router to log IKE messages to
# the 'kmd' log. Run 'show messages kmd' to retrieve these logs.
# set security ike traceoptions file kmd
# set security ike traceoptions file size 1024768
# set security ike traceoptions file files 10
# set security ike traceoptions flag all

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  
# #2: IPsec Configuration
#
# The IPsec proposal defines the protocol, authentication, encryption, and
# lifetime parameters for our IPsec security association.
# Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.  
#
set security ipsec proposal ipsec-prop-vpn-44a8938f-2 protocol esp
set security ipsec proposal ipsec-prop-vpn-44a8938f-2 authentication-algorithm hmac-sha1-96
set security ipsec proposal ipsec-prop-vpn-44a8938f-2 encryption-algorithm aes-128-cbc
set security ipsec proposal ipsec-prop-vpn-44a8938f-2 lifetime-seconds 3600

# The IPsec policy incorporates the Diffie-Hellman group and the IPsec
# proposal.
#
set security ipsec policy ipsec-pol-vpn-44a8938f-2 perfect-forward-secrecy keys group2
set security ipsec policy ipsec-pol-vpn-44a8938f-2 proposals ipsec-prop-vpn-44a8938f-2

# A security association is defined here. The IPsec Policy and IKE gateways
# are associated with a tunnel interface (st0.2).
# The tunnel interface ID is assumed; if other tunnels are defined on
# your router, you will need to specify a unique interface name 
# (for example, st0.20).
#
set security ipsec vpn vpn-44a8938f-2 bind-interface st0.2
set security ipsec vpn vpn-44a8938f-2 ike gateway gw-vpn-44a8938f-2
set security ipsec vpn vpn-44a8938f-2 ike ipsec-policy ipsec-pol-vpn-44a8938f-2
set security ipsec vpn vpn-44a8938f-2 df-bit clear 

# This option enables IPsec Dead Peer Detection, which causes periodic
# messages to be sent to ensure a Security Association remains operational.
#
set security ike gateway gw-vpn-44a8938f-2 dead-peer-detection

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
# #3: Tunnel Interface Configuration
#

# The tunnel interface is configured with the internal IP address.
#
set interfaces st0.2 family inet address 169.254.255.6/30
set interfaces st0.2 family inet mtu 1436
set security zones security-zone trust interfaces st0.2

# The security zone protecting external interfaces of the router must be 
# configured to allow IKE traffic inbound.
#
set security zones security-zone untrust host-inbound-traffic system-services ike

# The security zone protecting internal interfaces (including the logical 
# tunnel interfaces) must be configured to allow BGP traffic inbound.
#
set security zones security-zone trust host-inbound-traffic protocols bgp

# This option causes the router to reduce the Maximum Segment Size of
# TCP packets to prevent packet fragmentation.
#
set security flow tcp-mss ipsec-vpn mss 1387

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  
# #4: Border Gateway Protocol (BGP) Configuration
#                                                                                     
# BGP is used within the tunnel to exchange prefixes between the
# Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway    
# will announce the prefix corresponding to your VPC.
#            
# Your Customer Gateway may announce a default route (0.0.0.0/0), 
# which can be done with the EXPORT-DEFAULT policy.
#
# To advertise additional prefixes to Amazon VPC, add additional prefixes to the "default" term
# EXPORT-DEFAULT policy. Make sure the prefix is present in the routing table of the device with 
# a valid next-hop.
#                                                                             
# The BGP timers are adjusted to provide more rapid detection of outages.       
# 
# The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN) is configured
# as part of your Customer Gateway. If the ASN must be changed, the 
# Customer Gateway and VPN Connection will need to be recreated with AWS.
#
# We establish a basic route policy to export a default route to the
# Virtual Private Gateway.       
#
set policy-options policy-statement EXPORT-DEFAULT term default from route-filter 0.0.0.0/0 exact                                                               
set policy-options policy-statement EXPORT-DEFAULT term default then accept     
set policy-options policy-statement EXPORT-DEFAULT term reject then reject

set protocols bgp group ebgp type external

set protocols bgp group ebgp neighbor 169.254.255.5 export EXPORT-DEFAULT 
set protocols bgp group ebgp neighbor 169.254.255.5 peer-as 7224
set protocols bgp group ebgp neighbor 169.254.255.5 hold-time 30
set protocols bgp group ebgp neighbor 169.254.255.5 local-as YOUR_BGP_ASN
```

## How to Test the Customer Gateway Configuration<a name="juniper-srx-test"></a>

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

If your tunnels don't test successfully, see [Troubleshooting Juniper JunOS Customer Gateway Connectivity](Juniper_Troubleshooting.md)\.