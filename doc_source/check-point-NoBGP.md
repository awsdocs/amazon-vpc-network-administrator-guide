# Example: Check Point Device without Border Gateway Protocol<a name="check-point-NoBGP"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

This section has example configuration information provided by your integration team if your customer gateway is a Check Point Security Gateway device running R77\.10 or above, and using the Gaia operating system\. 

Before you begin, ensure that you've done the following:
+ You've created a Site\-to\-Site VPN connection in Amazon VPC\. For more information, see [Getting Started](https://docs.aws.amazon.com/vpc/latest/userguide/SetUpVPNConnections.html) in the *AWS Site\-to\-Site VPN User Guide*\.
+ You've read the [requirements](Introduction.md#CGRequirements) for your customer gateway device\.

**Topics**
+ [High\-Level View of the Customer Gateway](#check-point-NoBGP-overview)
+ [Configuration File](#check-point-NoBGP-example-config)
+ [Configuring the Check Point Device](#check-point-ui-configuration)
+ [How to Test the Customer Gateway Configuration](#test-check-point-NoBGP)

## High\-Level View of the Customer Gateway<a name="check-point-NoBGP-overview"></a>

The following diagram shows the general details of your customer gateway\. The VPN connection consists of two separate tunnels\. Using redundant tunnels ensures continuous availability in the case that a device fails\.

![\[Check Point without BGP high-level diagram\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/highlevel-generic-nobgp-diagram.png)

## Configuration File<a name="check-point-NoBGP-example-config"></a>

Your integration team can provide you with a configuration file that has the values you need to configure each tunnel and the IKE and IPsec settings for your VPN device\. The configuration file includes instructions on how to use the Gaia web portal and Check Point SmartDashboard to configure your device\. The same steps are provided in the next section\.

The following is an extract of an example configuration file\. The file contains two sections: `IPSec Tunnel #1` and `IPSec Tunnel #2`\. You must use the values provided in each section to configure each tunnel\.

```
! Amazon Web Services
! Virtual Private Cloud

! AWS uses unique identifiers to manipulate the configuration of 
! a VPN connection. Each VPN connection is assigned an identifier and is 
! associated with two other identifiers, namely the 
! customer gateway identifier and virtual private gateway identifier.
!
! Your VPN connection ID 		  : vpn-12345678
! Your virtual private gateway ID : vgw-12345678
! Your customer gateway ID		  : cgw-12345678
!
!
! This configuration consists of two tunnels. Both tunnels must be 
! configured on your customer gateway.
!

! --------------------------------------------------------------------------------
! IPSec Tunnel #1
! --------------------------------------------------------------------------------
! #1: Tunnel Interface Configuration

 ...
 
! --------------------------------------------------------------------------------
! --------------------------------------------------------------------------------
! IPSec Tunnel #2
! --------------------------------------------------------------------------------
! #1: Tunnel Interface Configuration

...
```

## Configuring the Check Point Device<a name="check-point-ui-configuration"></a>

The following procedures demonstrate how to configure the VPN tunnels, network objects, and security for your VPN connection\. You must replace the example values in the procedures with the values that are provided in the configuration file\. 

**Note**  
For more information, go to the [Check Point Security Gateway IPsec VPN to Amazon Web Services VPC](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk100726) article on the Check Point Support Center\.

**Topics**
+ [Step 1: Configure Tunnel Interface](#check-point-configure-tunnel)
+ [Step 2: Configure the Static Route](#check-point-static-routes)
+ [Step 3: Create Network Objects](#check-point-define-network-objects)
+ [Step 4: Create a VPN Community and Configure IKE and IPsec](#check-point-vpn-community)
+ [Step 5: Configure the Firewall](#check-point-firewall)
+ [Step 6: Enable Dead Peer Detection and TCP MSS Clamping](#check-point-dpd)

### Step 1: Configure Tunnel Interface<a name="check-point-configure-tunnel"></a>

The first step is to create the VPN tunnels and provide the private \(inside\) IP addresses of the customer gateway and virtual private gateway for each tunnel\. To create the first tunnel, use the information provided under the `IPSec Tunnel #1` section of the configuration file\. To create the second tunnel, use the values provided in the `IPSec Tunnel #2` section of the configuration file\. 

**To configure the tunnel interface**

1. Open the Gaia portal of your Check Point Security Gateway device\.

1. Choose **Network Interfaces**, **Add**, **VPN tunnel**\.

1. In the dialog box, configure the settings as follows, and choose **OK** when you are done:
   + For **VPN Tunnel ID**, enter any unique value, such as 1\.
   + For **Peer**, enter a unique name for your tunnel, such as `AWS_VPC_Tunnel_1` or `AWS_VPC_Tunnel_2`\.
   + Ensure that **Numbered** is selected, and for **Local Address**, enter the IP address specified for `CGW Tunnel IP` in the configuration file, for example, `169.254.44.234`\. 
   + For **Remote Address**, enter the IP address specified for `VGW Tunnel IP` in the configuration file, for example, `169.254.44.233`\.  
![\[Check Point Add VPN Tunnel dialog box\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/check-point-create-tunnel.png)

1. Connect to your security gateway over SSH\. If you're using the non\-default shell, change to clish by running the following command: `clish`

1. For tunnel 1, run the following command:

   ```
   set interface vpnt1 mtu 1436
   ```

   For tunnel 2, run the following command:

   ```
   set interface vpnt2 mtu 1436
   ```

1. Repeat these steps to create a second tunnel, using the information under the `IPSec Tunnel #2` section of the configuration file\.

### Step 2: Configure the Static Route<a name="check-point-static-routes"></a>

In this step, you specify the static route to the subnet in the VPC for each tunnel to enable you to send traffic over the tunnel interfaces\. The second tunnel enables failover in case there is an issue with the first tunnel\. If an issue is detected, the policy\-based static route is removed from the routing table, and the second route is activated\. You must also enable the Check Point gateway to ping the other end of the tunnel to check if the tunnel is up\.

**To configure the static routes**

1. In the Gaia portal, choose **IPv4 Static Routes**, **Add**\.

1. Specify the CIDR of your subnet, for example, `10.28.13.0/24`\.

1. Choose **Add Gateway**, **IP Address**\.

1. Enter the IP address specified for `VGW Tunnel IP` in the configuration file \(for example, `169.254.44.233`\), and specify a priority of 1\.

1. Select **Ping**\.

1. Repeat steps 3 and 4 for the second tunnel, using the `VGW Tunnel IP` value under the `IPSec Tunnel #2` section of the configuration file\. Specify a priority of 2\.  
![\[Check Point Edit Destination Route dialog box\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/check-point-static-routes.png)

1. Choose **Save**\.

If you're using a cluster, repeat the steps above for the other members of the cluster\.

### Step 3: Create Network Objects<a name="check-point-define-network-objects"></a>

In this step, you create a network object for each VPN tunnel, specifying the public \(outside\) IP addresses for the virtual private gateway\. You later add these network objects as satellite gateways for your VPN community\. You also need to create an empty group to act as a placeholder for the VPN domain\. 

**To define a new network object**

1. Open the Check Point SmartDashboard\.

1. For **Groups**, open the context menu and choose **Groups**, **Simple Group**\. You can use the same group for each network object\.

1. For **Network Objects**, open the context \(right\-click\) menu and choose **New**, **Interoperable Device**\.

1. For **Name**, enter the name you provided for your tunnel, for example, `AWS_VPC_Tunnel_1` or `AWS_VPC_Tunnel_2`\.

1. For **IPv4 Address**, enter the outside IP address of the virtual private gateway provided in the configuration file, for example, `54.84.169.196`\. Save your settings and close the dialog box\.  
![\[Check Point Interoperable Device dialog box\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/check-point-network-device.png)

1. In the SmartDashboard, open your gateway properties and in the category pane, choose **Topology**\. 

1. To retrieve the interface configuration, choose **Get Topology**\.

1. In the **VPN Domain** section, choose **Manually defined**, and browse to and select the empty simple group that you created in step 2\. Choose **OK**\.
**Note**  
You can keep any existing VPN domain that you've configured\. However, ensure that the hosts and networks that are used or served by the new VPN connection are not declared in that VPN domain, especially if the VPN domain is automatically derived\.

1. Repeat these steps to create a second network object, using the information under the `IPSec Tunnel #2` section of the configuration file\.

**Note**  
If you're using clusters, then edit the topology and define the interfaces as cluster interfaces\. Use the IP addresses specified in the configuration file\. 

### Step 4: Create a VPN Community and Configure IKE and IPsec<a name="check-point-vpn-community"></a>

In this step, you create a VPN community on your Check Point gateway, to which you add the network objects \(interoperable devices\) for each tunnel\. You also configure the Internet Key Exchange \(IKE\) and IPsec settings\.

**To create and configure the VPN community, IKE, and IPsec settings**

1. From your gateway properties, choose **IPSec VPN** in the category pane\.

1. Choose **Communities**, **New**, **Star Community**\.

1. Provide a name for your community \(for example, `AWS_VPN_Star`\), and then choose **Center Gateways** in the category pane\.

1. Choose **Add**, and add your gateway or cluster to the list of participant gateways\.

1. In the category pane, choose **Satellite Gateways**, **Add**, and add the interoperable devices you created earlier \(`AWS_VPC_Tunnel_1` and `AWS_VPC_Tunnel_2`\) to the list of participant gateways\.

1. In the category pane, choose **Encryption**\. In the **Encryption Method** section, choose **IKEv1 only**\. In the **Encryption Suite** section, choose **Custom**, **Custom Encryption**\.

1. In the dialog box, configure the encryption properties as follows, and choose **OK** when you're done:
   + IKE Security Association \(Phase 1\) Properties:
     + **Perform key exchange encryption with**: AES\-128
     + **Perform data integrity with**: SHA1
   + IPsec Security Association \(Phase 2\) Properties:
     + **Perform IPsec data encryption with**: AES\-128
     + **Perform data integrity with**: SHA\-1

1. In the category pane, choose **Tunnel Management**\. Choose **Set Permanent Tunnels**, **On all tunnels in the community**\. In the **VPN Tunnel Sharing** section, choose **One VPN tunnel per Gateway pair**\.

1. In the category pane, expand **Advanced Settings**, and choose **Shared Secret**\.

1. Select the peer name for the first tunnel, choose **Edit**, and enter the pre\-shared key as specified in the configuration file in the `IPSec Tunnel #1` section\.

1. Select the peer name for the second tunnel, choose **Edit**, and enter the pre\-shared key as specified in the configuration file in the `IPSec Tunnel #2` section\.  
![\[Check Point Interoperable Shared Secret dialog box\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/check-point-shared-secret.png)

1. Still in the **Advanced Settings** category, choose **Advanced VPN Properties**, configure the properties as follows, and choose **OK** when you're done:
   + IKE \(Phase 1\):
     + **Use Diffie\-Hellman group**: `Group 2`
     + **Renegotiate IKE security associations every** `480` **minutes**
   + IPsec \(Phase 2\):
     + Choose **Use Perfect Forward Secrecy**
     + **Use Diffie\-Hellman group**: `Group 2`
     + **Renegotiate IPsec security associations every** `3600` **seconds**

### Step 5: Configure the Firewall<a name="check-point-firewall"></a>

In this step, you configure a policy with firewall rules and directional match rules that allow communication between the VPC and the local network\. You then install the policy on your gateway\.

**To create firewall rules**

1. In the SmartDashboard, choose **Global Properties** for your gateway\. In the category pane, expand **VPN**, and choose **Advanced**\.

1. Choose **Enable VPN Directional Match in VPN Column**, and save your changes\.

1. In the SmartDashboard, choose **Firewall**, and create a policy with the following rules: 
   + Allow the VPC subnet to communicate with the local network over the required protocols\. 
   + Allow the local network to communicate with the VPC subnet over the required protocols\.

1. Open the context menu for the cell in the VPN column, and choose **Edit Cell**\. 

1. In the **VPN Match Conditions** dialog box, choose **Match traffic in this direction only**\. Create the following directional match rules by choosing **Add** for each, and choose **OK** when you're done:
   + `internal_clear` > VPN community \(The VPN star community you created earlier, for example, `AWS_VPN_Star`\)
   + VPN community > VPN community
   + VPN community > `internal_clear`

1. In the SmartDashboard, choose **Policy**, **Install**\. 

1. In the dialog box, choose your gateway and choose **OK** to install the policy\.

### Step 6: Enable Dead Peer Detection and TCP MSS Clamping<a name="check-point-dpd"></a>

Your Check Point gateway can use Dead Peer Detection \(DPD\) to identify when an IKE association is down\.

To configure DPD for a permanent tunnel, the permanent tunnel must be configured in the AWS VPN community \(refer to Step 8 in [Step 4: Create a VPN Community and Configure IKE and IPsec](#check-point-vpn-community)\)\.

By default, the `tunnel_keepalive_method` property for a VPN gateway is set to `tunnel_test`\. You must change the value to `dpd`\. Each VPN gateway in the VPN community that requires DPD monitoring must be configured with the `tunnel_keepalive_method` property, including any 3rd party VPN gateway\. You cannot configure different monitoring mechanisms for the same gateway\.

You can update the `tunnel_keepalive_method` property using the GuiDBedit tool\.

**To modify the tunnel\_keepalive\_method property**

1. Open the Check Point SmartDashboard, and choose **Security Management Server**, **Domain Management Server**\.

1. Choose **File**, **Database Revision Control\.\.\.** and create a revision snapshot\.

1. Close all SmartConsole windows, such as the SmartDashboard, SmartView Tracker, and SmartView Monitor\.

1. Start the GuiBDedit tool\. For more information, see the [Check Point Database Tool](http://supportcontent.checkpoint.com/solutions?id=sk13009) article on the Check Point Support Center\. 

1. Choose **Security Management Server**, **Domain Management Server**\.

1. In the upper left pane, choose **Table**, **Network Objects**, **network\_objects**\. 

1. In the upper right pane, select the relevant **Security Gateway**, **Cluster** object\. 

1. Press CTRL\+F, or use the **Search** menu to search for the following: `tunnel_keepalive_method`\.

1. In the lower pane, open the context menu for `tunnel_keepalive_method`, and choose **Edit\.\.\.**\. Choose **dpd** and choose **OK**\.

1. Repeat steps 7–9 for each gateway that's part of the AWS VPN Community\.

1. Choose **File**, **Save All**\.

1. Close the GuiDBedit tool\.

1. Open the Check Point SmartDashboard, and choose **Security Management Server**, **Domain Management Server**\.

1. Install the policy on the relevant **Security Gateway**, **Cluster** object\.

For more information, see the [New VPN features in R77\.10](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk97746) article on the Check Point Support Center\.

TCP MSS clamping reduces the maximum segment size of TCP packets to prevent packet fragmentation\.

**To enable TCP MSS clamping**

1. Navigate to the following directory: `C:\Program Files (x86)\CheckPoint\SmartConsole\R77.10\PROGRAM\`\.

1. Open the Check Point Database Tool by running the `GuiDBEdit.exe` file\.

1. Choose **Table**, **Global Properties**, **properties**\.

1. For `fw_clamp_tcp_mss`, choose **Edit**\. Change the value to `true` and choose **OK**\.

## How to Test the Customer Gateway Configuration<a name="test-check-point-NoBGP"></a>

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

On the Check Point gateway side, you can verify the tunnel status by running the following command from the command line tool in expert mode:

```
vpn tunnelutil
```

In the options that display, choose 1 to verify the IKE associations and 2 to verify the IPsec associations\.

You can also use the Check Point Smart Tracker Log to verify that packets over the connection are being encrypted\. For example, the following log indicates that a packet to the VPC was sent over tunnel 1 and was encrypted\.

![\[Check Point log file\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/check-point-log.png)