# Example: Yamaha Device<a name="Yamaha"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

In this section, we walk you through an example of the configuration information provided by your integration team if your customer gateway device is a Yamaha RT107e, RTX1200, RTX1210, RTX1500, RTX3000, or SRT100 router\.

Two diagrams illustrate the example configuration\. The first diagram shows the high\-level layout of the customer gateway device, and the second diagram shows the details of the example configuration\. You should use the real configuration information that you receive from your integration team and apply it to your customer gateway device\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#CustomerGateway4)
+ [A Detailed View of the Customer Gateway Device and an Example Configuration](#CustomerGatewayDetailedView4)
+ [How to Test the Customer Gateway Configuration](#TestingConfig4)

## A High\-Level View of the Customer Gateway Device<a name="CustomerGateway4"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in case a device fails\.

![\[Yamaha high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-yamaha-diagram.png)

## A Detailed View of the Customer Gateway Device and an Example Configuration<a name="CustomerGatewayDetailedView4"></a>

The diagram in this section illustrates an example Yamaha customer gateway device\. Following the diagram, there is a corresponding example of the configuration information your integration team should provide\. The example configuration contains a set of information for each of the tunnels that you must configure\.

In addition, the example configuration refers to these items that you must provide:
+ *YOUR\_UPLINK\_ADDRESS*—The IP address for the Internet\-routable external interface on the customer gateway device\. The address must be static, and may be behind a device performing network address translation \(NAT\)\. To ensure that NAT traversal \(NAT\-T\) can function, you must adjust your firewall rules to unblock UDP port 4500\.
+ *YOUR\_LOCAL\_NETWORK\_ADDRESS*—The IP address that is assigned to the LAN interface connected to your local network \(most likely a private address such as 192\.168\.0\.1\)
+ *YOUR\_BGP\_ASN*—The customer gateway's BGP ASN \(we use 65000 by default\)

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-44a8938f\), virtual private gateway ID \(vgw\-8db04f81\), the IP addresses \(72\.21\.209\.\*, 169\.254\.255\.\*\), and the remote ASN \(7224\)\. Replace these example values with the actual values from the configuration information that you receive\.

In addition, you must also:
+ Configure the outside interface \(referred to as *LAN3* in the example configuration\)\.
+ Configure the tunnel interface IDs \(referred to as *Tunnel \#1* and *Tunnel \#2* in the example configuration\)\.
+ Configure all internal routing that moves traffic between the customer gateway and your local network\.

In the following diagram and example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

![\[Yamaha detailed diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/detailed-yamaha-diagram.png)

**Warning**  
The following configuration information is an example of what you can expect your integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here\. Otherwise, your implementation will fail\.

```
# Amazon Web Services 
# Virtual Private Cloud 
    
# AWS utilizes unique identifiers to manage the configuration of  
# a VPN Connection. Each VPN Connection is assigned an identifier and is  
# associated with two other identifiers, namely the  
# Customer Gateway Identifier and Virtual Private Gateway Identifier. 
# 
# Your VPN Connection ID            : vpn-44a8938f 
# Your Virtual Private Gateway ID   : vgw-8db04f81 
# Your Customer Gateway ID          : cgw-b4dc3961 
# 
# 
# This configuration consists of two tunnels. Both tunnels must be  
# configured on your Customer Gateway. 
# 
# -------------------------------------------------------------------------------- 
# IPsec Tunnel #1 
# -------------------------------------------------------------------------------- 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
   
# #1: Internet Key Exchange (IKE) Configuration 
# 
# A policy is established for the supported ISAKMP encryption,  
# authentication, Diffie-Hellman, lifetime, and key parameters. 
# 
# Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
# You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24. 
# The address of the external interface for your customer gateway must be a static address. 
# Your customer gateway may reside behind a device performing network address translation (NAT).
# To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall !rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
#
tunnel select 1  
ipsec ike encryption 1 aes-cbc 
ipsec ike group 1 modp1024 
ipsec ike hash 1 sha 
   
# This line stores the Pre Shared Key used to authenticate the 
# tunnel endpoints.
# 
ipsec ike pre-shared-key 1 text plain-text-password1 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
        
# #2: IPsec Configuration 
   
# The IPsec policy defines the encryption, authentication, and IPsec 
# mode parameters. 
# Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
#
# Note that there are a global list of IPSec policies, each identified by 
# sequence number. This policy is defined as #201, which may conflict with
# an existing policy using the same number. If so, we recommend changing 
# the sequence number to avoid conflicts.
#
   
ipsec tunnel 201 
ipsec sa policy 201 1 esp aes-cbc  sha-hmac 
    
# The IPsec profile references the IPsec policy and further defines 
# the Diffie-Hellman group and security association lifetime. 
   
ipsec ike duration ipsec-sa 1 3600 
ipsec ike pfs 1 on 
   
# Additional parameters of the IPsec configuration are set here. Note that  
# these parameters are global and therefore impact other IPsec  
# associations. 
# This option instructs the router to clear the "Don't Fragment"  
# bit from packets that carry this bit and yet must be fragmented, enabling 
# them to be fragmented. 
# 
ipsec tunnel outer df-bit clear 
   
# This option enables IPsec Dead Peer Detection, which causes periodic 
# messages to be sent to ensure a Security Association remains operational. 
   
ipsec ike keepalive use 1 on dpd 10 3 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png) 
        
# -------------------------------------------------------------------------------- 
# #3: Tunnel Interface Configuration 
#   
# A tunnel interface is configured to be the logical interface associated   
# with the tunnel. All traffic routed to the tunnel interface will be  
# encrypted and transmitted to the VPC. Similarly, traffic from the VPC 
# will be logically received on this interface. 
# 
# 
# The address of the interface is configured with the setup for your  
# Customer Gateway.  If the address changes, the Customer Gateway and VPN  
# Connection must be recreated with Amazon VPC. 
# 
ipsec ike local address 1 YOUR_LOCAL_NETWORK_ADDRESS 
ipsec ike remote address 1 72.21.209.225
ipsec ike local address id <id> 0.0.0.0/0
ipsec ike remote address id <id> 0.0.0.0/0
ip tunnel address 169.254.255.2/30 
ip tunnel remote address 169.254.255.1 
   
# This option causes the router to reduce the Maximum Segment Size of 
# TCP packets to prevent packet fragmentation 
   
ip tunnel tcp mss limit 1387
tunnel enable 1
tunnel select none
ipsec auto refresh on 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)
        
# -------------------------------------------------------------------------------- 
# #4: Border Gateway Protocol (BGP) Configuration 
#                                                                                      
# BGP is used within the tunnel to exchange prefixes between the 
# Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway     
# will announce the prefix corresponding to your VPC. 
#  
# Your Customer Gateway may announce a default route (0.0.0.0/0), 
# which can be done with the 'network' and 'default-originate' statements.
#           
# The BGP timers are adjusted to provide more rapid detection of outages. 
# 
# The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN) is configured 
# as part of your Customer Gateway. If the ASN must be changed, the  
# Customer Gateway and VPN Connection will need to be recreated with AWS. 
# 
bgp use on
bgp autonomous-system YOUR_BGP_ASN
bgp neighbor 1 7224 169.254.255.1 hold-time=30 local-address=169.254.255.2

# To advertise additional prefixes to Amazon VPC, copy the 'network' statement and 
# identify the prefix you wish to advertise. Make sure the 
# prefix is present in the routing table of the device with a valid next-hop.
# For example, the following two lines will advertise 192.168.0.0/16 and 10.0.0.0/16 to Amazon VPC
#
# bgp import filter 1 equal 10.0.0.0/16
# bgp import filter 1 equal 192.168.0.0/16
#

bgp import filter 1 equal 0.0.0.0/0
bgp import 7224 static filter 1

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png) 
        
# -------------------------------------------------------------------------------- 
# IPsec Tunnel #2 
# -------------------------------------------------------------------------------- 


# #1: Internet Key Exchange (IKE) Configuration 
# 
# A policy is established for the supported ISAKMP encryption,  
# authentication, Diffie-Hellman, lifetime, and key parameters. 
# 
# Please note, these sample configurations are for the minimum requirement of AES128, SHA1, and DH Group 2.
# You will need to modify these sample configuration files to take advantage of AES256, SHA256, or other DH groups like 2, 14-18, 22, 23, and 24. 
# The address of the external interface for your customer gateway must be a static address. 
# Your customer gateway may reside behind a device performing network address translation (NAT).
# To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
#
tunnel select 2 
ipsec ike encryption 2 aes-cbc 
ipsec ike group 2 modp1024 
ipsec ike hash 2 sha 

# This line stores the Pre Shared Key used to authenticate the 
# tunnel endpoints.
# 
ipsec ike pre-shared-key 2 text plain-text-password2

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png) 
        
# #2: IPsec Configuration 

# The IPsec policy defines the encryption, authentication, and IPsec 
# mode parameters. 
# Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.
#
# Note that there are a global list of IPsec policies, each identified by  
# sequence number. This policy is defined as #202, which may conflict with 
# an existing policy using the same number. If so, we recommend changing  
# the sequence number to avoid conflicts. 
# 

ipsec tunnel 202
ipsec sa policy 202 2 esp aes-cbc  sha-hmac

# The IPsec profile references the IPsec policy and further defines 
# the Diffie-Hellman group and security association lifetime. 

ipsec ike duration ipsec-sa 2 3600
ipsec ike pfs 2 on

# Additional parameters of the IPsec configuration are set here. Note that  
# these parameters are global and therefore impact other IPsec  
# associations. 
# This option instructs the router to clear the "Don't Fragment"  
# bit from packets that carry this bit and yet must be fragmented, enabling 
# them to be fragmented. 
# 
ipsec tunnel outer df-bit clear

# This option enables IPsec Dead Peer Detection, which causes periodic 
# messages to be sent to ensure a Security Association remains operational. 

ipsec ike keepalive use 2 on dpd 10 3

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
        
# -------------------------------------------------------------------------------- 
# #3: Tunnel Interface Configuration 
#  
# A tunnel interface is configured to be the logical interface associated   
# with the tunnel. All traffic routed to the tunnel interface will be  
# encrypted and transmitted to the VPC. Similarly, traffic from the VPC 
# will be logically received on this interface. 
# 
# Association with the IPsec security association is done through the  
# "tunnel protection" command. 
# 
# The address of the interface is configured with the setup for your  
# Customer Gateway.  If the address changes, the Customer Gateway and VPN  
# Connection must be recreated with Amazon VPC. 
# 
ipsec ike local address 2 YOUR_LOCAL_NETWORK_ADDRESS
ipsec ike remote address 2 72.21.209.193
ipsec ike local address id <id> 0.0.0.0/0
ipsec ike remote address id <id> 0.0.0.0/0
ip tunnel address 169.254.255.6/30 
ip tunnel remote address 169.254.255.5 

# This option causes the router to reduce the Maximum Segment Size of 
# TCP packets to prevent packet fragmentation 

ip tunnel tcp mss limit 1387
tunnel enable 2
tunnel select none
ipsec auto refresh on

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)
        
# -------------------------------------------------------------------------------- 
# #4: Border Gateway Protocol (BGP) Configuration 
#                                                                                     
# BGP is used within the tunnel to exchange prefixes between the 
# Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway     
# will announce the prefix corresponding to your VPC. 
#            
# Your Customer Gateway may announce a default route (0.0.0.0/0), 
# which can be done with the 'network' and 'default-originate' statements.
#
#            
# The BGP timers are adjusted to provide more rapid detection of outages. 
# 
# The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN) is configured 
# as part of your Customer Gateway. If the ASN must be changed, the  
# Customer Gateway and VPN Connection will need to be recreated with AWS. 
# 
bgp use on
bgp autonomous-system YOUR_BGP_ASN
bgp neighbor 2 7224 169.254.255.5 hold-time=30 local-address=169.254.255.6

# To advertise additional prefixes to Amazon VPC, copy the 'network' statement and 
# identify the prefix you wish to advertise. Make sure the 
# prefix is present in the routing table of the device with a valid next-hop.
# For example, the following two lines will advertise 192.168.0.0/16 and 10.0.0.0/16 to Amazon VPC
#
# bgp import filter 1 equal 10.0.0.0/16
# bgp import filter 1 equal 192.168.0.0/16
#

bgp import filter 1 equal 0.0.0.0/0
bgp import 7224 static filter 1

bgp configure refresh
```

## How to Test the Customer Gateway Configuration<a name="TestingConfig4"></a>

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

If your tunnels don't test successfully, see [Troubleshooting Yamaha Customer Gateway Connectivity](Yamaha_Troubleshooting.md)\.