# Example: SonicWALL Device<a name="sonicwall-bgp"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

This topic provides an example of how to configure your router if your customer gateway device is a SonicWALL router\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#sonicwall-bgp-overview)
+ [Example Configuration File](#sonicwall-bgp-config-file)
+ [Configuring the SonicWALL Device Using the Management Interface](#sonicwall-bgp-configure-device)
+ [How to Test the Customer Gateway Configuration](#sonicwall-bgp-test)

## A High\-Level View of the Customer Gateway Device<a name="sonicwall-bgp-overview"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels: *Tunnel 1* and *Tunnel 2*\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Customer gateway high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-diagram.png)

## Example Configuration File<a name="sonicwall-bgp-config-file"></a>

The configuration file downloaded from Amazon VPC includes the values needed to use the command line tools on OS 6\.2 to configure each tunnel and the IKE and IPsec settings for your SonicWALL device\. 

**Important**  
The following configuration information uses example values\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
! Amazon Web Services
! Virtual Private Cloud
!
! VPN Connection Configuration
! ================================================================================
! AWS utilizes unique identifiers to manipulate the configuration of
! a VPN Connection. Each VPN Connection is assigned a VPN Connection Identifier
! and is associated with two other identifiers, namely the
! Customer Gateway Identifier and the Virtual Private Gateway Identifier.
!
! Your VPN Connection ID                   : vpn-44a8938f
! Your Virtual Private Gateway ID          : vgw-8db04f81
! Your Customer Gateway ID                 : cgw-ff628496
!
! This configuration consists of two tunnels. Both tunnels must be 
! configured on your Customer Gateway.
!
! This configuration was tested on a SonicWALL TZ 600 running OS 6.2.5.1-26n
!
! You may need to populate these values throughout the config based on your setup:
! <vpc_subnet> - VPC address range
!
! IPSec Tunnel !1
! ================================================================================
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
! #1: Internet Key Exchange (IKE) Configuration
! 
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You can modify these sample configuration files to use AES128, SHA1, AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
config
address-object ipv4 AWSVPC network 172.30.0.0/16
vpn policy tunnel-interface vpn-44a8938f-1
gateway primary 72.21.209.193
bound-to interface X1
auth-method shared-secret 
shared-secret PRE-SHARED-KEY-IN-PLAIN-TEXT
ike-id local ip your_customer_gateway_IP_address 
ike-id peer ip 72.21.209.193
end
!
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
! #2: IPSec Configuration
! 
! The IPSec (Phase 2) proposal defines the protocol, authentication, 
! encryption, and lifetime parameters for our IPSec security association.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
!
config
proposal ipsec lifetime 3600
proposal ipsec authentication sha1
proposal ipsec encryption aes128
proposal ipsec perfect-forward-secrecy dh-group 2
proposal ipsec protocol ESP
keep-alive 
enable
commit 
end 
!
!
! You can use other supported IPSec parameters for encryption such as AES256, and other DH groups such as 2, 5, 14-18, 22, 23, and 24.
! IPSec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
! recommend configuring DPD on your endpoint as follows:
  - DPD Interval             : 120
  - DPD Retries              : 3
! To configure Dead Peer Detection for the SonicWall device, use the SonicOS management interface.
!
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png) 
! #3: Tunnel Interface Configuration
!  
! The tunnel interface is configured with the internal IP address.
!
! To establish connectivity between your internal network and the VPC, you
! must have an interface facing your internal network in the "Trust" zone.
!
config
tunnel-interface vpn T1
ip-assignment VPN static
ip 169.254.44.242 netmask 255.255.255.252
!
!
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  
! #4: Border Gateway Protocol (BGP) Configuration:
! 
! BGP is used within the tunnel to exchange prefixes between the
! Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway    
! will announce the prefix corresponding to your VPC.
! !
! The local BGP Autonomous System Number (ASN) (65000)
! is configured as part of your Customer Gateway. If the ASN must 
! be changed, the Customer Gateway and VPN Connection will need to be recreated with AWS.
!
routing
bgp
configure terminal 
router bgp YOUR_BGP_ASN
network <Local_subnet>/24
neighbor 169.254.44.242 remote-as 7224
neighbor 169.254.44.242 timers 10 30
neighbor 169.254.44.242 soft-reconfiguration inbound
end
write
exit
commit
end
!
! IPSec Tunnel #2
! --------------------------------------------------------------------------------
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
! #1: Internet Key Exchange (IKE) Configuration
! 
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You can modify these sample configuration files to use AES128, SHA1, AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
config
address-object ipv4 AWSVPC network 172.30.0.0/16
vpn policy tunnel-interface vpn-44a8938f-1
gateway primary 72.21.209.225
bound-to interface X1
auth-method shared-secret 
shared-secret PRE-SHARED-KEY-IN-PLAIN-TEXT
ike-id local ip your_customer_gateway_IP_address 
ike-id peer ip 72.21.209.225 
end
!
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
! #2: IPSec Configuration
! 
! The IPSec (Phase 2) proposal defines the protocol, authentication, 
! encryption, and lifetime parameters for our IPSec security association.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
!
config
proposal ipsec lifetime 3600
proposal ipsec authentication sha1
proposal ipsec encryption aes128
proposal ipsec perfect-forward-secrecy dh-group 2
proposal ipsec protocol ESP
keep-alive 
enable
commit 
end 
!
!
! You can use other supported IPSec parameters for encryption such as AES256, and other DH groups such as 2, 5, 14-18, 22, 23, and 24.
! IPSec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
! recommend configuring DPD on your endpoint as follows:
  - DPD Interval             : 120
  - DPD Retries              : 3
! To configure Dead Peer Detection for the SonicWall device, use the SonicOS management interface.
!
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png) 
! #3: Tunnel Interface Configuration
!  
! The tunnel interface is configured with the internal IP address.
!
! To establish connectivity between your internal network and the VPC, you
! must have an interface facing your internal network in the "Trust" zone.
!
config
tunnel-interface vpn T2
ip-assignment VPN static
ip 169.254.44.114 netmask 255.255.255.252
!
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  
! #4: Border Gateway Protocol (BGP) Configuration:
! 
! BGP is used within the tunnel to exchange prefixes between the
! Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway    
! will announce the prefix corresponding to your VPC.
!
! The local BGP Autonomous System Number (ASN) (65000)
! is configured as part of your Customer Gateway. If the ASN must 
! be changed, the Customer Gateway and VPN Connection will need to be recreated with AWS.
!
routing
bgp
configure terminal 
router bgp YOUR_BGP_ASN
network <Local_subnet>/24
neighbor 169.254.44.114 remote-as 7224
neighbor 169.254.44.114 timers 10 30
neighbor 169.254.44.114 soft-reconfiguration inbound
end
write
exit
commit
end
```

## Configuring the SonicWALL Device Using the Management Interface<a name="sonicwall-bgp-configure-device"></a>

You can also configure the SonicWALL device using the SonicOS management interface\. For more information, see [Configuring the SonicWALL Device Using the Management Interface](sonicwall-static.md#sonicwall-static-configure-device)\.

You cannot configure BGP for the device using the management interface\. Instead, use the command line instructions provided in the example configuration file above, under the section named **BGP**\.

## How to Test the Customer Gateway Configuration<a name="sonicwall-bgp-test"></a>

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

If your tunnels don't test successfully, see [Troubleshooting Generic Device Customer Gateway Connectivity Using Border Gateway Protocol](Generic_Troubleshooting.md)\.