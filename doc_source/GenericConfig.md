# Example: Generic Customer Gateway Device Using Border Gateway Protocol<a name="GenericConfig"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

If your customer gateway device isn't one of the types discussed earlier in this guide, your integration team can provide you with generic information that you can use to configure your customer gateway device\. This section contains an example of that information\. 

Two diagrams illustrate the example configuration\. The first diagram shows the high\-level layout of the customer gateway device, and the second diagram shows details from the example configuration\. You should use the real configuration information that you receive from your integration team and apply it to your customer gateway device\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#HighLevelCustomerGateway5)
+ [A Detailed View of the Customer Gateway Device and an Example Configuration](#DetailedViewCustomerGateway5)
+ [How to Test the Customer Gateway Configuration](#TestingConfig5)

## A High\-Level View of the Customer Gateway Device<a name="HighLevelCustomerGateway5"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Generic high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-diagram.png)

## A Detailed View of the Customer Gateway Device and an Example Configuration<a name="DetailedViewCustomerGateway5"></a>

The diagram in this section illustrates an example generic customer gateway device\. Following the diagram, there is a corresponding example of the configuration information your integration team should provide\. The example configuration contains a set of information for each of the tunnels that you must configure\.

In addition, the example configuration refers to these items that you must provide:
+ *YOUR\_UPLINK\_ADDRESS*—The IP address for the Internet\-routable external interface on the customer gateway device\. The address must be static, and may be behind a device performing network address translation \(NAT\)\. To ensure that NAT traversal \(NAT\-T\) can function, you must adjust your firewall rules to unblock UDP port 4500\.
+ *YOUR\_BGP\_ASN*—The customer gateway's BGP ASN \(we use 65000 by default\)

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-44a8938f\), virtual private gateway ID \(vgw\-8db04f81\), the IP addresses \(72\.21\.209\.\*, 169\.254\.255\.\*\), and the remote ASN \(7224\)\. Replace these example values with the actual values from the configuration information that you receive\.

In the following diagram and example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

![\[Generic detailed diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/detailed-generic-diagram.png)

```
Amazon Web Services
Virtual Private Cloud

VPN Connection Configuration
===============================================
AWS utilizes unique identifiers to manipulate the configuration of 
a VPN Connection. Each VPN Connection is assigned a VPN identifier 
and is associated with two other identifiers, namely the 
Customer Gateway Identifier and the Virtual Private Gateway Identifier.

Your VPN Connection ID           : vpn-44a8938f
Your Virtual Private Gateway ID  : vgw-8db04f81
Your Customer Gateway ID         : cgw-b4dc3961

A VPN Connection consists of a pair of IPsec tunnel security associations (SAs). 
It is important that both tunnel security associations be configured. 


IPsec Tunnel #1
================================================
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png) 
#1: Internet Key Exchange Configuration

Configure the IKE SA as follows:
Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
The address of the external interface for your customer gateway must be a static address. 
Your customer gateway may reside behind a device performing network address translation (NAT).
To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
- IKE version              : IKEv1
- Authentication Method    : Pre-Shared Key 
- Pre-Shared Key           : plain-text-password1
- Authentication Algorithm : sha1
- Encryption Algorithm     : aes-128-cbc
- Lifetime                 : 28800 seconds
- Phase 1 Negotiation Mode : main
- Diffie-Hellman           : Group 2

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
#2: IPsec Configuration

Configure the IPsec SA as follows:
Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
- Protocol                 : esp
- Authentication Algorithm : hmac-sha1-96
- Encryption Algorithm     : aes-128-cbc
- Lifetime                 : 3600 seconds
- Mode                     : tunnel
- Perfect Forward Secrecy  : Diffie-Hellman Group 2

IPsec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
recommend configuring DPD on your endpoint as follows:
- DPD Interval             : 10
- DPD Retries              : 3

IPsec ESP (Encapsulating Security Payload) inserts additional
headers to transmit packets. These headers require additional space, 
which reduces the amount of space available to transmit application data.
To limit the impact of this behavior, we recommend the following 
configuration on your Customer Gateway:
- TCP MSS Adjustment       : 1387 bytes
- Clear Don't Fragment Bit : enabled
- Fragmentation            : Before encryption

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
#3: Tunnel Interface Configuration

Your Customer Gateway must be configured with a tunnel interface that is
associated with the IPsec tunnel. All traffic transmitted to the tunnel
interface is encrypted and transmitted to the Virtual Private Gateway.  

The Customer Gateway and Virtual Private Gateway each have two addresses that relate
to this IPsec tunnel. Each contains an outside address, upon which encrypted
traffic is exchanged. Each also contain an inside address associated with
the tunnel interface.

The Customer Gateway outside IP address was provided when the Customer Gateway
was created. Changing the IP address requires the creation of a new
Customer Gateway.

The Customer Gateway inside IP address should be configured on your tunnel
interface. 

Outside IP Addresses:
- Customer Gateway:        : YOUR_UPLINK_ADDRESS 
- Virtual Private Gateway  : 72.21.209.193

Inside IP Addresses
- Customer Gateway         : 169.254.255.2/30
- Virtual Private Gateway  : 169.254.255.1/30

Configure your tunnel to fragment at the optimal size:
- Tunnel interface MTU     : 1436 bytes

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)
#4: Border Gateway Protocol (BGP) Configuration:

The Border Gateway Protocol (BGPv4) is used within the tunnel, between the inside
IP addresses, to exchange routes from the VPC to your home network. Each
BGP router has an Autonomous System Number (ASN). Your ASN was provided 
to AWS when the Customer Gateway was created.

BGP Configuration Options:
- Customer Gateway ASN        : YOUR_BGP_ASN 
- Virtual Private Gateway ASN : 7224
- Neighbor IP Address         : 169.254.255.1
- Neighbor Hold Time          : 30

Configure BGP to announce routes to the Virtual Private Gateway. The gateway
will announce prefixes to your customer gateway based upon the prefix you 
assigned to the VPC at creation time.

IPsec Tunnel #2
=====================================================
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)
#1: Internet Key Exchange Configuration

Configure the IKE SA as follows:
Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
The address of the external interface for your customer gateway must be a static address. 
Your customer gateway may reside behind a device performing network address translation (NAT).
To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
- IKE version              : IKEv1
- Authentication Method    : Pre-Shared Key 
- Pre-Shared Key           : plain-text-password2
- Authentication Algorithm : sha1
- Encryption Algorithm     : aes-128-cbc
- Lifetime                 : 28800 seconds
- Phase 1 Negotiation Mode : main
- Diffie-Hellman           : Group 2

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
#2: IPsec Configuration

Configure the IPsec SA as follows:
Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
- Protocol                 : esp
- Authentication Algorithm : hmac-sha1-96
- Encryption Algorithm     : aes-128-cbc
- Lifetime                 : 3600 seconds
- Mode                     : tunnel
- Perfect Forward Secrecy  : Diffie-Hellman Group 2

IPsec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
recommend configuring DPD on your endpoint as follows:
- DPD Interval             : 10
- DPD Retries              : 3

IPsec ESP (Encapsulating Security Payload) inserts additional
headers to transmit packets. These headers require additional space, 
which reduces the amount of space available to transmit application data.
To limit the impact of this behavior, we recommend the following 
configuration on your Customer Gateway:
- TCP MSS Adjustment       : 1387 bytes
- Clear Don't Fragment Bit : enabled
- Fragmentation            : Before encryption

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
#3: Tunnel Interface Configuration

Your Customer Gateway must be configured with a tunnel interface that is
associated with the IPsec tunnel. All traffic transmitted to the tunnel
interface is encrypted and transmitted to the Virtual Private Gateway. 

The Customer Gateway and Virtual Private Gateway each have two addresses that relate
to this IPsec tunnel. Each contains an outside address, upon which encrypted
traffic is exchanged. Each also contain an inside address associated with
the tunnel interface.

The Customer Gateway outside IP address was provided when the Customer Gateway
was created. Changing the IP address requires the creation of a new
Customer Gateway.

The Customer Gateway inside IP address should be configured on your tunnel
interface. 

Outside IP Addresses:
- Customer Gateway:        : YOUR_UPLINK_ADDRESS 
- Virtual Private Gateway  : 72.21.209.193

Inside IP Addresses
- Customer Gateway         : 169.254.255.6/30
- Virtual Private Gateway  : 169.254.255.5/30  

Configure your tunnel to fragment at the optimal size:
- Tunnel interface MTU     : 1436 bytes

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png) " 
#4: Border Gateway Protocol (BGP) Configuration:

The Border Gateway Protocol (BGPv4) is used within the tunnel, between the inside
IP addresses, to exchange routes from the VPC to your home network. Each
BGP router has an Autonomous System Number (ASN). Your ASN was provided 
to AWS when the Customer Gateway was created.

BGP Configuration Options:
- Customer Gateway ASN        : YOUR_BGP_ASN 
- Virtual Private Gateway ASN : 7224
- Neighbor IP Address         : 169.254.255.5
- Neighbor Hold Time          : 30

Configure BGP to announce routes to the Virtual Private Gateway. The gateway
will announce prefixes to your customer gateway based upon the prefix you 
assigned to the VPC at creation time.
```

## How to Test the Customer Gateway Configuration<a name="TestingConfig5"></a>

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