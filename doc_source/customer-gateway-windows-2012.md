# Configuring Windows Server 2012 R2 as a Customer Gateway Device<a name="customer-gateway-windows-2012"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

You can configure Windows Server 2012 R2 as a customer gateway device for your VPC\. Use the following process whether you are running Windows Server 2012 R2 on an EC2 instance in a VPC, or on your own server\.

**Topics**
+ [Configuring Your Windows Server](#cgw-win2012-prerequisites-windows-server)
+ [Step 1: Create a VPN Connection and Configure Your VPC](#cgw-win2012-create-vpn)
+ [Step 2: Download the Configuration File for the VPN Connection](#cgw-win2012-download-config)
+ [Step 3: Configure the Windows Server](#cgw-win2012-configure-server)
+ [Step 4: Set Up the VPN Tunnel](#cgw-win2012-setup-tunnel)
+ [Step 5: Enable Dead Gateway Detection](#cgw-win2012-gateway-detection)
+ [Step 6: Test the VPN Connection](#cgw-win2012-test-connection)

## Configuring Your Windows Server<a name="cgw-win2012-prerequisites-windows-server"></a>

To configure Windows Server as a customer gateway device, ensure that you have Windows Server 2012 R2 on your own network, or on an EC2 instance in a VPC\. If you use an EC2 instance that you launched from a Windows AMI, do the following:
+ Disable source/destination checking for the instance:

  1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

  1. Select your Windows Server instance, and choose **Actions**, **Networking**, **Change Source/Dest\. Check**\. Choose **Yes, Disable**\.
+ Update your adapter settings so that you can route traffic from other instances:

  1. Connect to your Windows instance\. For more information, see [Connecting to Your Windows Instance](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html)\.

  1. Open the Control Panel, and start the Device Manager\.

  1. Expand the **Network adapters** node\. 

  1. Select the AWS PV network device, choose **Action**, **Properties**\.

  1. On the **Advanced** tab, disable the **IPv4 Checksum Offload**, **TCP Checksum Offload \(IPv4\)**, and **UDP Checksum Offload \(IPv4\)** properties, and then choose **OK**\.
+ Associate an Elastic IP address with the instance:

  1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

  1. In the navigation pane, choose **Elastic IPs**\. Choose **Allocate new address**\.

  1. Select the Elastic IP address, and choose **Actions**, **Associate Address**\.

  1. For **Instance**, select your Windows Server instance\. Choose **Associate**\.

  Take note of this address — you need it when you create the customer gateway in your VPC\. 
+ Ensure the instance's security group rules allow outbound IPsec traffic\. By default, a security group allows all outbound traffic; however, if the security group's outbound rules have been modified from their original state, you must create the following outbound custom protocol rules for IPsec traffic: IP protocol 50, IP protocol 51, and UDP 500\. 

Take note of the CIDR range for your network in which the Windows server is located, for example, `172.31.0.0/16`\.

## Step 1: Create a VPN Connection and Configure Your VPC<a name="cgw-win2012-create-vpn"></a>

To create a VPN connection from your VPC, you must first create a virtual private gateway and attach it to your VPC\. Then you can create a VPN connection and configure your VPC\. You must also have the CIDR range for your network in which the Windows server is located, for example, `172.31.0.0/16`\.

**To create a virtual private gateway**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **Virtual Private Gateways**, **Create Virtual Private Gateway**\.

1. You can optionally enter a name for your virtual private gateway and choose **Yes, Create**\.

1. Select the virtual private gateway that you created and choose **Attach to VPC**\.

1. In the **Attach to VPC** dialog box, select your VPC from the list and choose **Yes, Attach**\.

**To create a VPN connection**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **VPN Connections**, **Create VPN Connection**\.

1. Select the virtual private gateway from the list\. 

1. For **Customer Gateway**, choose **New**\. For **IP address**, specify the public IP address of your Windows Server\. 
**Note**  
The IP address must be static and may be behind a device performing network address translation \(NAT\)\. To ensure that NAT traversal \(NAT\-T\) can function, you must adjust your firewall rules to unblock UDP port 4500\. If your customer gateway is an EC2 Windows Server instance, use its Elastic IP address\.

1. Select the **Static** routing option, enter the **Static IP Prefixes** values for your network in CIDR notation, and then choose **Yes, Create**\.

**To configure your VPC**
+ Create a private subnet in your VPC \(if you don't have one already\) for launching instances to communicate with the Windows server\. For more information, see [Adding a Subnet to Your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#AddaSubnet)\. 
**Note**  
A private subnet is a subnet that does not have a route to an internet gateway\. The routing for this subnet is described in the next item\.
+ Update your route tables for the VPN connection:
  + Add a route to your private subnet's route table with the virtual private gateway as the target, and the Windows server's network \(CIDR range\) as the destination\.
  + Enable route propagation for the virtual private gateway\. For more information, see [Route Tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html) in the *Amazon VPC User Guide*\.
+ Create a security group configuration for your instances that allows communication between your VPC and network:
  + Add rules that allow inbound RDP or SSH access from your network\. This enables you to connect to instances in your VPC from your network\. For example, to allow computers in your network to access Linux instances in your VPC, create an inbound rule with a type of SSH, and the source set to the CIDR range of your network; for example, `172.31.0.0/16`\. For more information, see [Security Groups for Your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) in the *Amazon VPC User Guide*\.
  + Add a rule that allows inbound ICMP access from your network\. This enables you to test your VPN connection by pinging an instance in your VPC from your Windows server\. 

## Step 2: Download the Configuration File for the VPN Connection<a name="cgw-win2012-download-config"></a>

You can use the Amazon VPC console to download a Windows server configuration file for your VPN connection\.

**To download the configuration file**

1. Open the Amazon VPC console at [https://console\.aws\.amazon\.com/vpc/](https://console.aws.amazon.com/vpc/)\.

1. In the navigation pane, choose **VPN Connections**\.

1. Select your VPN connection and choose **Download Configuration**\.

1. Select **Microsoft** as the vendor, **Windows Server** as the platform, and **2012 R2** as the software\. Choose **Yes, Download**\. You can open the file or save it\.

The configuration file contains a section of information similar to the following example\. You see this information presented twice, one time for each tunnel\. Use this information when configuring the Windows Server 2012 R2 server\.

```
vgw-1a2b3c4d Tunnel1
--------------------------------------------------------------------	
Local Tunnel Endpoint:       203.0.113.1
Remote Tunnel Endpoint:      203.83.222.237
Endpoint 1:                  [Your_Static_Route_IP_Prefix]
Endpoint 2:                  [Your_VPC_CIDR_Block]
Preshared key:               xCjNLsLoCmKsakwcdoR9yX6GsEXAMPLE
```

`Local Tunnel Endpoint`  
The IP address for the customer gateway—in this case, your Windows server—that terminates the VPN connection on your network's side\. If your customer gateway is a Windows server instance, this is the instance's private IP address\. 

`Remote Tunnel Endpoint`  
One of two IP addresses for the virtual private gateway that terminates the VPN connection on the AWS side of the connection\.

`Endpoint 1`  
The IP prefix that you specified as a static route when you created the VPN connection\. These are the IP addresses in your network that are allowed to use the VPN connection to access your VPC\.

`Endpoint 2`  
The IP address range \(CIDR block\) of the VPC attached to the virtual private gateway \(for example 10\.0\.0\.0/16\)\.

`Preshared key`  
The pre\-shared key that is used to establish the IPsec VPN connection between `Local Tunnel Endpoint` and `Remote Tunnel Endpoint`\.

We suggest that you configure both tunnels as part of the VPN connection\. Each tunnel connects to a separate VPN concentrator on the Amazon side of the VPN connection\. Although only one tunnel at a time is up, the second tunnel automatically establishes itself if the first tunnel goes down\. Having redundant tunnels ensure continuous availability in the case of a device failure\. Because only one tunnel is available at a time, the Amazon VPC console indicates that one tunnel is down\. This is expected behavior, so there's no action required from you\. 

With two tunnels configured, if a device failure occurs within AWS, your VPN connection automatically fails over to the second tunnel of the AWS virtual private gateway within a matter of minutes\. When you configure your customer gateway device, it's important that you configure both tunnels\.

**Note**  
From time to time, AWS performs routine maintenance on the virtual private gateway\. This maintenance may disable one of the two tunnels of your VPN connection for a brief period of time\. Your VPN connection automatically fails over to the second tunnel while we perform this maintenance\.

Additional information regarding the Internet Key Exchange \(IKE\) and IPsec Security Associations \(SA\) is presented in the downloaded configuration file\. Because the VPC VPN suggested settings are the same as the Windows Server 2012 R2 default IPsec configuration settings, minimal work is needed on your part\.

```
MainModeSecMethods:        DHGroup2-AES128-SHA1
MainModeKeyLifetime:       480min,0sess
QuickModeSecMethods:       ESP:SHA1-AES128+60min+100000kb
QuickModePFS:              DHGroup2
```

`MainModeSecMethods`  
The encryption and authentication algorithms for the IKE SA\. These are the suggested settings for the VPN connection, and are the default settings for Windows Server 2012 R2 IPsec VPN connections\.

`MainModeKeyLifetime`  
The IKE SA key lifetime\.  This is the suggested setting for the VPN connection, and is the default setting for Windows Server 2012 R2 IPsec VPN connections\.

`QuickModeSecMethods`  
The encryption and authentication algorithms for the IPsec SA\. These are the suggested settings for the VPN connection, and are the default settings for Windows Server 2012 R2 IPsec VPN connections\.

`QuickModePFS`  
 We suggest that you use master key perfect forward secrecy \(PFS\) for your IPsec sessions\.

## Step 3: Configure the Windows Server<a name="cgw-win2012-configure-server"></a>

Before you set up the VPN tunnel, you must install and configure Routing and Remote Access Services on your Windows server\. That allows remote users to access resources on your network\.

**To install Routing and Remote Access Services on Windows Server 2012 R2**

1. Log on to the Windows Server 2012 R2 server\.

1. Go to the **Start** menu, and choose **Server Manager**\.

1. Install Routing and Remote Access Services:

   1. From the **Manage** menu, choose **Add Roles and Features**\.

   1. On the **Before You Begin** page, verify that your server meets the prerequisites, and then choose **Next**\.

   1. Choose **Role\-based or feature\-based installation**, and then choose **Next**\.

   1. Choose **Select a server from the server pool**, select your Windows 2012 R2 server, and then choose **Next**\.

   1. Select **Network Policy and Access Services** in the list\. In the dialog box that displays, choose **Add Features** to confirm the features that are required for this role\.

   1. In the same list, choose **Remote Access**, **Next**\.

   1. On the **Select features** page, choose **Next**\.

   1. On the **Network Policy and Access Services** page, choose **Next**\. Leave **Network Policy Server** selected, and choose **Next**\.

   1. On the **Remote Access** page, choose **Next**\. On the next page, select **DirectAccess and VPN \(RAS\)**\. In the dialog box that displays, choose **Add Features** to confirm the features that are required for this role service\. In the same list, select **Routing**, and then choose **Next**\.

   1. On the **Web Server Role \(IIS\)** page, choose **Next**\. Leave the default selection, and choose **Next**\.

   1. Choose **Install**\. When the installation completes, choose **Close**\.

**To configure and enable Routing and Remote Access Server**

1. On the dashboard, choose **Notifications** \(the flag icon\)\. There should be a task to complete the post\-deployment configuration\. Choose the **Open the Getting Started Wizard** link\.

1. Choose **Deploy VPN only**\.

1. In the **Routing and Remote Access** dialog box, choose the server name, choose **Action**, and select **Configure and Enable Routing and Remote Access**\.

1. In the **Routing and Remote Access Server Setup Wizard**, on the first page, choose **Next**\.

1. On the **Configuration** page, choose **Custom Configuration**, **Next**\.

1. Choose **LAN routing**, **Next**, **Finish**\.

1. When prompted by the **Routing and Remote Access** dialog box, choose **Start service**\.

## Step 4: Set Up the VPN Tunnel<a name="cgw-win2012-setup-tunnel"></a>

You can configure the VPN tunnel by running the netsh scripts included in the downloaded configuration file, or by using the New Connection Security Rule wizard on the Windows server\. 

**Important**  
We suggest that you use master key perfect forward secrecy \(PFS\) for your IPsec sessions\. If you choose to run the netsh script, it includes a parameter to enable PFS \(`qmpfs=dhgroup2`\)\. You cannot enable PFS using the Windows Server 2012 R2 user interface — you must enable it using the command line\. 

### Option 1: Run netsh Script<a name="cgw-win2012-run-netsh"></a>

Copy the netsh script from the downloaded configuration file and replace the variables\. The following is an example script\.

```
netsh advfirewall consec add rule Name="vgw-1a2b3c4d Tunnel 1" ^
Enable=Yes Profile=any Type=Static Mode=Tunnel ^
LocalTunnelEndpoint=Windows_Server_Private_IP_address ^
RemoteTunnelEndpoint=203.83.222.236 Endpoint1=Your_Static_Route_IP_Prefix ^
Endpoint2=Your_VPC_CIDR_Block Protocol=Any Action=RequireInClearOut ^
Auth1=ComputerPSK Auth1PSK=xCjNLsLoCmKsakwcdoR9yX6GsEXAMPLE ^
QMSecMethods=ESP:SHA1-AES128+60min+100000kb ^
ExemptIPsecProtectedConnections=No ApplyAuthz=No QMPFS=dhgroup2
```

**Name**: You can replace the suggested name \(`vgw-1a2b3c4d Tunnel 1)` with a name of your choice\. 

**LocalTunnelEndpoint**: Enter the private IP address of the Windows server on your network\.

**Endpoint1**: The CIDR block of your network on which the Windows server resides, for example, `172.31.0.0/16`\.

**Endpoint2**: The CIDR block of your VPC or a subnet in your VPC, for example, `10.0.0.0/16`\.

Run the updated script in a command prompt window on your Windows server\. \(The ^ enables you to cut and paste wrapped text at the command line\.\) To set up the second VPN tunnel for this VPN connection, repeat the process using the second netsh script in the configuration file\.

When you are done, go to [2\.4: Configure the Windows Firewall](#cgw-win2012-firewall)\.

For more information about the netsh parameters, go to [Netsh AdvFirewall Consec Commands](http://technet.microsoft.com/en-us/library/dd736198%28v=ws.10%29.aspx) in the *Microsoft TechNet Library*\.

### Option 2: Use the Windows Server User Interface<a name="cgw-win2012-ui"></a>

You can also use the Windows server user interface to set up the VPN tunnel\. This section guides you through the steps\.

**Important**  
You can't enable master key perfect forward secrecy \(PFS\) using the Windows Server 2012 R2 user interface\. You must enable PFS using the command line, as described in [Enable Master Key Perfect Forward Secrecy](#cgw-win2012-enable-pfs)\.

**Topics**
+ [2\.1: Configure a Security Rule for a VPN Tunnel](#cgw-win2012-security-rule)
+ [2\.3: Confirm the Tunnel Configuration](#cgw-win2012-confirm-tunnel)
+ [Enable Master Key Perfect Forward Secrecy](#cgw-win2012-enable-pfs)

#### 2\.1: Configure a Security Rule for a VPN Tunnel<a name="cgw-win2012-security-rule"></a>

In this section, you configure a security rule on your Windows server to create a VPN tunnel\.

**To configure a security rule for a VPN tunnel**

1. Open Server Manager, choose **Tools**, and select **Windows Firewall with Advanced Security**\.

1. Select **Connection Security Rules**, choose **Action**, and then **New Rule**\.

1. In the **New Connection Security Rule** wizard, on the **Rule Type** page, choose **Tunnel**, and then choose **Next**\.

1. On the **Tunnel Type** page, under **What type of tunnel would you like to create**, choose **Custom configuration**\. Under **Would you like to exempt IPsec\-protected connections from this tunnel**, leave the default value checked \(**No\. Send all network traffic that matches this connection security rule through the tunnel**\), and then choose **Next**\.

1. On the **Requirements** page, choose **Require authentication for inbound connections\. Do not establish tunnels for outbound connections**, and then choose **Next**\.

1. On **Tunnel Endpoints** page, under **Which computers are in Endpoint 1**, choose **Add**\. Enter the CIDR range of your network \(behind your Windows server customer gateway device; for example, `172.31.0.0/16` \), and then choose **OK**\. The range can include the IP address of your customer gateway device\.

1. Under **What is the local tunnel endpoint \(closest to computer in Endpoint 1\)**, choose **Edit**\. In the **IPv4 address** field, enter the private IP address of your Windows server, and then choose **OK**\.

1. Under **What is the remote tunnel endpoint \(closest to computers in Endpoint 2\)**, choose **Edit**\. In the **IPv4 address** field, enter the IP address of the virtual private gateway for Tunnel 1 from the configuration file \(see `Remote Tunnel Endpoint`\), and then choose **OK**\.
**Important**  
If you are repeating this procedure for Tunnel 2, be sure to select the endpoint for Tunnel 2\.

1. Under **Which computers are in Endpoint 2**, choose **Add**\. In the **This IP address or subnet field**, enter the CIDR block of your VPC, and then choose **OK**\.
**Important**  
You must scroll in the dialog box until you locate **Which computers are in Endpoint 2**\. Do not choose **Next** until you have completed this step, or you won't be able to connect to your server\.  
![\[New Connection Security Rule Wizard: Tunnel Endpoints\]](http://docs.aws.amazon.com/vpc/latest/adminguide/images/tunnelendpoints_complete_win2012.png)

1. Confirm that all the settings you've specified are correct and choose **Next**\.

1. On the **Authentication Method** page, select **Advanced** and choose **Customize**\.

1. Under **First authentication methods**, choose **Add**\.

1. Select **Preshared key**, enter the pre\-shared key value from the configuration file and choose **OK**\.
**Important**  
If you are repeating this procedure for Tunnel 2, be sure to select the pre\-shared key for Tunnel 2\.

1. Ensure that **First authentication is optional** is not selected, and choose **OK**\.

1. Choose **Next**\.

1. On the **Profile** page, select all three check boxes: **Domain**, **Private**, and **Public**\. Choose **Next**\.

1. On the **Name** page, enter a name for your connection rule; for example, `VPN to AWS Tunnel 1`, and then choose **Finish**\.

Repeat the above procedure, specifying the data for Tunnel 2 from your configuration file\. 

After you've finished, you’ll have two tunnels configured for your VPN connection\.

#### 2\.3: Confirm the Tunnel Configuration<a name="cgw-win2012-confirm-tunnel"></a>

**To confirm the tunnel configuration**

1. Open Server Manager, choose **Tools**, select **Windows Firewall with Advanced Security**, and then select **Connection Security Rules**\.

1. Verify the following for both tunnels:
   + **Enabled** is `Yes`
   + **Endpoint 1** is the CIDR block for your network
   + **Endpoint 2** is the CIDR block of your VPC
   + **Authentication mode** is `Require inbound and clear outbound`
   + **Authentication method** is `Custom`
   + **Endpoint 1 port** is `Any`
   + **Endpoint 2 port** is `Any`
   + **Protocol** is `Any`

1. Select the first rule and choose **Properties**\.

1. On the **Authentication** tab, under **Method**, choose **Customize**, and verify that **First authentication methods** contains the correct pre\-shared key from your configuration file for the tunnel, and then choose **OK**\.

1. On the **Advanced** tab, verify that **Domain**, **Private**, and **Public** are all selected\.

1. Under **IPsec tunneling**, choose **Customize**\. Verify the following IPsec tunneling settings, and then choose **OK** and **OK** again to close the dialog box\.
   + **Use IPsec tunneling** is selected\.
   + **Local tunnel endpoint \(closest to Endpoint 1\)** contains the IP address of your Windows server\. If your customer gateway device is an EC2 instance, this is the instance's private IP address\. 
   + **Remote tunnel endpoint \(closest to Endpoint 2\)** contains the IP address of the virtual private gateway for this tunnel\.

1. Open the properties for your second tunnel\. Repeat steps 4 to 7 for this tunnel\.

#### Enable Master Key Perfect Forward Secrecy<a name="cgw-win2012-enable-pfs"></a>

You can enable master key perfect forward secrecy by using the command line\. You cannot enable this feature using the user interface\.

**To enable master key perfect forward secrecy**

1. In your Windows server, open a new command prompt window\.

1. Type the following command, replacing `rule_name` with the name you gave the first connection rule\.

   ```
   netsh advfirewall consec set rule name="rule_name" new QMPFS=dhgroup2 QMSecMethods=ESP:SHA1-AES128+60min+100000kb
   ```

1. Repeat step 2 for the second tunnel, this time replacing `rule_name` with the name that you gave the second connection rule\.

### 2\.4: Configure the Windows Firewall<a name="cgw-win2012-firewall"></a>

After setting up your security rules on your server, configure some basic IPsec settings to work with the virtual private gateway\.

**To configure the Windows firewall**

1. Open Server Manager, choose **Tools**, select **Windows Firewall with Advanced Security**, and then choose **Properties**\.

1. On the **IPsec Settings** tab, under **IPsec exemptions**, verify that **Exempt ICMP from IPsec** is **No \(default\)**\. Verify that **IPsec tunnel authorization** is **None**\.

1. Under **IPsec defaults**, choose **Customize**\.

1. Under **Key exchange \(Main Mode\)**, select **Advanced** and then choose **Customize**\.

1. In **Customize Advanced Key Exchange Settings**, under **Security methods**, verify that these default values are used for the first entry\.
   + Integrity: SHA\-1
   + Encryption: AES\-CBC 128
   + Key exchange algorithm: Diffie\-Hellman Group 2
   + Under **Key lifetimes**, verify that **Minutes** is `480` and **Sessions** is `0`\.

   These settings correspond to these entries in the configuration file:

   ```
   MainModeSecMethods: DHGroup2-AES128-SHA1,DHGroup2-3DES-SHA1
   MainModeKeyLifetime: 480min,0sec
   ```

1. Under **Key exchange options**, select **Use Diffie\-Hellman for enhanced security**, and then choose **OK**\.

1. Under **Data protection \(Quick Mode\)**, select **Advanced**, and then choose **Customize**\.

1. Select **Require encryption for all connection security rules that use these settings**\.

1. Under **Data integrity and encryption**, leave the default values:
   + Protocol: ESP
   + Integrity: SHA\-1
   + Encryption: AES\-CBC 128
   + Lifetime: 60 minutes

   These values correspond to the following entry from the configuration file\.

   ```
   QuickModeSecMethods: 
   ESP:SHA1-AES128+60min+100000kb
   ```

1. Choose **OK** to return to the **Customize IPsec Settings** dialog box and choose **OK** again to save the configuration\.

## Step 5: Enable Dead Gateway Detection<a name="cgw-win2012-gateway-detection"></a>

Next, configure TCP to detect when a gateway becomes unavailable\. You can do this by modifying this registry key: `HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters`\. Do not perform this step until you’ve completed the preceding sections\. After you change the registry key, you must reboot the server\.

**To enable dead gateway detection**

1. From your Windows server, launch the command prompt or a PowerShell session, and type **regedit** to start Registry Editor\.

1. Expand **HKEY\_LOCAL\_MACHINE**, expand **SYSTEM**, expand **CurrentControlSet**, expand **Services**, expand **Tcpip**, and then expand **Parameters**\.

1. From the **Edit** menu, select **New** and select **DWORD \(32\-bit\) Value**\.

1. Enter the name **EnableDeadGWDetect**\.

1. Select **EnableDeadGWDetect** and choose **Edit**, **Modify**\.

1. In **Value data**, enter **1**, and then choose **OK**\.

1. Close the Registry Editor and reboot the server\.

For more information, see [EnableDeadGWDetect](http://technet.microsoft.com/en-us/library/cc960464.aspx) in the *Microsoft TechNet Library*\.

## Step 6: Test the VPN Connection<a name="cgw-win2012-test-connection"></a>

To test that the VPN connection is working correctly, launch an instance into your VPC, and ensure that it does not have an internet connection\. After you've launched the instance, ping its private IP address from your Windows server\. The VPN tunnel comes up when traffic is generated from the customer gateway device, therefore the ping command also initiates the VPN connection\.

**To launch an instance in your VPC and get its private IP address**

1. Open the Amazon EC2 console, and choose **Launch Instance**\. 

1. Select an Amazon Linux AMI, and select an instance type\.

1. On the **Step 3: Configure Instance Details** page, select your VPC from the **Network** list, and select a subnet from the **Subnet** list\. Ensure that you select the private subnet that you configured in [ Step 1: Create a VPN Connection and Configure Your VPC](#cgw-win2012-create-vpn)\. 

1. In the **Auto\-assign Public IP** list, ensure that the setting is set to **Disable**\.

1. Choose **Next** until you get to the** Step 6: Configure Security Group** page\. You can select an existing security group that you configured in [ Step 1: Create a VPN Connection and Configure Your VPC](#cgw-win2012-create-vpn)\. Or, you can create a new security group and ensure that it has a rule that allows all ICMP traffic from the IP address of your Windows server\.

1. Complete the rest of the steps in the wizard, and launch your instance\. 

1. On the **Instances** page, select your instance\. For **Private IPs**, note the private IP address on the details pane\.

Connect to or log on to your Windows server, open the command prompt, and then use the `ping` command to ping your instance using its private IP address; for example:

```
ping 10.0.0.4
```

```
Pinging 10.0.0.4 with 32 bytes of data:
Reply from 10.0.0.4: bytes=32 time=2ms TTL=62
Reply from 10.0.0.4: bytes=32 time=2ms TTL=62
Reply from 10.0.0.4: bytes=32 time=2ms TTL=62
Reply from 10.0.0.4: bytes=32 time=2ms TTL=62
		
Ping statistics for 10.0.0.4:
	Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
	Minimum = 2ms, Maximum = 2ms, Average = 2ms
```

If the `ping` command fails, check the following information:
+ Ensure that you have configured your security group rules to allow ICMP to the instance in your VPC\. If your Windows server is an EC2 instance, ensure that its security group's outbound rules allow IPsec traffic\. For more information, see [Configuring Your Windows Server](#cgw-win2012-prerequisites-windows-server)\.
+ Ensure that the operating system on the instance you are pinging is configured to respond to ICMP\. We recommend that you use one of the Amazon Linux AMIs\.
+ If the instance you are pinging is a Windows instance, connect to the instance and enable inbound ICMPv4 on the Windows firewall\.
+ Ensure that you have configured the route tables correctly for your VPC or your subnet\. For more information, see [ Step 1: Create a VPN Connection and Configure Your VPC](#cgw-win2012-create-vpn)\.
+ If your customer gateway device is a Windows server instance, ensure that you've disabled source/destination checking for the instance\. For more information, see [Configuring Your Windows Server](#cgw-win2012-prerequisites-windows-server)\.

In the Amazon VPC console, on the **VPN Connections** page, select your VPN connection\. The first tunnel is in the UP state\. The second tunnel should be configured, but it isn't used unless the first tunnel goes down\. It may take a few moments to establish the encrypted tunnels\.