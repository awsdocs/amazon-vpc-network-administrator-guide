# Troubleshooting Generic Device Customer Gateway without Border Gateway Protocol Connectivity<a name="Generic_Troubleshooting_noBGP"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

The following diagram and table provide general instructions for troubleshooting a customer gateway device that does not use Border Gateway Protocol\.

**Tip**  
When troubleshooting problems, you might find it useful to enable the debug features of your gateway device\. Consult your gateway device vendor for details\.

![\[Flow chart for troubleshooting generic customer gateway\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/troubleshooting-cgw-flow-nobgp-diagram.png)


|  |  | 
| --- |--- |
|  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IKE.png)  |  Determine if an IKE Security Association exists\. An IKE security association is required to exchange keys that are used to establish the IPsec Security Association\.  If no IKE security association exists, review your IKE configuration settings\. You must configure the encryption, authentication, perfect\-forward\-secrecy, and mode parameters as listed in the customer gateway configuration\. If an IKE security association exists, move on to IPsec\.  | 
|  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/IPsec.png)  |   Determine if an IPsec Security Association exists\. An IPsec security association is the tunnel itself\. Query your customer gateway to determine if an IPsec Security Association is active\. Proper configuration of the IPsec SA is critical\. You must configure the encryption, authentication, perfect\-forward\-secrecy, and mode parameters as listed in the customer gateway configuration\. If no IPsec Security Association exists, review your IPsec configuration\. If an IPsec Security Association exists, move on to the tunnel\.   | 
|  ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/Tunnel.png)  |  Confirm that the required firewall rules are set up \(for a list of the rules, see [Configuring a Firewall Between the Internet and Your Customer Gateway Device](Introduction.md#FirewallRules)\)\. If they are, move forward\. Determine if there is IP connectivity via the tunnel\. Each side of the tunnel has an IP address as specified in the customer gateway configuration\. The virtual private gateway address is the address used as the BGP neighbor address\. From your customer gateway, ping this address to determine if IP traffic is being properly encrypted and decrypted\. If the ping isn't successful, review your tunnel interface configuration to make sure that the proper IP address is configured\. If the ping is successful, move on to Routing\.  | 
|  Static routes  |  Routing: For each tunnel, do the following: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/vpc/latest/adminguide/Generic_Troubleshooting_noBGP.html) If the tunnels are not in this state, review your device configuration\. Make sure that both tunnels are in this state, and you're done\.  | 
|     |  Make sure that your virtual private gateway is attached to your VPC\. Your integration team does this in the AWS Management Console\.  | 

If you have questions or need further assistance, use the [Amazon VPC forum](https://forums.aws.amazon.com/forum.jspa?forumID=58)\. 