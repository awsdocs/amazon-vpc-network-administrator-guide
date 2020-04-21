# Example: Cisco IOS Device without Border Gateway Protocol<a name="Cisco_NoBGP"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

In this section, you get an example of the configuration information provided by your integration team if your customer gateway is a Cisco Integrated Services router running Cisco IOS software\.

Two diagrams illustrate the example configuration\. The first diagram shows the high\-level layout of the customer gateway, and the second diagram shows details from the example configuration\. You should use the real configuration information that you receive from your integration team, and apply it to your customer gateway\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway](#Cisco_NoBGP_overview)
+ [A Detailed View of the Customer Gateway and an Example Configuration](#Cisco_NoBGP_details)
+ [How to Test the Customer Gateway Configuration](#TestCustomerGateway_NoBGP)

## A High\-Level View of the Customer Gateway<a name="Cisco_NoBGP_overview"></a>

The following diagram shows the general details of your customer gateway\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Cisco ISO without BGP high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-cisco-asa-diagram.png)

## A Detailed View of the Customer Gateway and an Example Configuration<a name="Cisco_NoBGP_details"></a>

The diagram in this section illustrates an example Cisco IOS customer gateway \(without BGP\)\. Following the diagram, there is a corresponding example of the configuration information that your integration team should provide\. The example configuration contains a set of information for each of the tunnels that you must configure\.

In addition, the example configuration refers to this item that you must provide:
+ *YOUR\_UPLINK\_ADDRESS*—The IP address for the Internet\-routable external interface on the customer gateway\. The address must be static, and may be behind a device performing network address translation \(NAT\)\. To ensure that NAT traversal \(NAT\-T\) can function, you must adjust your firewall rules to unblock UDP port 4500\.

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-1a2b3c4d\), virtual private gateway ID \(vgw\-12345678\), the IP addresses \(205\.251\.233\.\*, 169\.254\.255\.\*\)\. Replace these example values with the actual values from the configuration information that you receive\.

In addition, you must:
+ Configure the outside interface\.
+ Configure the tunnel interface IDs \(referred to as *Tunnel1* and *Tunnel2* in the example configuration\)\.
+ Ensure that the Crypto ISAKMP Policy Sequence number is unique\.
+ Ensure that the Crypto IPsec Transform Set and the Crypto ISAKMP Policy Sequence are harmonious with any other IPsec tunnels configured on the device\.
+ Ensure that the SLA monitoring number is unique\.
+ Configure all internal routing that moves traffic between the customer gateway and your local network\.

In the following diagram and example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

![\[Cisco ISO without BGP detailed diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/detailed-cisco-nobgp-diagram.png)

**Warning**  
The following configuration information is an example of what you can expect your integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
! --------------------------------------------------------------------------------
! IPSec Tunnel #1
! --------------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration
!
! A policy is established for the supported ISAKMP encryption, 
! authentication, Diffie-Hellman, lifetime, and key parameters.
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
! Note that there are a global list of ISAKMP policies, each identified by 
! sequence number. This policy is defined as #200, which may conflict with
! an existing policy using the same number. If so, we recommend changing 
! the sequence number to avoid conflicts.
!
crypto isakmp policy 200
  encryption aes 128
  authentication pre-share
  group 2
  lifetime 28800
  hash sha
exit

! The ISAKMP keyring stores the Pre Shared Key used to authenticate the 
! tunnel endpoints.
!
crypto keyring keyring-vpn-1a2b3c4d-0
  local-address CUSTOMER_IP
  pre-shared-key address 205.251.233.121 key PASSWORD
exit

! An ISAKMP profile is used to associate the keyring with the particular 
! endpoint.
!
crypto isakmp profile isakmp-vpn-1a2b3c4d-0
  local-address CUSTOMER_IP
  match identity address 205.251.233.121
  keyring keyring-vpn-1a2b3c4d-0
exit

! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
!
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
!
crypto ipsec transform-set ipsec-prop-vpn-1a2b3c4d-0 esp-aes 128 esp-sha-hmac 
  mode tunnel
exit

! The IPSec profile references the IPSec transform set and further defines
! the Diffie-Hellman group and security association lifetime.
!
crypto ipsec profile ipsec-vpn-1a2b3c4d-0
  set pfs group2
  set security-association lifetime seconds 3600
  set transform-set ipsec-prop-vpn-1a2b3c4d-0
exit

! Additional parameters of the IPSec configuration are set here. Note that 
! these parameters are global and therefore impact other IPSec 
! associations.
! This option instructs the router to clear the "Don't Fragment" 
! bit from packets that carry this bit and yet must be fragmented, enabling
! them to be fragmented.
!
crypto ipsec df-bit clear

! This option enables IPSec Dead Peer Detection, which causes periodic
! messages to be sent to ensure a Security Association remains operational.
!
crypto isakmp keepalive 10 10 on-demand

! This configures the gateway's window for accepting out of order
! IPSec packets. A larger window can be helpful if too many packets 
! are dropped due to reordering while in transit between gateways.
!
crypto ipsec security-association replay window-size 128

! This option instructs the router to fragment the unencrypted packets
! (prior to encryption).
!
crypto ipsec fragmentation before-encryption
  

! --------------------------------------------------------------------------------
! #3: Tunnel Interface Configuration
!  
! A tunnel interface is configured to be the logical interface associated  
! with the tunnel. All traffic routed to the tunnel interface will be 
! encrypted and transmitted to the VPC. Similarly, traffic from the VPC
! will be logically received on this interface.
!
! Association with the IPSec security association is done through the 
! "tunnel protection" command.
!
! The address of the interface is configured with the setup for your 
! Customer Gateway.  If the address changes, the Customer Gateway and VPN 
! Connection must be recreated with Amazon VPC.
!
interface Tunnel1
  ip address 169.254.249.18 255.255.255.252
  ip virtual-reassembly
  tunnel source CUSTOMER_IP
  tunnel destination 205.251.233.121
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile ipsec-vpn-1a2b3c4d-0
  ! This option causes the router to reduce the Maximum Segment Size of
  ! TCP packets to prevent packet fragmentation.
  ip tcp adjust-mss 1387 
  no shutdown
exit

! ----------------------------------------------------------------------------
! #4 Static Route Configuration
!
! Your Customer Gateway needs to set a static route for the prefix corresponding to your 
! VPC to send traffic over the tunnel interface.
! An example for a VPC with the prefix 10.0.0.0/16 is provided below:
! ip route 10.0.0.0 255.255.0.0 Tunnel1 track 100 
!
! SLA Monitor is used to provide a failover between the two tunnels. If the primary tunnel fails, the redundant tunnel will automatically be used
! This sla is defined as #100, which may conflict with an existing sla using same number. 
! If so, we recommend changing the sequence number to avoid conflicts.
!
ip sla 100
   icmp-echo 169.254.249.17 source-interface Tunnel1
   timeout 1000
   frequency 5
exit
ip sla schedule 100  life forever start-time now
track 100 ip sla 100 reachability 
! --------------------------------------------------------------------------------
! --------------------------------------------------------------------------------
! IPSec Tunnel #2
! --------------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration
!
! A policy is established for the supported ISAKMP encryption, 
! authentication, Diffie-Hellman, lifetime, and key parameters.
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
! Note that there are a global list of ISAKMP policies, each identified by 
! sequence number. This policy is defined as #201, which may conflict with
! an existing policy using the same number. If so, we recommend changing 
! the sequence number to avoid conflicts.
!
crypto isakmp policy 201
  encryption aes 128
  authentication pre-share
  group 2
  lifetime 28800
  hash sha
exit

! The ISAKMP keyring stores the Pre Shared Key used to authenticate the 
! tunnel endpoints.
!
crypto keyring keyring-vpn-1a2b3c4d-1
  local-address CUSTOMER_IP
  pre-shared-key address 205.251.233.122 key PASSWORD
exit

! An ISAKMP profile is used to associate the keyring with the particular 
! endpoint.
!
crypto isakmp profile isakmp-vpn-1a2b3c4d-1
  local-address CUSTOMER_IP
  match identity address 205.251.233.122
  keyring keyring-vpn-1a2b3c4d-1
exit

! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
!
crypto ipsec transform-set ipsec-prop-vpn-1a2b3c4d-1 esp-aes 128 esp-sha-hmac 
  mode tunnel
exit

! The IPSec profile references the IPSec transform set and further defines
! the Diffie-Hellman group and security association lifetime.
!
crypto ipsec profile ipsec-vpn-1a2b3c4d-1
  set pfs group2
  set security-association lifetime seconds 3600
  set transform-set ipsec-prop-vpn-1a2b3c4d-1
exit

! Additional parameters of the IPSec configuration are set here. Note that 
! these parameters are global and therefore impact other IPSec 
! associations.
! This option instructs the router to clear the "Don't Fragment" 
! bit from packets that carry this bit and yet must be fragmented, enabling
! them to be fragmented.
!
crypto ipsec df-bit clear

! This option enables IPSec Dead Peer Detection, which causes periodic
! messages to be sent to ensure a Security Association remains operational.
!
crypto isakmp keepalive 10 10 on-demand

! This configures the gateway's window for accepting out of order
! IPSec packets. A larger window can be helpful if too many packets 
! are dropped due to reordering while in transit between gateways.
!
crypto ipsec security-association replay window-size 128

! This option instructs the router to fragment the unencrypted packets
! (prior to encryption).
!
crypto ipsec fragmentation before-encryption
  

! --------------------------------------------------------------------------------
! #3: Tunnel Interface Configuration
!  
! A tunnel interface is configured to be the logical interface associated  
! with the tunnel. All traffic routed to the tunnel interface will be 
! encrypted and transmitted to the VPC. Similarly, traffic from the VPC
! will be logically received on this interface.
!
! Association with the IPSec security association is done through the 
! "tunnel protection" command.
!
! The address of the interface is configured with the setup for your 
! Customer Gateway.  If the address changes, the Customer Gateway and VPN 
! Connection must be recreated with Amazon VPC.
!
interface Tunnel2
  ip address 169.254.249.22 255.255.255.252
  ip virtual-reassembly
  tunnel source CUSTOMER_IP
  tunnel destination 205.251.233.122
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile ipsec-vpn-1a2b3c4d-1
  ! This option causes the router to reduce the Maximum Segment Size of
  ! TCP packets to prevent packet fragmentation.
  ip tcp adjust-mss 1387 
  no shutdown
exit

! ----------------------------------------------------------------------------
! #4 Static Route Configuration
!
! Your Customer Gateway needs to set a static route for the prefix corresponding to your 
! VPC to send traffic over the tunnel interface.
! An example for a VPC with the prefix 10.0.0.0/16 is provided below:
! ip route 10.0.0.0 255.255.0.0 Tunnel2 track 200 
!
! SLA Monitor is used to provide a failover between the two tunnels. If the primary tunnel fails, the redundant tunnel will automatically be used
! This sla is defined as #200, which may conflict with an existing sla using same number. 
! If so, we recommend changing the sequence number to avoid conflicts.
!
ip sla 200
   icmp-echo 169.254.249.21 source-interface Tunnel2
   timeout 1000
   frequency 5
exit
ip sla schedule 200  life forever start-time now
track 200 ip sla 200 reachability 
! --------------------------------------------------------------------------------
```

## How to Test the Customer Gateway Configuration<a name="TestCustomerGateway_NoBGP"></a>

You can test the gateway configuration for each tunnel\.

**To test the customer gateway device configuration for each tunnel**

1. Ensure that the customer gateway device has a static route to your VPC, as suggested in the configuration templates provided by AWS\.

1. Ensure that a static route has been added to the VPN connection so that traffic can get back to your customer gateway device\. For example, if your local subnet prefix is `198.10.0.0/16`, you need to add a static route with that CIDR range to your VPN connection\. Make sure that both tunnels have a static route to your VPC\.

Next you must test the connectivity for each tunnel by launching an instance into your VPC, and pinging the instance from your home network\. Before you begin, make sure of the following:
+ Use an AMI that responds to ping requests\. We recommend that you use one of the Amazon Linux AMIs\.
+ Configure your instance's security group and network ACL to enable inbound ICMP traffic\.
+ Ensure that you have configured routing for your VPN connection \- your subnet's route table must contain a route to the virtual private gateway\. For more information, see [Enable Route Propagation in Your Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_VPN.html#vpn-configure-routing) in the *Amazon VPC User Guide*\.

**To test the end\-to\-end connectivity of each tunnel**

1. Launch an instance of one of the Amazon Linux AMIs into your VPC\. The Amazon Linux AMIs are listed in the launch wizard when you launch an instance from the AWS Management Console\. For more information, see the [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/)\.

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

If your tunnels don't test successfully, see [Troubleshooting Cisco IOS Customer Gateway without Border Gateway Protocol Connectivity](Cisco_Troubleshooting_NoBGP.md)\.