# Example: Cisco ASA Device<a name="Cisco_ASA"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

In this section, you get an example of the configuration information provided by your integration team if your customer gateway is a Cisco ASA device running Cisco ASA 8\.2\+ software\.

The diagram shows the high\-level layout of the customer gateway\. You should use the real configuration information that you receive from your integration team and apply it to your customer gateway\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway](#Cisco_ASA_overview)
+ [An Example Configuration](#Cisco_ASA_details)
+ [How to Test the Customer Gateway Configuration](#TestCustomerGateway_ASA)

## A High\-Level View of the Customer Gateway<a name="Cisco_ASA_overview"></a>

The following diagram shows the general details of your customer gateway\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Cisco ASA high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-cisco-asa-diagram.png)

Please note that some Cisco ASAs only support Active/Standby mode\. When you use these Cisco ASAs, you can have only one active tunnel at a time\. The other standby tunnel becomes active if the first tunnel becomes unavailable\. With this redundancy, you should always have connectivity to your VPC through one of the tunnels\.

## An Example Configuration<a name="Cisco_ASA_details"></a>

 The configuration in this section is an example of the configuration information your integration team should provide\. The example configuration contains a set of information for each of the tunnels that you must configure\.

The example configuration includes example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-12345678\) and virtual private gateway ID \(vgw\-12345678\), and placeholders for the AWS endpoints \(*AWS\_ENDPOINT\_1* and *AWS\_ENDPOINT\_2*\)\. Replace these example values with the actual values from the configuration information that you receive\.

In addition, you must:
+ Configure the outside interface\.
+ Ensure that the Crypto ISAKMP Policy Sequence number is unique\.
+ Ensure that the Crypto List Policy Sequence number is unique\.
+ Ensure that the Crypto IPsec Transform Set and the Crypto ISAKMP Policy Sequence are harmonious with any other IPsec tunnels configured on the device\.
+ Ensure that the SLA monitoring number is unique\.
+ Configure all internal routing that moves traffic between the customer gateway and your local network\.

**Important**  
The following configuration information is an example of what you can expect your integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
! Amazon Web Services
! Virtual Private Cloud
!
! AWS utilizes unique identifiers to manipulate the configuration of 
! a VPN Connection. Each VPN Connection is assigned an identifier and is 
! associated with two other identifiers, namely the 
! Customer Gateway Identifier and Virtual Private Gateway Identifier.
!
! Your VPN Connection ID                  : vpn-12345678
! Your Virtual Private Gateway ID         : vgw-12345678
! Your Customer Gateway ID                : cgw-12345678
!
! This configuration consists of two tunnels. Both tunnels must be 
! configured on your Customer Gateway. Only a single tunnel will be up at a
! time to the VGW.
! 
! You may need to populate these values throughout the config based on your setup:
! outside_interface - External interface of the ASA
! outside_access_in - Inbound ACL on the external interface
! amzn_vpn_map - Outside crypto map
! vpc_subnet and vpc_subnet_mask - VPC address range
! local_subnet and local_subnet_mask - Local subnet address range
! sla_monitor_address - Target address that is part of acl-amzn to run SLA monitoring
!
! --------------------------------------------------------------------------------
! IPSec Tunnels
! --------------------------------------------------------------------------------
! #1: Internet Key Exchange (IKE) Configuration
!
! A policy is established for the supported ISAKMP encryption, 
! authentication, Diffie-Hellman, lifetime, and key parameters.
!
! Note that there are a global list of ISAKMP policies, each identified by 
! sequence number. This policy is defined as #201, which may conflict with
! an existing policy using the same or lower number depending on 
! the encryption type. If so, we recommend changing the sequence number to 
! avoid conflicts and overlap.
!
! Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
! You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24.
! The address of the external interface for your customer gateway must be a static address. 
! Your customer gateway may reside behind a device performing network address translation (NAT).
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!
crypto isakmp identity address 
crypto isakmp enable outside_interface
crypto isakmp policy 201
  encryption aes
  authentication pre-share
  group 2
  lifetime 28800
  hash sha
exit
!
! The tunnel group sets the Pre Shared Key used to authenticate the 
! tunnel endpoints.
!
tunnel-group AWS_ENDPOINT_1 type ipsec-l2l
tunnel-group AWS_ENDPOINT_1 ipsec-attributes
   pre-shared-key password_here
!
! This option enables IPSec Dead Peer Detection, which causes periodic
! messages to be sent to ensure a Security Association remains operational.
!
   isakmp keepalive threshold 10 retry 10
exit
!
tunnel-group AWS_ENDPOINT_2 type ipsec-l2l
tunnel-group AWS_ENDPOINT_2 ipsec-attributes
   pre-shared-key password_here
!
! This option enables IPSec Dead Peer Detection, which causes periodic
! messages to be sent to ensure a Security Association remains operational.
!
   isakmp keepalive threshold 10 retry 10
exit

! --------------------------------------------------------------------------------
! #2: Access List Configuration
!
! Access lists are configured to permit creation of tunnels and to send applicable traffic over them.
! This policy may need to be applied to an inbound ACL on the outside interface that is used to manage control-plane traffic. 
! This is to allow VPN traffic into the device from the Amazon endpoints.
!
access-list outside_access_in extended permit ip host AWS_ENDPOINT_1 host YOUR_UPLINK_ADDRESS
access-list outside_access_in extended permit ip host AWS_ENDPOINT_2 host YOUR_UPLINK_ADDRESS
!
! The following access list named acl-amzn specifies all traffic that needs to be routed to the VPC. Traffic will
! be encrypted and transmitted through the tunnel to the VPC. Association with the IPSec security association
! is done through the "crypto map" command.
!
! This access list should contain a static route corresponding to your VPC CIDR and allow traffic from any subnet.
! If you do not wish to use the "any" source, you must use a single access-list entry for accessing the VPC range.
! If you specify more than one entry for this ACL without using "any" as the source, the VPN will function erratically.
! The any rule is also used so the security association will include the ASA outside interface where the SLA monitor
! traffic will be sourced from.
! See section #4 regarding how to restrict the traffic going over the tunnel
!
!
access-list acl-amzn extended permit ip any vpc_subnet vpc_subnet_mask

!---------------------------------------------------------------------------------
! #3: IPSec Configuration
!
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.  
!
crypto ipsec ikev1 transform-set transform-amzn esp-aes esp-sha-hmac

! The crypto map references the IPSec transform set and further defines
! the Diffie-Hellman group and security association lifetime. The mapping is created
! as #1, which may conflict with an existing crypto map using the same
! number. If so, we recommend changing the mapping number to avoid conflicts.
!
crypto map amzn_vpn_map 1 match address acl-amzn
crypto map amzn_vpn_map 1 set pfs group2
crypto map amzn_vpn_map 1 set peer AWS_ENDPOINT_1 AWS_ENDPOINT_2
crypto map amzn_vpn_map 1 set transform-set transform-amzn
crypto map amzn_vpn_map 1 set security-association lifetime seconds 3600
!
! Only set this if you do not already have an outside crypto map, and it is not applied:
!
crypto map amzn_vpn_map interface outside_interface
!
! Additional parameters of the IPSec configuration are set here. Note that
! these parameters are global and therefore impact other IPSec
! associations.
!
! This option instructs the firewall to clear the "Don't Fragment"
! bit from packets that carry this bit and yet must be fragmented, enabling
! them to be fragmented.
!
crypto ipsec df-bit clear-df outside_interface
!
! This configures the gateway's window for accepting out of order
! IPSec packets. A larger window can be helpful if too many packets
! are dropped due to reordering while in transit between gateways.
!
crypto ipsec security-association replay window-size 128
!
! This option instructs the firewall to fragment the unencrypted packets
! (prior to encryption).
!
crypto ipsec fragmentation before-encryption outside_interface
!
! This option causes the firewall to reduce the Maximum Segment Size of
! TCP packets to prevent packet fragmentation.
sysopt connection tcpmss 1379
!
! In order to keep the tunnel in an active or always up state, the ASA needs to send traffic to the subnet
! defined in acl-amzn. SLA monitoring can be configured to send pings to a destination in the subnet and
! will keep the tunnel active. This traffic needs to be sent to a target that will return a response.
! This can be manually tested by sending a ping to the target from the ASA sourced from the outside interface.
! A possible destination for the ping is an instance within the VPC. For redundancy multiple SLA monitors 
! can be configured to several instances to protect against a single point of failure.
!
! The monitor is created as #1, which may conflict with an existing monitor using the same
! number. If so, we recommend changing the sequence number to avoid conflicts.
!
sla monitor 1
   type echo protocol ipIcmpEcho sla_monitor_address interface outside_interface
   frequency 5
exit
sla monitor schedule 1 life forever start-time now
!
! The firewall must allow icmp packets to use "sla monitor"
icmp permit any outside_interface

!---------------------------------------------------------------------------------
! #4: VPN Filter
! The VPN Filter will restrict traffic that is permitted through the tunnels. By default all traffic is denied. 
! The first entry provides an example to include traffic between your VPC Address space and your office.
! You may need to run 'clear crypto isakmp sa', in order for the filter to take effect.
!
! access-list amzn-filter extended permit ip vpc_subnet vpc_subnet_mask local_subnet local_subnet_mask
access-list amzn-filter extended deny ip any any
group-policy filter internal
group-policy filter attributes
vpn-filter value amzn-filter
tunnel-group AWS_ENDPOINT_1 general-attributes
default-group-policy filter
exit
tunnel-group AWS_ENDPOINT_2 general-attributes
default-group-policy filter
exit

!---------------------------------------------------------------------------------------
! #5: NAT Exemption
! If you are performing NAT on the ASA you will have to add a nat exemption rule.
! This varies depending on how NAT is set up.  It should be configured along the lines of:
! object network obj-SrcNet
!   subnet 0.0.0.0 0.0.0.0
! object network obj-amzn
!   subnet vpc_subnet vpc_subnet_mask
! nat (inside,outside) 1 source static obj-SrcNet obj-SrcNet destination static obj-amzn obj-amzn
! If using version 8.2 or older, the entry would need to look something like this:
! nat (inside) 0 access-list acl-amzn
! Or, the same rule in acl-amzn should be included in an existing no nat ACL.
```

## How to Test the Customer Gateway Configuration<a name="TestCustomerGateway_ASA"></a>

When using Cisco ASA as a customer gateway, only one tunnel is in the UP state\. The second tunnel should be configured, but is only used if the first tunnel goes down\. The second tunnel cannot be in the UP state when the first tunnel is in the UP state\. Your console displays that only one tunnel is up and shows the second tunnel as down\. This is expected behavior for Cisco ASA customer gateway tunnels because ASA as a customer gateway only supports a single tunnel being up at one time\.

You can test the gateway configuration for each tunnel\.

**To test the customer gateway configuration for each tunnel**
+ Ensure that a static route has been added to the VPN connection so that traffic can get back to your customer gateway\. For example, if your local subnet prefix is `198.10.0.0/16`, add a static route with that CIDR range to your VPN connection\. Make sure that both tunnels have a static route to your VPC\.

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