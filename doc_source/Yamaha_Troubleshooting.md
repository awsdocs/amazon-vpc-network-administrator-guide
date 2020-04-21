# Troubleshooting Yamaha Customer Gateway Connectivity<a name="Yamaha_Troubleshooting"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

When you troubleshoot the connectivity of a Yamaha customer gateway, consider four things: IKE, IPsec, tunnel, and BGP\. You can troubleshoot these areas in any order, but we recommend that you start with IKE \(at the bottom of the network stack\) and move up\.

## IKE<a name="YamahaIKE"></a>

Use the following command\. The response shows a customer gateway with IKE configured correctly\.

```
# show ipsec sa gateway 1
```

```
sgw  flags local-id                      remote-id        # of sa
--------------------------------------------------------------------------
1    U K   YOUR_LOCAL_NETWORK_ADDRESS     72.21.209.225    i:2 s:1 r:1
```

You should see a line containing a `remote-id` value for the Remote Gateway specified in the tunnels\. You can list all the security associations \(SAs\) by omitting the tunnel number\.

For further troubleshooting, run the following commands to enable DEBUG level log messages that provide diagnostic information\.

```
# syslog debug on
# ipsec ike log message-info payload-info key-info
```

To cancel the logged items, use the following command:

```
# no ipsec ike log
# no syslog debug on
```

## IPsec<a name="YamahaIPsec"></a>

Use the following command\. The response shows a customer gateway with IPsec configured correctly\.

```
# show ipsec sa gateway 1 detail
```

```
SA[1] Duration: 10675s
Local ID: YOUR_LOCAL_NETWORK_ADDRESS
Remote ID: 72.21.209.225
Protocol: IKE
Algorithm: AES-CBC, SHA-1, MODP 1024bit

SPI: 6b ce fd 8a d5 30 9b 02 0c f3 87 52 4a 87 6e 77 
Key: ** ** ** ** **  (confidential)   ** ** ** ** **
----------------------------------------------------
SA[2] Duration: 1719s
Local ID: YOUR_LOCAL_NETWORK_ADDRESS
Remote ID: 72.21.209.225
Direction: send
Protocol: ESP (Mode: tunnel)
Algorithm: AES-CBC (for Auth.: HMAC-SHA)
SPI: a6 67 47 47 
Key: ** ** ** ** **  (confidential)   ** ** ** ** **
----------------------------------------------------
SA[3] Duration: 1719s
Local ID: YOUR_LOCAL_NETWORK_ADDRESS
Remote ID: 72.21.209.225
Direction: receive
Protocol: ESP (Mode: tunnel)
Algorithm: AES-CBC (for Auth.: HMAC-SHA)
SPI: 6b 98 69 2b 
Key: ** ** ** ** **  (confidential)   ** ** ** ** **
----------------------------------------------------
SA[4] Duration: 10681s
Local ID: YOUR_LOCAL_NETWORK_ADDRESS
Remote ID: 72.21.209.225
Protocol: IKE
Algorithm: AES-CBC, SHA-1, MODP 1024bit
SPI: e8 45 55 38 90 45 3f 67 a8 74 ca 71 ba bb 75 ee 
Key: ** ** ** ** **  (confidential)   ** ** ** ** **
----------------------------------------------------
```

For each tunnel interface, you should see both `receive sas` and `send sas`\.

For further troubleshooting, use the following command to enable debugging\.

```
# syslog debug on
# ipsec ike log message-info payload-info key-info
```

Use the following command to disable debugging\.

```
# no ipsec ike log
# no syslog debug on
```

## Tunnel<a name="YamahaTunnel"></a>

First, check that you have the necessary firewall rules in place\. For a list of the rules, see [Configuring a Firewall Between the Internet and Your Customer Gateway Device](Introduction.md#FirewallRules)\.

If your firewall rules are set up correctly, then continue troubleshooting with the following command:

```
# show status tunnel 1
```

```
TUNNEL[1]: 
Description: 
  Interface type: IPsec
  Current status is Online.
  from 2011/08/15 18:19:45.
  5 hours 7 minutes 58 seconds  connection.
  Received:    (IPv4) 3933 packets [244941 octets]
               (IPv6) 0 packet [0 octet]
  Transmitted: (IPv4) 3933 packets [241407 octets]
               (IPv6) 0 packet [0 octet]
```

Make sure that the `current status` value is online and that `Interface type` is IPsec\. Make sure to run the command on both tunnel interfaces\. To resolve any problems here, review the configuration\.

## BGP<a name="YamahaBGP"></a>

Use the following command:

```
# show status bgp neighbor
```

```
BGP neighbor is 169.254.255.1, remote AS 7224, local AS 65000, external link
  BGP version 0, remote router ID 0.0.0.0
  BGP state = Active
  Last read 00:00:00, hold time is 0, keepalive interval is 0 seconds
  Received 0 messages, 0 notifications, 0 in queue
  Sent 0 messages, 0 notifications, 0 in queue
  Connection established 0; dropped 0
  Last reset never
Local host: unspecified
Foreign host: 169.254.255.1, Foreign port: 0

BGP neighbor is 169.254.255.5, remote AS 7224, local AS 65000, external link
  BGP version 0, remote router ID 0.0.0.0
  BGP state = Active
  Last read 00:00:00, hold time is 0, keepalive interval is 0 seconds
  Received 0 messages, 0 notifications, 0 in queue
  Sent 0 messages, 0 notifications, 0 in queue
  Connection established 0; dropped 0
  Last reset never
Local host: unspecified
Foreign host: 169.254.255.5, Foreign port:
```

Here, both neighbors should be listed\. For each, you should see a `BGP state` value of Active\.

If the BGP peering is up, verify that your customer gateway router is advertising the default route \(0\.0\.0\.0/0\) to the VPC\. 

```
# show status bgp neighbor 169.254.255.1 advertised-routes 
```

```
Total routes: 1
*: valid route
  Network            Next Hop        Metric LocPrf Path
* default            0.0.0.0              0        IGP
```

Additionally, ensure that you're receiving the prefix corresponding to your VPC from the virtual private gateway\. 

```
# show ip route
```

```
Destination         Gateway          Interface       Kind  Additional Info.
default             ***.***.***.***   LAN3(DHCP)    static  
10.0.0.0/16         169.254.255.1    TUNNEL[1]       BGP  path=10124
```

For further troubleshooting, review the configuration\.

## Virtual Private Gateway Attachment<a name="YamahaVGW"></a>

Make sure that your virtual private gateway is attached to your VPC\. Your integration team does this with the AWS Management Console\.

If you have questions or need further assistance, please use the [Amazon VPC forum](https://forums.aws.amazon.com/forum.jspa?forumID=58)\. 