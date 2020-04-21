# Example: Palo Alto Networks Device<a name="palo-alto"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

The following topic provides example configuration information provided by your integration team if your customer gateway device is a Palo Alto Networks PANOS 4\.1\.2\+ device\.

Two diagrams illustrate the example configuration\. The first diagram shows the high\-level layout of the customer gateway device, and the second diagram shows the details of the example configuration\. You should use the real configuration information that you receive from your integration team and apply it to your customer gateway device\.

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [A High\-Level View of the Customer Gateway Device](#palo-alto-high-level)
+ [A Detailed View of the Customer Gateway Device and an Example Configuration](#palo-alto-detailed-view)
+ [How to Test the Customer Gateway Configuration](#palo-alto-testing)

## A High\-Level View of the Customer Gateway Device<a name="palo-alto-high-level"></a>

The following diagram shows the general details of your customer gateway device\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Customer gateway high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-diagram.png)

## A Detailed View of the Customer Gateway Device and an Example Configuration<a name="palo-alto-detailed-view"></a>

The diagram in this section illustrates an example Palo Alto customer gateway device\. Following the diagram, there is a corresponding example of the configuration information your integration team should provide\. The example configuration contains a set of information for each of the tunnels that you must configure\.

In addition, the example configuration refers to these items that you must provide:
+ *YOUR\_UPLINK\_ADDRESS*—The IP address for the Internet\-routable external interface on the customer gateway device \(which must be static, and may be behind a device performing network address translation \(NAT\); however, NAT traversal \(NAT\-T\) is not supported\)\.
+ *YOUR\_BGP\_ASN*—The customer gateway's BGP ASN \(we use 65000 by default\)

The example configuration includes several example values to help you understand how configuration works\. For example, we provide example values for the VPN connection ID \(vpn\-44a8938f\), virtual private gateway ID \(vgw\-8db04f81\), the IP addresses \(72\.21\.209\.\*, 169\.254\.255\.\*\), and the remote ASN \(7224\)\. Replace these example values with the actual values from the configuration information that you receive\.

In the following diagram and example configuration, you must replace the placeholder values are indicated by colored italic text with values that apply to your particular configuration\.

![\[Palo Alto detailed diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/detailed-palo-alto-diagram.png)

**Warning**  
The following configuration information is an example of what you can expect your integration team to provide\. Many of the values in the following example are different from the actual configuration information that you receive\. You must use the actual values and not the example values shown here, or your implementation will fail\.

```
! Amazon Web Services
! Virtual Private Cloud

! AWS utilizes unique identifiers to manipulate the configuration of 
! a VPN Connection. Each VPN Connection is assigned an identifier and is 
! associated with two other identifiers, namely the 
! Customer Gateway Identifier and Virtual Private Gateway Identifier.
!
! Your VPN Connection ID 		  : vpn-44a8938f
! Your Virtual Private Gateway ID  : vgw-8db04f81
! Your Customer Gateway ID		 : cgw-b4dc3961
!
!
! This configuration consists of two tunnels. Both tunnels must be 
! configured on your Customer Gateway.
!
! --------------------------------------------------------------------------------
! IPSec Tunnel #1
! --------------------------------------------------------------------------------

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)    
   
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

 configure
 edit network ike crypto-profiles ike-crypto-profiles ike-crypto-vpn-44a8938f-0
  set dh-group group2
  set hash sha1
  set lifetime seconds  28800
  set encryption aes128
  top

 edit network ike gateway ike-vpn-44a8938f-0
  set protocol ikev1 dpd interval 10 retry 3 enable yes
  set protocol ikev1 ike-crypto-profile ike-crypto-vpn-44a8938f-0 exchange-mode main
  set authentication pre-shared-key key plain-text-password1
  set local-address ip YOUR_UPLINK_ADDRESS
  set local-address interface ethernet1/1
  set peer-address ip 72.21.209.193 
 top


![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  
! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
!
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.

 edit network ike crypto-profiles ipsec-crypto-profiles ipsec-vpn-44a8938f-0
  set esp authentication sha1
  set esp encryption aes128
  set dh-group group2 lifetime seconds 3600
 top



![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
        
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


 edit network interface tunnel
  set ip 169.254.255.5/30
  set units tunnel.1
  set mtu 1427
 top

 edit network tunnel ipsec ipsec-tunnel-1
  set auto-key ike-gateway ike-vpn-44a8938f-0
  set auto-key ipsec-crypto-profile ipsec-vpn-44a8938f-0
  set tunnel-interface tunnel.1
  set anti-replay yes



![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  
        
! --------------------------------------------------------------------------------

! #4: Border Gateway Protocol (BGP) Configuration
!                                                                                     
! BGP is used within the tunnel to exchange prefixes between the
! Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway    
! will announce the prefix corresponding to your VPC.
!            
!
!
! The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN)
! is configured as part of your Customer Gateway. If the ASN must 
! be changed, the Customer Gateway and VPN Connection will need to be recreated with AWS.
!


 edit network virtual-router default protocol bgp
  set enable yes
  set router-id YOUR_UPLINK_ADDRESS
  set local-as YOUR_BGP_ASN
   edit peer-group AmazonBGP
    edit peer amazon-tunnel-44a8938f-0
     set connection-options keep-alive-interval 10
     set connection-options hold-time 30
     set enable yes
     set local-address ip 169.254.255.5/30
     set local-address interface tunnel.1
     set peer-as 7224
     set peer-address ip 169.254.255.2
     top



! Your Customer Gateway may announce a default route (0.0.0.0/0) to us. 

 edit network virtual-router default protocol bgp policy
  set export rules vr-export action allow 
  set match address-prefix 0.0.0.0/0 exact yes
  set used-by AmazonBGP enable yes
 top
                

! To advertise additional prefixes to Amazon VPC, add these prefixes to the 'address-prefix'
! statement and identify the prefix you wish to advertise. Make sure the prefix is present 
! in the routing table of the device with a valid next-hop. If you want to advertise 
! 192.168.0.0/16 to Amazon, this can be done using the following.

 edit network virtual-router default protocol bgp policy
  set export rules vr-export action allow 
  set match address-prefix 192.168.0.0/16 exact yes
  set used-by AmazonBGP enable yes
 top


!




![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  
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
! To ensure that NAT traversal (NAT-T) can function, you must adjust your firewall !rules to unblock UDP port 4500. If not behind NAT, we recommend disabling NAT-T.
!

 configure
 edit network ike crypto-profiles ike-crypto-profiles ike-crypto-vpn-44a8938f-1
  set dh-group group2
  set hash sha1
  set lifetime seconds  28800
  set encryption aes128
  top

 edit network ike gateway ike-vpn-44a8938f-1
  set protocol ikev1 dpd interval 10 retry 3 enable yes
  set protocol ikev1 ike-crypto-profile ike-crypto-vpn-35a6445c-1 exchange-mode main
  set authentication pre-shared-key key plain-text-password2
  set local-address ip YOUR_UPLINK_ADDRESS
  set local-address interface ethernet1/1
  set peer-address ip 72.21.209.225 
 top




![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  
        
! #2: IPSec Configuration
! 
! The IPSec transform set defines the encryption, authentication, and IPSec
! mode parameters.
!
! Please note, you may use these additionally supported IPSec parameters for encryption like AES256 and other DH groups like 2, 5, 14-18, 22, 23, and 24.

 edit network ike crypto-profiles ipsec-crypto-profiles ipsec-vpn-44a8938f-1
  set esp authentication sha1
  set esp encryption aes128
  set dh-group group2 lifetime seconds 3600
 top



![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  
        
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


 edit network interface tunnel
  set ip 169.254.255.1/30
  set units tunnel.2
  set mtu 1427
 top

 edit network tunnel ipsec ipsec-tunnel-2
  set auto-key ike-gateway ike-vpn-44a8938f-1
  set auto-key ipsec-crypto-profile ipsec-vpn-44a8938f-1
  set tunnel-interface tunnel.2
  set anti-replay yes




![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  
        
! #4: Border Gateway Protocol (BGP) Configuration
!                                                                                     
! BGP is used within the tunnel to exchange prefixes between the
! Virtual Private Gateway and your Customer Gateway. The Virtual Private Gateway    
! will announce the prefix corresponding to your VPC.
!            
!
!
! The local BGP Autonomous System Number (ASN) (YOUR_BGP_ASN)
! is configured as part of your Customer Gateway. If the ASN must 
! be changed, the Customer Gateway and VPN Connection will need to be recreated with AWS.
!


 edit network virtual-router default protocol bgp
  set enable yes
  set router-id YOUR_UPLINK_ADDRESS
  set local-as YOUR_BGP_ASN
   edit peer-group AmazonBGP
    edit peer amazon-tunnel-44a8938f-1
     set connection-options keep-alive-interval 10
     set connection-options hold-time 30
     set enable yes
     set local-address ip 169.254.255.1/30
     set local-address interface tunnel.2
     set peer-as 7224
     set peer-address ip 169.254.255.6.113
     top



! Your Customer Gateway may announce a default route (0.0.0.0/0) to us. 

 edit network virtual-router default protocol bgp policy
  set export rules vr-export action allow 
  set match address-prefix 0.0.0.0/0 exact yes
  set used-by AmazonBGP enable yes
 top
                

! To advertise additional prefixes to Amazon VPC, add these prefixes to the 'address-prefix'
! statement and identify the prefix you wish to advertise. Make sure the prefix is present 
! in the routing table of the device with a valid next-hop. If you want to advertise 
! 192.168.0.0/16 to Amazon, this can be done using the following.

 edit network virtual-router default protocol bgp policy
  set export rules vr-export action allow 
  set match address-prefix 192.168.0.0/16 exact yes
  set used-by AmazonBGP enable yes
 top


!
```

## How to Test the Customer Gateway Configuration<a name="palo-alto-testing"></a>

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