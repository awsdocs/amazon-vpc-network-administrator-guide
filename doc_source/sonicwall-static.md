# Example: SonicWALL SonicOS Device Without Border Gateway Protocol<a name="sonicwall-static"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

This topic provides an example of how to configure your router if your customer gateway device is a SonicWALL router running SonicOS 5\.9 or 6\.2\. 

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#sonicwall-static-overview)
+ [Example Configuration File](#sonicwall-static-config-file)
+ [Configuring the SonicWALL Device Using the Management Interface](#sonicwall-static-configure-device)
+ [How to Test the Customer Gateway Configuration](#sonicwall-static-test)

## A High\-Level View of the Customer Gateway Device<a name="sonicwall-static-overview"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels: *Tunnel 1* and *Tunnel 2*\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Customer gateway high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-nobgp-diagram.png)

## Example Configuration File<a name="sonicwall-static-config-file"></a>

The configuration file downloaded from Amazon VPC includes the values needed to use the command line tools on OS 6\.2 and configure each tunnel and the IKE and IPsec settings for your SonicWALL device\. 

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
! configured on your customer gateway.
!
! This configuration was tested on a SonicWALL TZ 600 running OS 6.2.5.1-26n
!
! You may need to populate these values throughout the config based on your setup:
! <vpc_subnet> - VPC IP address range
! ================================================================================
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
! #1: Internet Key Exchange (IKE) Configuration
! 
! These sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
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
! You can use other supported IPSec parameters for encryption such as AES256, and other DH groups such as 1,2, 5, 14-18, 22, 23, and 24.

! IPSec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
! recommend configuring DPD on your endpoint as follows:
!  - DPD Interval             : 120
!  - DPD Retries              : 3
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
!
config
tunnel-interface vpn T1
ip-assignment VPN static
ip 169.254.255.6 netmask 255.255.255.252
exit
!
! 
! #4 Static Route Configuration
!
! Create a firewall policy permitting traffic from your local subnet to the VPC subnet and vice versa
! This example policy permits all traffic from the local subnet to the VPC through the tunnel interface.
!
!
policy interface T1 metric 1 source any destination name AWSVPC service any gateway 169.254.255.5
!    
IPSec Tunnel !2 
================================================================================
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png) 
! #1: Internet Key Exchange (IKE) Configuration
! 
! These sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You can modify these sample configuration files to use AES128, SHA1, AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
config
address-object ipv4 AWSVPC network 172.30.0.0/16
vpn policy tunnel-interface vpn-44a8938f-2
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
! You can use other supported IPSec parameters for encryption such as AES256, and other DH groups such as 1,2, 5, 14-18, 22, 23, and 24.
!
! IPSec Dead Peer Detection (DPD) will be enabled on the AWS Endpoint. We
! recommend configuring DPD on your endpoint as follows:
!  - DPD Interval             : 120
!  - DPD Retries              : 3
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
!
config
tunnel-interface vpn T2
ip-assignment VPN static
ip 169.254.255.2 netmask 255.255.255.252
!
! #4 Static Route Configuration
!
! Create a firewall policy permitting traffic from your local subnet to the VPC subnet and vice versa
! This example policy permits all traffic from the local subnet to the VPC through the tunnel interface.
!
!
policy interface T2 metric 1 source any destination name AWSVPC service any gateway 169.254.255.1
```

## Configuring the SonicWALL Device Using the Management Interface<a name="sonicwall-static-configure-device"></a>

The following procedure demonstrates how to configure the VPN tunnels on the SonicWALL device using the SonicOS management interface\. You must replace the example values in the procedures with the values that are provided in the configuration file\. 

**To configure the tunnels**

1. Open the SonicWALL SonicOS management interface\. 

1. In the left pane, choose **VPN**, **Settings**\. Under **VPN Policies**, choose **Add\.\.\.**\.

1. In the VPN policy window on the **General ** tab, complete the following information:
   + **Policy Type**: Choose **Tunnel Interface**\.
   + **Authentication Method**: Choose **IKE using Preshared Secret**\.
   + **Name**: Enter a name for the VPN policy\. We recommend that you use the name of the VPN ID, as provided in the configuration file\.
   + **IPsec Primary Gateway Name or Address**: Enter the IP address of the virtual private gateway \(AWS endpoint\) as provided in the configuration file; for example, `72.21.209.193`\.
   + **IPsec Secondary Gateway Name or Address**: Leave the default value\.
   + **Shared Secret**: Enter the pre\-shared key as provided in the configuration file, and enter it again in **Confirm Shared Secret**\.
   + **Local IKE ID**: Enter the IPv4 address of the customer gateway \(the SonicWALL device\)\. 
   + **Peer IKE ID**: Enter the IPv4 address of the virtual private gateway \(AWS endpoint\)\.

1. On the **Network** tab, complete the following information:
   + Under **Local Networks**, choose **Any address**\. We recommend this option to prevent connectivity issues from your local network\. 
   + Under **Remote Networks**, choose **Choose a destination network from list**\. Create an address object with the CIDR of your VPC in AWS\.

1. On the **Proposals** tab, complete the following information\. 
   + Under **IKE \(Phase 1\) Proposal**, do the following:
     + **Exchange**: Choose **Main Mode**\.
     + **DH Group**: Enter a value for the Diffie\-Hellman group; for example, `2`\. 
     + **Encryption**: Choose **AES\-128** or **AES\-256**\.
     + **Authentication**: Choose **SHA1** or **SHA256**\.
     + **Life Time**: Enter `28800`\.
   + Under **IKE \(Phase 2\) Proposal**, do the following:
     + **Protocol**: Choose **ESP**\.
     + **Encryption**: Choose **AES\-128** or **AES\-256**\.
     + **Authentication**: Choose **SHA1** or **SHA256**\.
     + Select the **Enable Perfect Forward Secrecy** check box, and choose the Diffie\-Hellman group\.
     + **Life Time**: Enter `3600`\.
**Important**  
If you created your virtual private gateway before October 2015, you must specify Diffie\-Hellman group 2, AES\-128, and SHA1 for both phases\.

1. On the **Advanced** tab, complete the following information:
   + Select **Enable Keep Alive**\.
   + Select **Enable Phase2 Dead Peer Detection** and enter the following:
     + For **Dead Peer Detection Interval**, enter `60` \(this is the minimum that the SonicWALL device accepts\)\.
     + For **Failure Trigger Level**, enter `3`\.
   + For **VPN Policy bound to**, select **Interface X1**\. This is the interface that's typically designated for public IP addresses\.

1. Choose **OK**\. On the **Settings** page, the **Enable** check box for the tunnel should be selected by default\. A green dot indicates that the tunnel is up\.

## How to Test the Customer Gateway Configuration<a name="sonicwall-static-test"></a>

You must first test the gateway configuration for each tunnel\.

**To test the customer gateway device configuration for each tunnel**
+ On your customer gateway device, verify that you have added a static route to the VPC CIDR IP space to use the tunnel interface\.

Next, you must test the connectivity for each tunnel by launching an instance into your VPC and pinging the instance from your home network\. Before you begin, make sure of the following:
+ Use an AMI that responds to ping requests\. We recommend that you use one of the Amazon Linux AMIs\.
+ Configure your instance's security group and network ACL to enable inbound ICMP traffic\.
+ Ensure that you have configured routing for your VPN connection; your subnet's route table must contain a route to the virtual private gateway\. For more information, see [Enable Route Propagation in Your Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_VPN.html#vpn-configure-routing) in the *Amazon VPC User Guide*\.

**To test the end\-to\-end connectivity of each tunnel**

1. Launch an instance of one of the Amazon Linux AMIs into your VPC\. The Amazon Linux AMIs are available in the **Quick Start** menu when you use the Launch Instances wizard in the AWS Management Console\. For more information, see the [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/)\.

1. After the instance is running, get its private IP address \(for example, 10\.0\.0\.4\)\. The console displays the address as part of the instance's details\.

1. On a system in your home network, use the ping command with the instance's IP address\. Make sure that the computer you ping from is behind the customer gateway\. A successful response should be similar to the following:

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

If your tunnels don't test successfully, see [Troubleshooting Generic Device Customer Gateway Connectivity Using Border Gateway Protocol](Generic_Troubleshooting.md)\.