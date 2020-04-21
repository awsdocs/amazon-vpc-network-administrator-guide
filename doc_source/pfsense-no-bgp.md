# Example: Netgate PfSense Device without Border Gateway Protocol<a name="pfsense-no-bgp"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

This topic provides an example of how to configure your router if your customer gateway is a Netgate pfSense firewall running OS 2\.2\.5 or later\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#pfsense-high-level)
+ [Example Configuration](#pfsense-example)
+ [How to Test the Customer Gateway Configuration](#pfsense-test)

## A High\-Level View of the Customer Gateway Device<a name="pfsense-high-level"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels: *Tunnel 1* and *Tunnel 2*\. Using redundant tunnels ensures continuous availability in the case that a device fails\. 

You should use the real configuration information that you receive from your integration team and apply it to your customer gateway device\. 

![\[Generic high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-nobgp-diagram.png)

## Example Configuration<a name="pfsense-example"></a>

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-12345678\), virtual private gateway ID \(vgw\-12345678\), and placeholders for the AWS endpoints \(AWS\_ENDPOINT\_1 and AWS\_ENDPOINT\_2\)\. 

In the following example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

**Important**  
The following configuration information is an example of what you can expect an integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
! Amazon Web Services
! Virtual Private Cloud

! AWS utilizes unique identifiers to manipulate the configuration of 
! a VPN Connection. Each VPN Connection is assigned an identifier and is 
! associated with two other identifiers, namely the 
! Customer Gateway Identifier and Virtual Private Gateway Identifier.
!
! Your VPN Connection ID 		  : vpn-12345678
! Your Virtual Private Gateway ID  : vgw-12345678
! Your Customer Gateway ID		  : cgw-12345678
!
!
! This configuration consists of two tunnels. Both tunnels must be 
! configured on your Customer Gateway for redundancy.
!
! --------------------------------------------------------------------------------
! IPSec Tunnel #1
! --------------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration
!
! A policy is established for the supported ISAKMP encryption, authentication, Diffie-Hellman, lifetime, 
! and key parameters.The IKE peer is configured with the supported IKE encryption,  authentication, Diffie-Hellman, lifetime, and key 
! parameters.Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256,  or other DH 
! groups like 2, 14-18, 22, 23, and 24. The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT). To 
! ensure that NAT traversal (NAT-T) can function, you must adjust your firewall 
! rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
!
Go to VPN-->IPSec. Add a new Phase1 entry (click + button )

General information
 a. Disabled : uncheck
 b. Key Exchange version :V1
 c. Internet Protocol : IPv4
 d. Interface : WAN
 e. Remote Gateway: AWS_ENPOINT_1
 f. Description: Amazon-IKE-vpn-12345678-0
 
 Phase 1 proposal (Authentication)
 a. Authentication Method: Mutual PSK
 b. Negotiation mode : Main
 c. My identifier : My IP address
 d. Peer identifier : Peer IP address
 e. Pre-Shared Key: plain-text-password1
 
 Phase 1 proposal (Algorithms)
 a. Encryption algorithm : aes128 
 b. Hash algorithm :  sha1
 c. DH key group :  2
 d. Lifetime : 28800 seconds
 
 Advanced Options
 a. Disable Rekey : uncheck
 b. Responder Only : uncheck
 c. NAT Traversal : Auto
 d. Deed Peer Detection : Enable DPD
    Delay between requesting peer acknowledgement : 10 seconds
	Number of consecutive failures allowed before disconnect : 3 retries
	
	

! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.

Expand the VPN configuration clicking in "+" and then create a new Phase2 entry as follows:

 a. Disabled :uncheck
 b. Mode : Tunnel
 c. Local Network : Type: LAN subnet
    Address :  ! Enter your local network CIDR in the Address tab 
 d. Remote Network : Type : Network 
    Address :  ! Enter your remote network CIDR in the Address tab
 e. Description : Amazon-IPSec-vpn-12345678-0
 
 Phase 2 proposal (SA/Key Exchange)
 a. Protocol : ESP
 b. Encryption algorigthms :aes128 
  c. Hash algorithms : sha1
  d. PFS key group :   2
e. Lifetime : 3600 seconds 

Advanced Options

Automatically ping host : ! Provide the IP address of an EC2 instance in VPC that will respond to ICMP.


! --------------------------------------------------------------------------------


! --------------------------------------------------------------------------------
! IPSec Tunnel #2
! --------------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration
!
! A policy is established for the supported ISAKMP encryption, authentication, Diffie-Hellman, lifetime, 
! and key parameters.The IKE peer is configured with the supported IKE encryption,  authentication, Diffie-Hellman, lifetime, and key 
! parameters.Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256,  or other DH 
! groups like 2, 14-18, 22, 23, and 24. The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT). To 
! ensure that NAT traversal (NAT-T) can function, you must adjust your firewall 
! rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
!
Go to VPN-->IPSec. Add a new Phase1 entry (click + button )

