# Troubleshooting Juniper ScreenOS Customer Gateway Connectivity<a name="Juniper_ScreenOs_Troubleshooting"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

When you troubleshoot the connectivity of a Juniper ScreenOS\-based customer gateway, consider four things: IKE, IPsec, tunnel, and BGP\. You can troubleshoot these areas in any order, but we recommend that you start with IKE \(at the bottom of the network stack\) and move up\. 

## IKE and IPsec<a name="IKEIPsec"></a>

Use the following command\. The response shows a customer gateway with IKE configured correctly\.

```
ssg5-serial-> get sa
```

```
total configured sa: 2
HEX ID    Gateway         Port Algorithm     SPI      Life:sec kb Sta   PID vsys
00000002<   72.21.209.225  500 esp:a128/sha1 80041ca4  3385 unlim A/-    -1 0
00000002>   72.21.209.225  500 esp:a128/sha1 8cdd274a  3385 unlim A/-    -1 0
00000001<   72.21.209.193  500 esp:a128/sha1 ecf0bec7  3580 unlim A/-    -1 0
00000001>   72.21.209.193  500 esp:a128/sha1 14bf7894  3580 unlim A/-    -1 0
```

You should see one or more lines containing a Remote Address of the Remote Gateway specified in the tunnels\. The `Sta` value should be `A/-` and `SPI` should be a hexadecimal number other than `00000000`\. Entries in other states indicate that IKE is not configured properly\.

For further troubleshooting, enable the IKE trace options \(as recommended in the example configuration information \(see [Example: Juniper ScreenOS Device](Juniper-with-screenos.md)\)\.

## Tunnel<a name="TunnelFirewall"></a>

First, double\-check that you have the necessary firewall rules in place\. For a list of the rules, see [Configuring a Firewall Between the Internet and Your Customer Gateway Device](Introduction.md#FirewallRules)\.

If your firewall rules are set up correctly, then continue troubleshooting with the following command:

```
ssg5-serial-> get interface tunnel.1
```

```
  Interface tunnel.1:
  description tunnel.1
  number 20, if_info 1768, if_index 1, mode route
  link ready
  vsys Root, zone Trust, vr trust-vr
  admin mtu 1500, operating mtu 1500, default mtu 1500
  *ip 169.254.255.2/30
  *manage ip 169.254.255.2
  route-deny disable
  bound vpn:
    IPSEC-1

  Next-Hop Tunnel Binding table
  Flag Status Next-Hop(IP)    tunnel-id  VPN

  pmtu-v4 disabled
  ping disabled, telnet disabled, SSH disabled, SNMP disabled
  web disabled, ident-reset disabled, SSL disabled

  OSPF disabled  BGP enabled  RIP disabled  RIPng disabled  mtrace disabled
  PIM: not configured  IGMP not configured
  NHRP disabled
  bandwidth: physical 0kbps, configured egress [gbw 0kbps mbw 0kbps]
             configured ingress mbw 0kbps, current bw 0kbps
             total allocated gbw 0kbps
```

Make sure that you see `link:ready`, and that the `IP` address matches the customer gateway tunnel inside address\.

Next, use the following command, replacing `169.254.255.1` with the inside IP address of your virtual private gateway\. Your results should look like the response shown here\.

```
ssg5-serial-> ping 169.254.255.1
```

```
Type escape sequence to abort

Sending 5, 100-byte ICMP Echos to 169.254.255.1, timeout is 1 seconds
!!!!!
Success Rate is 100 percent (5/5), round-trip time min/avg/max=32/32/33 ms
```

For further troubleshooting, review the configuration\.

## BGP<a name="BGPCommand"></a>

Use the following command:

```
ssg5-serial-> get vrouter trust-vr protocol bgp neighbor
```

```
Peer AS Remote IP       Local IP          Wt Status   State     ConnID Up/Down
--------------------------------------------------------------------------------
   7224 169.254.255.1   169.254.255.2    100 Enabled  ESTABLISH     10 00:01:01
   7224 169.254.255.5   169.254.255.6    100 Enabled  ESTABLISH     11 00:00:59
```

Both BGP peers should be listed as State: `ESTABLISH`, which means the BGP connection to the virtual private gateway is active\.

For further troubleshooting, use the following command, replacing `169.254.255.1` with the inside IP address of your virtual private gateway\. 

```
ssg5-serial-> get vr trust-vr prot bgp neigh 169.254.255.1
```

```
peer: 169.254.255.1,  remote AS: 7224, admin status: enable
type: EBGP, multihop: 0(disable), MED: node default(0)
connection state: ESTABLISH, connection id: 18 retry interval: node default(120s), cur retry time 15s
configured hold time: node default(90s), configured keepalive: node default(30s)
configured adv-interval: default(30s)
designated local IP: n/a
local IP address/port: 169.254.255.2/13946, remote IP address/port: 169.254.255.1/179
router ID of peer: 169.254.255.1, remote AS: 7224
negotiated hold time: 30s, negotiated keepalive interval: 10s
route map in name: , route map out name:
weight: 100 (default)
self as next hop: disable
send default route to peer: disable
ignore default route from peer: disable
send community path attribute: no
reflector client: no
Neighbor Capabilities:
  Route refresh: advertised and received
  Address family IPv4 Unicast:  advertised and received
force reconnect is disable
total messages to peer: 106, from peer: 106
update messages to peer: 6, from peer: 4
Tx queue length 0, Tx queue HWM: 1
route-refresh messages to peer: 0, from peer: 0
last reset 00:05:33 ago, due to BGP send Notification(Hold Timer Expired)(code 4 : subcode 0)
number of total successful connections: 4
connected: 2 minutes 6 seconds
Elapsed time since last update: 2 minutes 6 seconds
```

If the BGP peering is up, verify that your customer gateway router is advertising the default route \(0\.0\.0\.0/0\) to the VPC\. This command applies to ScreenOS version 6\.2\.0 and higher\.

```
ssg5-serial-> get vr trust-vr protocol bgp  rib neighbor 169.254.255.1 advertised
```

```
i: IBGP route, e: EBGP route, >: best route, *: valid route
               Prefix         Nexthop    Wt  Pref   Med Orig    AS-Path
--------------------------------------------------------------------------------------
>i          0.0.0.0/0         0.0.0.0 32768   100     0  IGP
Total IPv4 routes advertised: 1
```

Additionally, ensure that you're receiving the prefix corresponding to your VPC from the virtual private gateway\. This command applies to ScreenOS version 6\.2\.0 and higher\.

```
ssg5-serial-> get vr trust-vr protocol bgp  rib neighbor 169.254.255.1 received
```

```
i: IBGP route, e: EBGP route, >: best route, *: valid route
               Prefix         Nexthop    Wt  Pref   Med Orig    AS-Path
--------------------------------------------------------------------------------------
>e*     10.0.0.0/16   169.254.255.1   100   100   100  IGP   7224
Total IPv4 routes received: 1
```

## Virtual Private Gateway Attachment<a name="VGWAttachment"></a>

Make sure that your virtual private gateway is attached to your VPC\. Your integration team does this with the AWS Management Console\.

If you have questions or need further assistance, please use the [Amazon VPC forum](https://forums.aws.amazon.com/forum.jspa?forumID=58)\. 