# Your Customer Gateway Device<a name="Introduction"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

**Topics**
+ [What Is a Customer Gateway Device?](#CustomerGateway)
+ [Overview of Setting Up a VPN Connection](#Summary)
+ [AWS VPN CloudHub and Redundant Customer Gateways](#VPNCloudHub)
+ [Configuring Multiple VPN Connections to Your VPC](#MultipleVPNConnections)
+ [Customer Gateway Devices We've Tested](#DevicesTested)
+ [Requirements for Your Customer Gateway Device](#CGRequirements)
+ [Configuring a Firewall Between the Internet and Your Customer Gateway Device](#FirewallRules)

## What Is a Customer Gateway Device?<a name="CustomerGateway"></a>

An Amazon VPC VPN connection links your data center \(or network\) to your Amazon Virtual Private Cloud \(VPC\)\. A *customer gateway device* is the anchor on your side of that connection\. It can be a physical or software appliance\. The anchor on the AWS side of the VPN connection is called a *virtual private gateway*\.

The following diagram shows your network, the customer gateway, the VPN connection that goes to the virtual private gateway, and the VPC\. There are two lines between the customer gateway device and virtual private gateway because the VPN connection consists of two tunnels to provide increased availability for the Amazon VPC service\. If there's a device failure within AWS, your VPN connection automatically fails over to the second tunnel so that your access isn't interrupted\. From time to time, AWS also performs routine maintenance on the virtual private gateway, which may briefly disable one of the two tunnels of your VPN connection\. Your VPN connection automatically fails over to the second tunnel while this maintenance is performed\. When you configure your customer gateway, it's therefore important that you configure both tunnels\.

![\[Basic VPN diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/basic-cust-gateway-diagram.png)

You can create additional VPN connections to other VPCs using the same customer gateway device\. You can reuse the same customer gateway IP address for each of those VPN connections\.

When you create a VPN connection, the VPN tunnel comes up when traffic is generated from your side of the VPN connection\. The virtual private gateway is not the initiator; your customer gateway device must initiate the tunnels\. AWS VPN endpoints support rekey and can start renegotiations when phase 1 is about to expire if the customer gateway device hasn't sent any renegotiation traffic\.

 For more information about the components of a VPN connection, see [VPN Connections](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html#VPN) in the *AWS Site\-to\-Site VPN User Guide*\.

To protect against a loss of connectivity if your customer gateway device becomes unavailable, you can set up a second VPN connection\. For more information about redundant connections, see [Using Redundant VPN Connections to Provide Failover](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.

### Your Role<a name="YourRole"></a>

Throughout this guide, we refer to your company's *integration team*, which is the person \(or persons\) at your company working to integrate your infrastructure with Amazon VPC\. This team \(which may or may not consist of you\) must use the [AWS Management Console](http://aws.amazon.com/console) to create a VPN connection and get the information that you need for configuring your customer gateway\. Your company might have a separate team for each task \(an integration team that uses the AWS Management Console\)\. They might have a separate network engineering group that has access to network devices and configures the customer gateway\. This guide assumes that you're someone in the network engineering group who receives information from your company's integration team so you can then configure the customer gateway device\. 

## Overview of Setting Up a VPN Connection<a name="Summary"></a>

The process of setting up the VPN connection in AWS is covered in the *AWS Site\-to\-Site VPN User Guide*\. One task in the overall process is to configure the customer gateway\. To create the VPN connection, AWS needs information about the customer gateway, and you must configure the customer gateway device itself\. 

To set up a VPN connection, follow these general steps:

1. Designate an appliance to act as your customer gateway device\. For more information, see [Customer Gateway Devices We've Tested](#DevicesTested) and [Requirements for Your Customer Gateway Device](#CGRequirements)\.

1. Get the necessary [Network Information](#DetermineNetworkInfo), and provide this information to the team to create the VPN connection in AWS\.

1. Create the VPN connection in AWS and get the configuration file for your customer gateway\. For more information about how to configure an AWS VPN connection, see [Setting Up an AWS VPN Connection](https://docs.aws.amazon.com/vpn/latest/s2svpn/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.

1. Configure your customer gateway device using the information from the configuration file\. Examples are provided in this guide\.

1. Generate traffic from your side of the VPN connection to bring up the VPN tunnel\.

### Network Information<a name="DetermineNetworkInfo"></a>

To create a VPN connection in AWS, you need the following information\.


****  

| Item | Comments | 
| --- | --- | 
| Customer gateway vendor \(for example, Cisco\), platform \(for example, ISR Series Routers\), and software version \(for example, IOS 12\.4\) | This information is used to generate a configuration file for the customer gateway device\. | 
| The internet\-routable IP address for the customer gateway device's external interface\. | The value must be static\. If your customer gateway device resides behind a device performing network address translation \(NAT\), use the public IP address of the NAT device\.  For source NAT, do not use the public IP address of the customer gateway as the source IP address for packets that are sent through a VPN tunnel\. Instead, use a different IP address that is not in use\. | 
| \(Optional\) Border Gateway Protocol \(BGP\) Autonomous System Number \(ASN\) of the customer gateway\. | You can use an existing ASN assigned to your network\. If you don't have one, you can use a private ASN in the 64512–65534 range\. Otherwise, we assume that the BGP ASN for the customer gateway is 65000\. | 
|  \(Optional\) The ASN for the Amazon side of the BGP session\.  |  Specified when creating a virtual private gateway\. If you do not specify a value, the default ASN applies\. For more information, see [Virtual Private Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_VPN.html#VPNGateway)\.  | 
|  \(Optional\) Tunnel information for each VPN tunnel  |  You can configure some of the tunnel options for the VPN connection\. For more information, see [Site\-to\-Site VPN Tunnel Options for Your Site\-to\-Site VPN Connection](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNTunnels.html) in the *AWS Site\-to\-Site VPN User Guide*\.  | 
| \(Optional\) Private certificate from AWS Certificate Manager Private Certificate Authority to authenticate your VPN | You must create a private certificate using AWS Certificate Manager Private Certificate Authority\. For information about creating a private certificate, see [Creating and Managing a Private CA](https://docs.aws.amazon.com/acm-pca/latest/userguide/PcaCreatingManagingCA.html) in the AWS Certificate Manager Private Certificate Authority User Guide\. | 

The configuration file for your customer gateway device includes the values that you specify for the above items\. It also contains any additional values required for setting up the VPN tunnels, including the outside IP address for the virtual private gateway\. This value is static unless you recreate the VPN connection in AWS\.

### Routing Information<a name="cgw-routing-info"></a>

AWS recommends advertising more specific BGP routes to influence routing decisions in the virtual private gateway\. Check your vendor documentation for the commands that are specific to your device\.

For more information about route priority, see [Route Tables and VPN Route Priority](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNRoutingTypes.html#vpn-route-priority) in the *AWS Site\-to\-Site VPN User Guide*\.

## AWS VPN CloudHub and Redundant Customer Gateways<a name="VPNCloudHub"></a>

You can establish multiple VPN connections to a single virtual private gateway from multiple customer gateway devices\. This configuration can be used in different ways\. You can have redundant customer gateway devices between your data center and your VPC, or you can have multiple locations connected to the AWS VPN CloudHub\.

If you have redundant customer gateway devices, each device advertises the same prefix \(for example, 0\.0\.0\.0/0\) to the virtual private gateway\. We use BGP routing to determine the path for traffic\. If one customer gateway device fails, the virtual private gateway directs all traffic to the working customer gateway device\.

If you use the AWS VPN CloudHub configuration, multiple sites can access your VPC or securely access each other using a simple hub\-and\-spoke model\. You configure each customer gateway device to advertise a site\-specific prefix \(such as 10\.0\.0\.0/24, 10\.0\.1\.0/24\) to the virtual private gateway\. The virtual private gateway routes traffic to the appropriate site and advertises the reachability of one site to all other sites\.

To configure the AWS VPN CloudHub, use the Amazon VPC console to create multiple customer gateways, each with the public IP address of the gateway\. You must use a unique Border Gateway Protocol \(BGP\) Autonomous System Number \(ASN\) for each\. Then create a VPN connection from each customer gateway to a common virtual private gateway\. Use the instructions that follow to configure each customer gateway device to connect to the virtual private gateway\.

To enable instances in your VPC to reach the virtual private gateway \(and then your customer gateway devices\), you must configure routes in your VPC routing tables\. For complete instructions, see [VPN Connections](https://docs.aws.amazon.com/vpn/latest/s2svpn/vpn-connections.html) in the *AWS Site\-to\-Site VPN User Guide*\. For AWS VPN CloudHub, you can configure an aggregate route in your VPC routing table \(for example, 10\.0\.0\.0/16\)\. Use more specific prefixes between customer gateways devices and the virtual private gateway\.

## Configuring Multiple VPN Connections to Your VPC<a name="MultipleVPNConnections"></a>

You can create up to ten VPN connections for your VPC\. You can use multiple VPN connections to link your remote offices to the same VPC\. For example, if you have offices in Los Angeles, Chicago, New York, and Miami, you can link each of these offices to your VPC\. You can also use multiple VPN connections to establish redundant customer gateway devices from a single location\.

**Note**  
If you need more than ten VPN connections, [submit a request](http://aws.amazon.com/contact-us/vpc-request/) to increase your quota\.

When you create multiple VPN connections, the virtual private gateway sends network traffic to the appropriate VPN connection using statically assigned routes or BGP route advertisements\. Which one depends on how the VPN connection was configured\. Statically assigned routes are preferred over BGP advertised routes in cases where identical routes exist in the virtual private gateway\. If you select the option to use BGP advertisement, then you cannot specify static routes\.

When you have customer gateway devices at multiple geographic locations, each device should advertise a unique set of IP ranges specific to the location\. When you establish redundant customer gateway devices at a single location, both devices should advertise the same IP ranges\.

When a virtual private gateway receives routing information, it uses path selection to determine how to route traffic\. For more information, see [Route Tables and VPN Route Priority](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNRoutingTypes.html#vpn-route-priority) in the *AWS Site\-to\-Site VPN User Guide*\.

The following diagram shows the configuration of multiple VPNs\.

![\[Multiple VPN layout\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/branch-offices-diagram.png)

## Customer Gateway Devices We've Tested<a name="DevicesTested"></a>

Your customer gateway device can be a physical or software appliance\.

This guide presents information about how to configure the following devices that we have tested with:
+ Check Point Security Gateway running R77\.10 \(or later\) software
+ Cisco ASA running Cisco ASA 8\.2 \(or later\) software
+ Cisco IOS running Cisco IOS 12\.4 \(or later\) software
+ SonicWALL running SonicOS 5\.9 \(or later\) software
+ Fortinet Fortigate 40\+ Series running FortiOS 4\.0 \(or later\) software 
+ Juniper J\-Series running JunOS 9\.5 \(or later\) software
+ Juniper SRX running JunOS 11\.0 \(or later\) software
+ Juniper SSG running ScreenOS 6\.1, or 6\.2 \(or later\) software
+ Juniper ISG running ScreenOS 6\.1, or 6\.2 \(or later\) software
+ Netgate pfSense running OS 2\.2\.5 \(or later\) software\.
+ Palo Alto Networks PANOS 4\.1\.2 \(or later\) software
+ Yamaha RT107e, RTX1200, RTX1210, RTX1500, RTX3000 and SRT100 routers
+ Microsoft Windows Server 2008 R2 \(or later\) software
+ Microsoft Windows Server 2012 R2 \(or later\) software 
+ Zyxel Zywall Series 4\.20 \(or later\) software for statically routed VPN connections, or 4\.30 \(or later\) software for dynamically routed VPN connections

If you have one of these devices, but configure it for IPsec in a different way than presented in this guide, feel free to alter our suggested configuration to match your particular needs\.

## Requirements for Your Customer Gateway Device<a name="CGRequirements"></a>

There are four main parts to the configuration of your customer gateway device\. Throughout this guide, we use a symbol for each of these parts to help you understand what you need to do\. The following table shows the four parts and the corresponding symbols\.


|  |  | 
| --- |--- |
|  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  |  IKE Security Association \(required to exchange keys used to establish the IPsec security association\)  | 
|  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  |  IPsec Security Association \(handles the tunnel's encryption, authentication, and so on\.\)  | 
|  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  |  Tunnel interface \(receives traffic going to and from the tunnel\)  | 
| Optional ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)  |  BGP peering \(exchanges routes between the customer gateway device and the virtual private gateway\) for devices that use BGP  | 

If you have a device that isn't in the preceding list of tested devices, this section describes the requirements the device must meet for you to use it with Amazon VPC\. The following table lists the requirement the customer gateway device must adhere to, the related RFC \(for reference\), and comments about the requirement\. For an example of the configuration information if your device isn't one of the tested Cisco or Juniper devices, see [Example: Generic Customer Gateway Device Using Border Gateway Protocol](GenericConfig.md)\.

Each VPN connection consists of 2 separate tunnels\. Each tunnel contains an IKE Security Association, an IPsec Security Association, and a BGP Peering\. You are limited to 1 unique Security Association \(SA\) pair per tunnel \(1 inbound and 1 outbound\), and therefore 2 unique SA pairs in total for 2 tunnels \(4 SAs\)\. Some devices use a policy\-based VPN and create as many SAs as ACL entries\. Therefore, you may need to consolidate your rules and then filter so you don't permit unwanted traffic\.

The VPN tunnel comes up when traffic is generated from your side of the VPN connection\. The AWS endpoint is not the initiator; your customer gateway device must initiate the tunnels\.


|  Requirement  |  RFC |  Comments | 
| --- | --- | --- | 
|  Establish IKE Security Association using pre\-shared keys   ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)   |   [RFC 2409](http://tools.ietf.org/html/rfc2409)  [RFC 7296](https://tools.ietf.org/html/rfc7296)  | The IKE Security Association is established first between the virtual private gateway and customer gateway device using the pre\-shared key or a private certificate using AWS Certificate Manager Private Certificate Authority as the authenticator\. Upon establishment, IKE negotiates an ephemeral key to secure future IKE messages\. Proper establishment of an IKE Security Association requires complete agreement among the parameters, including encryption and authentication parameters\.When you create a VPN connection in AWS, you can specify your own pre\-shared key for each tunnel, or you can let AWS generate one for you\. Alternatively, you can specify the private certificate using AWS Certificate Manager Private Certificate Authority to use for your customer gateway device\. For more information, about configuring VPN tunnels see [Configuring the VPN Tunnels for Your VPN Connection](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html#VPNTunnels) in the *AWS Site\-to\-Site VPN User Guide*\.The following versions are supported: IKEv1 and IKEv2\.We support Main mode only with IKEv1\. | 
|  Establish IPsec Security Associations in Tunnel mode  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)   |   [RFC 4301](http://tools.ietf.org/html/rfc4301)   |  Using the IKE ephemeral key, keys are established between the virtual private gateway and customer gateway device to form an IPsec Security Association \(SA\)\. Traffic between gateways is encrypted and decrypted using this SA\. The ephemeral keys used to encrypt traffic within the IPsec SA are automatically rotated by IKE on a regular basis to ensure confidentiality of communications\.  | 
|  Use the AES 128\-bit encryption or AES 256\-bit encryption function  |   [RFC 3602](http://tools.ietf.org/html/rfc3602)   |  The encryption function is used to ensure privacy among both IKE and IPsec Security Associations\.  | 
|  Use the SHA\-1 or SHA\-256 hashing function  |   [RFC 2404](http://tools.ietf.org/html/rfc2404)   |  This hashing function is used to authenticate both IKE and IPsec Security Associations\.  | 
|  Use Diffie\-Hellman Perfect Forward Secrecy\. The following groups are supported: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/vpc/latest/adminguide/Introduction.html)  |   [RFC 2409](http://tools.ietf.org/html/rfc2409)   |  IKE uses Diffie\-Hellman to establish ephemeral keys to secure all communication between customer gateway devices and virtual private gateways\.  | 
|  Use IPsec Dead Peer Detection  |   [RFC 3706](http://tools.ietf.org/html/rfc3706)   |  The use of Dead Peer Detection enables the VPN devices to rapidly identify when a network condition prevents delivery of packets across the internet\. When this occurs, the gateways delete the Security Associations and attempt to create new associations\. During this process, the alternate IPsec tunnel is used if possible\.  | 
|  Bind tunnel to logical interface \(route\-based VPN\)  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)   |   None   |  Your gateway must support the ability to bind the IPsec tunnel to a logical interface\. The logical interface contains an IP address used to establish BGP peering to the virtual private gateway\. This logical interface should perform no additional encapsulation \(for example, GRE, IP in IP\)\. Your interface should be set to a 1399 byte Maximum Transmission Unit \(MTU\)\.   | 
|  Fragment IP packets before encryption  |   [RFC 4459](http://tools.ietf.org/html/rfc4459)   |  When packets are too large to be transmitted, they must be fragmented\. We do not reassemble fragmented encrypted packets\. Therefore, your VPN device must fragment packets *before* encapsulating with the VPN headers\. The fragments are individually transmitted to the remote host, which reassembles them\. For more information about fragmentation, see the [ IP fragmentation](http://en.wikipedia.org/wiki/IP_fragmentation) Wikipedia article\.  | 
|  \(Optional\) Establish BGP peerings  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/BGP.png)   |   [RFC 4271](http://tools.ietf.org/html/rfc4271)   |  BGP is used to exchange routes between the customer gateway device and virtual private gateway for devices that use BGP\. All BGP traffic is encrypted and transmitted via the IPsec Security Association\. BGP is required for both gateways to exchange the IP prefixes reachable through the IPsec SA\.  | 

We recommend that you use the techniques listed in the following table\. That helps you minimize problems related to the amount of data that can be transmitted through the IPsec tunnel\. Because the connection encapsulates packets with additional network headers \(including IPsec\), the amount of data that can be transmitted in a single packet is reduced\. 


|  Technique  |  RFC |  Comments | 
| --- | --- | --- | 
|  Adjust the maximum segment size of TCP packets entering the VPN tunnel  |   [RFC 4459](http://tools.ietf.org/html/rfc4459)   |  TCP packets are often the most prevalent type of packet across IPsec tunnels\. Some gateways can change the TCP Maximum Segment Size parameter\. This causes the TCP endpoints \(clients, servers\) to reduce the amount of data sent with each packet\. This is an ideal approach, as the packets arriving at the VPN devices are small enough to be encapsulated and transmitted\.  | 
|  Reset the "Don't Fragment" flag on packets  |   [RFC 791](http://tools.ietf.org/html/rfc791)   |  Some packets carry a flag, known as the Don't Fragment \(DF\) flag, that indicates that the packet should not be fragmented\. If the packets carry the flag, the gateways generate an ICMP Path MTU Exceeded message\. In some cases, applications do not contain adequate mechanisms for processing these ICMP messages and reducing the amount of data transmitted in each packet\. Some VPN devices can override the DF flag and fragment packets unconditionally as required\. If your customer gateway device has this ability, we recommend that you use it as appropriate\.   | 

An AWS VPN connection does not support Path MTU Discovery \([RFC 1191](https://tools.ietf.org/html/rfc1191)\)\.

If you have a firewall between your customer gateway device and the internet, see [Configuring a Firewall Between the Internet and Your Customer Gateway Device](#FirewallRules)\.

## Configuring a Firewall Between the Internet and Your Customer Gateway Device<a name="FirewallRules"></a>

To use this service, you must have an internet\-routable IP address to use as the endpoint for the IPsec tunnels connecting your customer gateway device to the virtual private gateway\. If a firewall is in place between the internet and your gateway, the rules in the following tables must be in place to establish the IPsec tunnels\. The virtual private gateway addresses are in the configuration information that you get from the integration team\.


**Inbound \(from the Internet\)**  

|  | 
| --- |
|  Input Rule I1  | 
|  Source IP  |  Virtual Private Gateway 1  | 
|  Dest IP  |  Customer Gateway  | 
|  Protocol  |  UDP  | 
|  Source Port  |  500  | 
|  Destination  |  500  | 
|  Input Rule I2  | 
|  Source IP  |  Virtual Private Gateway 2  | 
|  Dest IP  |  Customer Gateway  | 
|  Protocol  |  UDP  | 
|  Source Port  |  500  | 
|  Destination Port  |  500  | 
|  Input Rule I3  | 
|  Source IP  |  Virtual Private Gateway 1  | 
|  Dest IP  |  Customer Gateway  | 
|  Protocol  |  IP 50 \(ESP\)  | 
|  Input Rule I4  | 
|  Source IP  |  Virtual Private Gateway 2  | 
|  Dest IP  |  Customer Gateway  | 
|  Protocol  |  IP 50 \(ESP\)  | 


**Outbound \(to the Internet\)**  

|  | 
| --- |
|  Output Rule O1  | 
|  Source IP  |  Customer Gateway  | 
|  Dest IP  |  Virtual Private Gateway 1  | 
|  Protocol  |  UDP  | 
|  Source Port  |  500  | 
|  Destination Port  |  500  | 
|  Output Rule O2  | 
|  Source IP  |  Customer Gateway  | 
|  Dest IP  |  Virtual Private Gateway 2  | 
|  Protocol  |  UDP  | 
|  Source Port  |  500  | 
|  Destination Port  |  500  | 
|  Output Rule O3  | 
|  Source IP  |  Customer Gateway  | 
|  Dest IP  |  Virtual Private Gateway 1  | 
|  Protocol  |  IP 50 \(ESP\)   | 
|  Output Rule O4  | 
|  Source IP  |  Customer Gateway  | 
|  Dest IP  |  Virtual Private Gateway 2  | 
|  Protocol  |  IP 50 \(ESP\)  | 

Rules I1, I2, O1, and O2 enable the transmission of IKE packets\. Rules I3, I4, O3, and O4 enable the transmission of IPsec packets containing the encrypted network traffic\.

If you are using NAT traversal \(NAT\-T\) on your device, then you must include rules that allow UDP access over port 4500\. Check if your device is advertising NAT\-T\.