General information
 a. Disabled : uncheck
 b. Key Exchange version :V1
 c. Internet Protocol : IPv4
 d. Interface : WAN
 e. Remote Gateway: AWS_ENPOINT_2
 f. Description: Amazon-IKE-vpn-12345678-1
 
 Phase 1 proposal (Authentication)
 a. Authentication Method: Mutual PSK
 b. Negotiation mode : Main
 c. My identifier : My IP address
 d. Peer identifier : Peer IP address
 e. Pre-Shared Key: plain-text-password2
 
 Phase 1 proposal (Algorithms)
 a. Encryption algorithm : aes128 
 b. Hash algorithm :  sha1
 c. DH key group :  2
 d. Lifetime : 28800 seconds
 
 Advanced Options
 a. Disable Rekey : uncheck
 b. Responder Only : uncheck
 c. NAT Traversal : Auto
 d. Deed Peer Detection : Enable DPD
    Delay between requesting peer acknowledgement : 10 seconds
	Number of consecutive failures allowed before disconnect : 3 retries
	
	

! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.

Expand the VPN configuration clicking in "+" and then create a new Phase2 entry as follows:

 a. Disabled :uncheck
 b. Mode : Tunnel
 c. Local Network : Type: LAN subnet
    Address :  ! Enter your local network CIDR in the Address tab 
 d. Remote Network : Type : Network 
    Address :  ! Enter your remote network CIDR in the Address tab
 e. Description : Amazon-IPSec-vpn-12345678-1
 
 Phase 2 proposal (SA/Key Exchange)
 a. Protocol : ESP
 b. Encryption algorigthms :aes128 
  c. Hash algorithms : sha1
  d. PFS key group :   2
e. Lifetime : 3600 seconds 

Advanced Options

Automatically ping host : ! Provide the IP address of an EC2 instance in VPC that will respond to ICMP.
```

## How to Test the Customer Gateway Configuration<a name="pfsense-test"></a>

You must first test the gateway configuration for each tunnel\.

**To test the customer gateway configuration for each tunnel**
+ In the Amazon VPC console, ensure that a static route has been added to the VPN connection so that traffic can get back to your customer gateway\. For example, if your local subnet prefix is 198\.10\.0\.0/16, you must add a static route with that CIDR range to your VPN connection\. Make sure that both tunnels have a static route to your VPC\. 

Next you must test the connectivity for each tunnel by launching an instance into your VPC, and pinging the instance from your home network\. Before you begin, make sure of the following:
+ Use an AMI that responds to ping requests\. We recommend that you use one of the Amazon Linux AMIs\.
+ Configure your instance's security group and network ACL to enable inbound ICMP traffic\.
+ Ensure that you have configured routing for your VPN connection \- your subnet's route table must contain a route to the virtual private gateway\. For more information, see [Enable Route Propagation in Your Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_VPN.html#vpn-configure-routing) in the *Amazon VPC User Guide*\.

**To test the end\-to\-end connectivity of each tunnel**

1. Launch an instance from one of the Amazon Linux AMIs into your VPC\. The Amazon Linux AMIs are available in the Quick Start menu when you use the Launch Instances Wizard in the Amazon EC2 console\. For more information, see [Launching an Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html)in the *Amazon EC2 User Guide for Linux Instances*\.

1. After the instance is running, get its private IP address \(for example, 10\.0\.0\.4\)\. The console displays the address as part of the instance's details\.

1. On a system in your home network, use the ping command with the instance's IP address\. Make sure that the computer you ping from is behind the customer gateway\. A successful response should be similar to the following\.

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
If you ping an instance from your customer gateway router, ensure that you are sourcing ping messages from an internal IP address, not a tunnel IP address\. Some AMIs don't respond to ping messages from tunnel IP addresses\.

1. \(Optional\) To test tunnel failover, you can temporarily disable one of the tunnels on your customer gateway, and repeat the above step\. You cannot disable a tunnel on the AWS side of the VPN connection\. 