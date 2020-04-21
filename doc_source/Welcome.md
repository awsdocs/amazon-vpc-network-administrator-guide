# Welcome<a name="Welcome"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

Welcome to the *AWS Site\-to\-Site VPN Network Administrator Guide*\. This guide is for customers who plan to use an AWS Site\-to\-Site VPN connection with their virtual private cloud \(VPC\)\. The topics in this guide help you configure your customer gateway device, which is the device on your side of the VPN connection\. 

Although the term VPN connection is a general term, in this documentation, a VPN connection refers to the connection between your VPC and your own on\-premises network\. Site\-to\-Site VPN supports Internet Protocol security \(IPsec\) VPN connections\. For a list of customer gateway devices that we have tested with, see [Customer Gateway Devices We've Tested](Introduction.md#DevicesTested)\.

The VPN connection lets you bridge your VPC and IT infrastructure\. You extend your existing security and management policies to EC2 instances in your VPC as if they were running within your own infrastructure\. 

For more information, see the following topics:
+ [Your Customer Gateway Device](Introduction.md)
+ [Example: Check Point Device with Border Gateway Protocol](check-point-bgp.md)
+ [Example: Check Point Device without Border Gateway Protocol](check-point-NoBGP.md)
+ [Example: Cisco ASA Device](Cisco_ASA.md)
+ [Example: Cisco IOS Device](Cisco.md)
+ [Example: Cisco IOS Device without Border Gateway Protocol](Cisco_NoBGP.md)
+ [Example: Cisco ASA Device with a Virtual Tunnel Interface and Border Gateway Protocol](cisco-asa-vti-bgp.md)
+ [Example: Cisco ASA Device with a Virtual Tunnel Interface \(without Border Gateway Protocol\)](cisco-asa-vti-no-bgp.md)
+ [Example: SonicWALL SonicOS Device Without Border Gateway Protocol](sonicwall-static.md)
+ [Example: SonicWALL Device](sonicwall-bgp.md)
+ [Example: Juniper J\-Series JunOS Device](Juniper.md)
+ [Example: Juniper SRX JunOS Device](juniper-srx.md)
+ [Example: Juniper ScreenOS Device](Juniper-with-screenos.md)
+ [Example: Netgate PfSense Device without Border Gateway Protocol](pfsense-no-bgp.md)
+ [Example: Palo Alto Networks Device](palo-alto.md)
+ [Example: Yamaha Device](Yamaha.md)
+ [Example: Generic Customer Gateway Device Using Border Gateway Protocol](GenericConfig.md)
+ [Example: Generic Customer Gateway Device without Border Gateway Protocol](GenericConfigNoBGP.md)
+ [Configuring Windows Server 2008 R2 as a Customer Gateway Device](CustomerGateway-Windows.md)
+ [Configuring Windows Server 2012 R2 as a Customer Gateway Device](customer-gateway-windows-2012.md)