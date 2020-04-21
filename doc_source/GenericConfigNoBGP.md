# Example: Generic Customer Gateway Device without Border Gateway Protocol<a name="GenericConfigNoBGP"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

If your customer gateway device isn't one of the types discussed earlier in this guide, your integration team can provide you with generic information that you can use to configure your customer gateway device\. This section contains an example of that information\. 

Two diagrams illustrate the example configuration\. The first diagram shows the high\-level layout of the customer gateway device, and the second diagram shows details from the example configuration\. You should use the real configuration information that you receive from your integration team and apply it to your customer gateway device\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#HighLevelCustomerGateway6)
+ [A Detailed View of the Customer Gateway Device and an Example Configuration](#DetailedViewCustomerGateway6)
+ [How to Test the Customer Gateway Configuration](#TestingConfig6)

## A High\-Level View of the Customer Gateway Device<a name="HighLevelCustomerGateway6"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels: *Tunnel 1* and *Tunnel 2*\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Generic high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-nobgp-diagram.png)

## A Detailed View of the Customer Gateway Device and an Example Configuration<a name="DetailedViewCustomerGateway6"></a>

The diagram in this section illustrates a generic customer gateway device that uses static routing for its VPN connection\. It does not support dynamic routing, or Border Gateway Protocol \(BGP\)\. Following the diagram, there is a corresponding example of the configuration information your integration team should give you\. The example configuration contains a set of information for each of the two tunnels you must configure\.

In addition, the example configuration refers to one item that you must provide:
+ *YOUR\_UPLINK\_ADDRESS*—The IP address for the Internet\-routable external interface on the customer gateway device\. The address must be static, and may be behind a device performing network address translation \(NAT\)\. To ensure that NAT traversal \(NAT\-T\) can function, you must adjust your firewall rules to unblock UDP port 4500\.

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-44a8938f\), virtual private gateway ID \(vgw\-8db04f81\), and the VGW IP addresses \(72\.21\.209\.\*, 169\.254\.255\.\*\)\. Replace these example values with the actual values from the configuration information that you receive\.

In the following diagram and example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

![\[Generic detailed diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/detailed-generic-nobgp-diagram.png)

**Important**  
The following configuration information is an example of what you can expect an integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
Amazon Web Services
Virtual Private Cloud

VPN Connection Configuration
================================================================================
AWS utilizes unique identifiers to manipulate the configuration of
a VPN Connection. Each VPN Connection is assigned a VPN Connection Identifier
and is associated with two other identifiers, namely the
Customer Gateway Identifier and the Virtual Private Gateway Identifier.

Your VPN Connection ID                   : vpn-44a8938f
Your Virtual Private Gateway ID          : vgw-8db04f81
Your Customer Gateway ID                 : cgw-ff628496

A VPN Connection consists of a pair of IPSec tunnel security associations (SAs).
It is important that both tunnel security associations be configured.

                
IPSec Tunnel #1
================================================================================
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png) 
#1: Internet Key Exchange Configuration

Configure the IKE SA as follows
Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
The address of the external interface for your customer gateway must be a static address. 
Your customer gateway may reside behind a device performing network address translation (NAT).
To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
  - IKE version              : IKEv1
  - Authentication Method    : Pre-Shared Key
  - Pre-Shared Key           : PRE-SHARED-KEY-IN-PLAIN-TEXT
  - Authentication Algorithm : sha1
  - Encryption Algorithm     : aes-128-cbc
  - Lifetime                 : 28800 seconds
  - Phase 1 Negotiation Mode : main
  - Diffie-Hellman           : Group 2

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
#2: IPSec Configuration

Configure the IPSec SA as follows:
Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
  - Protocol                 : esp
  - Authentication Algorithm : hmac-sha1-96
  - Encryption Algorithm     : aes-128-cbc
  - Lifetime                 : 3600 seconds
  - Mode                     : tunnel
  - Perfect Forward Secrecy  : Diffie-Hellman Group 2

IPSec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
recommend configuring DPD on your endpoint as follows:
  - DPD Interval             : 10
  - DPD Retries              : 3

IPSec ESP (Encapsulating Security Payload) inserts additional
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
associated with the IPSec tunnel. All traffic transmitted to the tunnel
interface is encrypted and transmitted to the Virtual Private Gateway.

The Customer Gateway and Virtual Private Gateway each have two addresses that relate
to this IPSec tunnel. Each contains an outside address, upon which encrypted
traffic is exchanged. Each also contain an inside address associated with
the tunnel interface.

The Customer Gateway outside IP address was provided when the Customer Gateway
was created. Changing the IP address requires the creation of a new
Customer Gateway.

The Customer Gateway inside IP address should be configured on your tunnel
interface.

Outside IP Addresses:
  - Customer Gateway                    : YOUR_UPLINK_ADDRESS
  - Virtual Private Gateway             : 72.21.209.193

Inside IP Addresses
  - Customer Gateway                    : 169.254.255.74/30
  - Virtual Private Gateway             : 169.254.255.73/30

Configure your tunnel to fragment at the optimal size:
  - Tunnel interface MTU     : 1436 bytes
    
#4: Static Routing Configuration:

To route traffic between your internal network and your VPC,
you will need a static route added to your router.

Static Route Configuration Options:

  - Next hop       : 169.254.255.73

You should add static routes towards your internal network on the VGW.
The VGW will then send traffic towards your internal network over
the tunnels.

                
                
IPSec Tunnel #2 
================================================================================
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
  - Pre-Shared Key           : PRE-SHARED-KEY-IN-PLAIN-TEXT
  - Authentication Algorithm : sha1
  - Encryption Algorithm     : aes-128-cbc
  - Lifetime                 : 28800 seconds
  - Phase 1 Negotiation Mode : main
  - Diffie-Hellman           : Group 2
  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
#2: IPSec Configuration

Configure the IPSec SA as follows:
Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
  - Protocol                 : esp
  - Authentication Algorithm : hmac-sha1-96
  - Encryption Algorithm     : aes-128-cbc
  - Lifetime                 : 3600 seconds
  - Mode                     : tunnel
  - Perfect Forward Secrecy  : Diffie-Hellman Group 2

IPSec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
recommend configuring DPD on your endpoint as follows:
  - DPD Interval             : 10
  - DPD Retries              : 3

IPSec ESP (Encapsulating Security Payload) inserts additional
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
associated with the IPSec tunnel. All traffic transmitted to the tunnel
interface is encrypted and transmitted to the Virtual Private Gateway.

The Customer Gateway and Virtual Private Gateway each have two addresses that relate
to this IPSec tunnel. Each contains an outside address, upon which encrypted
traffic is exchanged. Each also contain an inside address associated with
the tunnel interface.

The Customer Gateway outside IP address was provided when the Customer Gateway
was created. Changing the IP address requires the creation of a new
Customer Gateway.

The Customer Gateway inside IP address should be configured on your tunnel
interface.

Outside IP Addresses:
  - Customer Gateway                    : YOUR_UPLINK_ADDRESS
  - Virtual Private Gateway             : 72.21.209.225

Inside IP Addresses
  - Customer Gateway                    : 169.254.255.78/30
  - Virtual Private Gateway             : 169.254.255.77/30

Configure your tunnel to fragment at the optimal size:
  - Tunnel interface MTU     : 1436 bytes
    
#4: Static Routing Configuration:

To route traffic between your internal network and your VPC,
you will need a static route added to your router.

Static Route Configuration Options:

  - Next hop       : 169.254.255.77

You should add static routes towards your internal network on the VGW.
The VGW will then send traffic towards your internal network over
the tunnels.
```

## How to Test the Customer Gateway Configuration<a name="TestingConfig6"></a>

You must first test the gateway configuration for each tunnel\.

**To test the customer gateway device configuration for each tunnel**
+ On your customer gateway device, verify that you have added a static route to the VPC CIDR IP space to use the tunnel interface\.

Next you must test the connectivity for each tunnel by launching an instance into your VPC, and pinging the instance from your home network\. Before you begin, make sure of the following:
+ Use an AMI that responds to ping requests\. We recommend that you use one of the Amazon Linux AMIs\.
+ Configure your instance's security group and network ACL to enable inbound ICMP traffic\.
+ Ensure that you have configured routing for your VPN connection \- your subnet's route table must contain a route to the virtual private gateway\. For more information, see [Enable Route Propagation in Your Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_VPN.html#vpn-configure-routing) in the *Amazon VPC User Guide*\.

**To test the end\-to\-end connectivity of each tunnel**

1. Launch an instance of one of the Amazon Linux AMIs into your VPC\. The Amazon Linux AMIs are available in the Quick Start menu when you use the Launch Instances Wizard in the AWS Management Console\. For more information, see the [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/)\.

1. After the instance is running, get its private IP address \(for example, 10\.0\.0\.4\)\. The console displays the address as part of the instance's details\.

1. On a system in your home network, use the ping command with the instance's IP address\. Make sure that the computer you ping from is behind the customer gateway\. A successful response should be similar to the following\.

   ```
   PROMPT> ping 10.0.0.4
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
If you ping an instance from your customer gateway router, ensure that you are sourcing ping messages from an internal IP address, not a tunnel IP address\. Some AMIs don't respond to ping messages from tunnel IP addresses\.

If your tunnels don't test successfully, see [Troubleshooting Generic Device Customer Gateway Connectivity Using Border Gateway Protocol](Generic_Troubleshooting.md)\.