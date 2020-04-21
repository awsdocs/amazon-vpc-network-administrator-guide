# Example: Cisco ASA Device with a Virtual Tunnel Interface and Border Gateway Protocol<a name="cisco-asa-vti-bgp"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

In this section, you get an example of the configuration information provided by your integration team if your customer gateway is a Cisco ASA device running Cisco ASA 9\.7\.1\+ software\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway](#cisco-asa-vti-bgp-overview)
+ [Example Configuration](#cisco-asa-vti-bgp-details)
+ [How to Test the Customer Gateway Configuration](#cisco-asa-vti-bgp-test)

## A High\-Level View of the Customer Gateway<a name="cisco-asa-vti-bgp-overview"></a>

The following diagram shows the general details of your customer gateway\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Cisco ASA high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-diagram.png)

Cisco ASAs from version 9\.7\.1 and later support Active/Active mode\. When you use these Cisco ASAs, you can have both tunnels active at the same time\. With this redundancy, you should always have connectivity to your VPC through one of the tunnels\.

## Example Configuration<a name="cisco-asa-vti-bgp-details"></a>

The configuration in this section is an example of the configuration information your integration team should provide\. The example configuration contains a set of information for each of the tunnels that you must configure\.

The example configuration includes example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-12345678\) and virtual private gateway ID \(vgw\-12345678\), and placeholders for the AWS endpoints \(*AWS\_ENDPOINT\_1* and *AWS\_ENDPOINT\_2*\)\. Replace these example values with the actual values from the configuration information that you receive\.

In addition, you must do the following:
+ Configure the outside interface\.
+ Ensure that the Crypto ISAKMP Policy Sequence number is unique\.
+ Ensure that the Crypto IPsec Transform Set and the Crypto ISAKMP Policy Sequence are harmonious with any other IPsec tunnels configured on the device\.
+ Configure all internal routing that moves traffic between the customer gateway and your local network\.

**Important**  
The following configuration information is an example of what you can expect your integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
! Amazon Web Services
! Virtual Private Cloud

! AWS utilizes unique identifiers to manipulate the configuration of
! a VPN Connection. Each VPN Connection is assigned an identifier and is
! associated with two other identifiers, namely the
! Customer Gateway Identifier and Virtual Private Gateway Identifier.
!
! Your VPN Connection ID                  : vpn-12345678
! Your Virtual Private Gateway ID         : vgw-12345678
! Your Customer Gateway ID		    : cgw-12345678
!
!
! This configuration consists of two tunnels. Both tunnels must be
! configured on your Customer Gateway.
!

! -------------------------------------------------------------------------
! IPSec Tunnel #1
! -------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration
!
! A policy is established for the supported ISAKMP encryption,
! authentication, Diffie-Hellman, lifetime, and key parameters.
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address.
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall !rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
! Note that there are a global list of ISAKMP policies, each identified by
! sequence number. This policy is defined as #200, which may conflict with
! an existing policy using the same number. If so, we recommend changing
! the sequence number to avoid conflicts.
!

crypto ikev1 enable 'outside_interface' 

crypto ikev1 policy 200
  encryption aes 
  authentication pre-share
  group 2
  lifetime 28800
  hash sha
		
! -------------------------------------------------------------------------		
! #2: IPSec Configuration
!
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
!
crypto ipsec ikev1 transform-set ipsec-prop-vpn-12345678-0 esp-aes  esp-sha-hmac


! The IPSec profile references the IPSec transform set and further defines
! the Diffie-Hellman group and security association lifetime.
!
crypto ipsec profile ipsec-vpn-12345678-0
  set pfs group2
  set security-association lifetime seconds 3600
  set ikev1 transform-set ipsec-prop-vpn-12345678-0
exit

! Additional parameters of the IPSec configuration are set here. Note that
! these parameters are global and therefore impact other IPSec
! associations.
! This option instructs the router to clear the "Don't Fragment"
! bit from packets that carry this bit and yet must be fragmented, enabling
! them to be fragmented. 
!
!You will need to replace the outside_interface with the interface name of your ASA Firewall.

crypto ipsec df-bit clear-df 'outside_interface'



! This option causes the firewall to reduce the Maximum Segment Size of
! TCP packets to prevent packet fragmentation.

sysopt connection tcpmss 1379


! This configures the gateway's window for accepting out of order
! IPSec packets. A larger window can be helpful if too many packets
! are dropped due to reordering while in transit between gateways.
!
crypto ipsec security-association replay window-size 128

! This option instructs the router to fragment the unencrypted packets
! (prior to encryption).
!You will need to replace the outside_interface with the interface name of your ASA Firewall.
!
crypto ipsec fragmentation before-encryption 'outside_interface'



! -------------------------------------------------------------------------


! The tunnel group sets the Pre Shared Key used to authenticate the 
! tunnel endpoints.
!
tunnel-group 13.54.43.86 type ipsec-l2l
tunnel-group 13.54.43.86 ipsec-attributes
   ikev1 pre-shared-key pre-shared-key
!
! This option enables IPSec Dead Peer Detection, which causes semi-periodic
! messages to be sent to ensure a Security Association remains operational.
!
   isakmp keepalive threshold 10 retry 10
exit

! -------------------------------------------------------------------------
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
!You will need to replace the outside_interface with the interface name of your ASA Firewall.



interface Tunnel1
  nameif Tunnel-int-vpn-12345678-0		
  ip address 169.254.33.198 255.255.255.252
  tunnel source interface 'outside_interface'
  tunnel destination 13.54.43.86
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile ipsec-vpn-12345678-0
  no shutdown
exit

! -------------------------------------------------------------------------

! #4: Border Gateway Protocol (BGP) Configuration
!
! BGP is used within the tunnel to exchange prefixes between the
! Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway
! will announce the prefix corresponding to your VPC.
!
! Your Customer Gateway may announce a default route (0.0.0.0/0),
! which can be done with the 'network' and 'default-originate' statements.
!
! The BGP timers are adjusted to provide more rapid detection of outages.
!
! The local BGP Autonomous System Number (ASN) (65343) is configured
! as part of your Customer Gateway. If the ASN must be changed, the
! Customer Gateway and VPN Connection will need to be recreated with AWS.
!
router bgp 65343
  address-family ipv4 unicast
    neighbor 169.254.33.197 remote-as 7224
    neighbor 169.254.33.197 timers 10 30 30
    neighbor 169.254.33.197 default-originate
    neighbor 169.254.33.197 activate
    
! To advertise additional prefixes to Amazon VPC, copy the 'network' statement
! and identify the prefix you wish to advertise. Make sure the prefix is present
! in the routing table of the device with a valid next-hop.
    network 0.0.0.0
    no auto-summary
no synchronization
  exit-address-family
exit
!

! -------------------------------------------------------------------------
! IPSec Tunnel #2
! -------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration
!
! A policy is established for the supported ISAKMP encryption,
! authentication, Diffie-Hellman, lifetime, and key parameters.
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address.
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall !rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
! Note that there are a global list of ISAKMP policies, each identified by
! sequence number. This policy is defined as #201, which may conflict with
! an existing policy using the same number. If so, we recommend changing
! the sequence number to avoid conflicts.
!

crypto ikev1 enable 'outside_interface' 

crypto ikev1 policy 201
  encryption aes 
  authentication pre-share
  group 2
  lifetime 28800
  hash sha
		
! -------------------------------------------------------------------------	
! #2: IPSec Configuration
!
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
!
crypto ipsec ikev1 transform-set ipsec-prop-vpn-12345678-1 esp-aes  esp-sha-hmac


! The IPSec profile references the IPSec transform set and further defines
! the Diffie-Hellman group and security association lifetime.
!
crypto ipsec profile ipsec-vpn-12345678-1
  set pfs group2
  set security-association lifetime seconds 3600
  set ikev1 transform-set ipsec-prop-vpn-12345678-1
exit						

! Additional parameters of the IPSec configuration are set here. Note that
! these parameters are global and therefore impact other IPSec
! associations.
! This option instructs the router to clear the "Don't Fragment"
! bit from packets that carry this bit and yet must be fragmented, enabling
! them to be fragmented. 
!
!You will need to replace the outside_interface with the interface name of your ASA Firewall.

crypto ipsec df-bit clear-df 'outside_interface'



! This option causes the firewall to reduce the Maximum Segment Size of
! TCP packets to prevent packet fragmentation.

sysopt connection tcpmss 1379


! This configures the gateway's window for accepting out of order
! IPSec packets. A larger window can be helpful if too many packets
! are dropped due to reordering while in transit between gateways.
!
crypto ipsec security-association replay window-size 128

! This option instructs the router to fragment the unencrypted packets
! (prior to encryption).
!You will need to replace the outside_interface with the interface name of your ASA Firewall.
!
crypto ipsec fragmentation before-encryption 'outside_interface'



! -------------------------------------------------------------------------


! The tunnel group sets the Pre Shared Key used to authenticate the 
! tunnel endpoints.
!
tunnel-group 52.65.137.78 type ipsec-l2l
tunnel-group 52.65.137.78 ipsec-attributes
   ikev1 pre-shared-key pre-shared-key
!
! This option enables IPSec Dead Peer Detection, which causes semi-periodic
! messages to be sent to ensure a Security Association remains operational.
!
   isakmp keepalive threshold 10 retry 10
exit

! -------------------------------------------------------------------------
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
!You will need to replace the outside_interface with the interface name of your ASA Firewall.



interface Tunnel2
  nameif Tunnel-int-vpn-12345678-1		
  ip address 169.254.33.194 255.255.255.252
  tunnel source interface 'outside_interface'
  tunnel destination 52.65.137.78
  tunnel mode ipsec ipv4
  tunnel protection ipsec profile ipsec-vpn-12345678-1
  no shutdown
exit

! -------------------------------------------------------------------------

! #4: Border Gateway Protocol (BGP) Configuration
!
! BGP is used within the tunnel to exchange prefixes between the
! Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway
! will announce the prefix corresponding to your VPC.
!
! Your Customer Gateway may announce a default route (0.0.0.0/0),
! which can be done with the 'network' and 'default-originate' statements.
!
! The BGP timers are adjusted to provide more rapid detection of outages.
!
! The local BGP Autonomous System Number (ASN) (65343) is configured
! as part of your Customer Gateway. If the ASN must be changed, the
! Customer Gateway and VPN Connection will need to be recreated with AWS.
!
router bgp 65343
  address-family ipv4 unicast
    neighbor 169.254.33.193 remote-as 7224
    neighbor 169.254.33.193 timers 10 30 30
    neighbor 169.254.33.193 default-originate
    neighbor 169.254.33.193 activate
    
! To advertise additional prefixes to Amazon VPC, copy the 'network' statement
! and identify the prefix you wish to advertise. Make sure the prefix is present
! in the routing table of the device with a valid next-hop.
    network 0.0.0.0
    no auto-summary
    no synchronization
  exit-address-family
exit
!
```

## How to Test the Customer Gateway Configuration<a name="cisco-asa-vti-bgp-test"></a>

When using Cisco ASA as a customer gateway in routed mode, both tunnels will be in the UP state\.

You can test the gateway configuration for each tunnel\.

**To test the customer gateway configuration for each tunnel**
+ Ensure that routes are advertised with BGP correctly and showing in routing table so that traffic can get back to your customer gateway\. For example, if your local subnet prefix is `198.10.0.0/16`, you must advertise it through BGP\. Make sure that both tunnels are configured with BGP routing\.

Next you must test the connectivity for each tunnel by launching an instance into your VPC, and pinging the instance from your home network\. Before you begin, make sure of the following:
+ Use an AMI that responds to ping requests\. We recommend that you use one of the Amazon Linux AMIs\.
+ Configure your instance's security group and network ACL to enable inbound ICMP traffic\.
+ Ensure that you have configured routing for your VPN connection \- your subnet's route table must contain a route to the virtual private gateway\. For more information, see [Enable Route Propagation in Your Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_VPN.html#vpn-configure-routing) in the *Amazon VPC User Guide*\.

**To test the end\-to\-end connectivity of each tunnel**

1. Launch an instance of one of the Amazon Linux AMIs into your VPC\. The Amazon Linux AMIs are listed in the launch wizard when you launch an instance from the AWS Management Console\. For more information, see the [Amazon VPC Getting Started Guide](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/)\.

1. After the instance is running, get its private IP address \(for example, `10.0.0.4`\)\. The console displays the address as part of the instance's details\.

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

If your tunnels do not test successfully, see [Troubleshooting Cisco ASA Customer Gateway Connectivity](Cisco_ASA_Troubleshooting.md)\.