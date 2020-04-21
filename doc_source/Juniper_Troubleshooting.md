# Troubleshooting Juniper JunOS Customer Gateway Connectivity<a name="Juniper_Troubleshooting"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

When you troubleshoot the connectivity of a Juniper customer gateway, consider four things: IKE, IPsec, tunnel, and BGP\. You can troubleshoot these areas in any order, but we recommend that you start with IKE \(at the bottom of the network stack\) and move up\. 

## IKE<a name="IKETroubleshooting"></a>

Use the following command\. The response shows a customer gateway with IKE configured correctly\.

```
user@router> show security ike security-associations
```

```
Index   Remote Address  State  Initiator cookie  Responder cookie  Mode
4       72.21.209.225   UP     c4cd953602568b74  0d6d194993328b02  Main
3       72.21.209.193   UP     b8c8fb7dc68d9173  ca7cb0abaedeb4bb  Main
```

You should see one or more lines containing a Remote Address of the Remote Gateway specified in the tunnels\. The `State` should be `UP`\. The absence of an entry, or any entry in another state \(such as `DOWN`\) is an indication that IKE is not configured properly\.

For further troubleshooting, enable the IKE trace options \(as recommended in the example configuration information \(see [Example: Juniper J\-Series JunOS Device](Juniper.md)\)\. Then run the following command to print a variety of debugging messages to the screen\.

```
user@router> monitor start kmd
```

From an external host, you can retrieve the entire log file with the following command:

```
scp username@router.hostname:/var/log/kmd
```

## IPsec<a name="IPsecTroubleshooting"></a>

Use the following command\. The response shows a customer gateway with IPsec configured correctly\.

```
user@router> show security ipsec security-associations
```

```
Total active tunnels: 2
ID      Gateway        Port  Algorithm        SPI      Life:sec/kb Mon vsys
<131073 72.21.209.225  500   ESP:aes-128/sha1 df27aae4 326/ unlim   -   0
>131073 72.21.209.225  500   ESP:aes-128/sha1 5de29aa1 326/ unlim   -   0
<131074 72.21.209.193  500   ESP:aes-128/sha1 dd16c453 300/ unlim   -   0
>131074 72.21.209.193  500   ESP:aes-128/sha1 c1e0eb29 300/ unlim   -   0
```

Specifically, you should see at least two lines per Gateway address \(corresponding to the Remote Gateway\)\. Note the carets at the beginning of each line \(< >\) which indicate the direction of traffic for the particular entry\. The output has separate lines for inbound traffic \("<", traffic from the virtual private gateway to this customer gateway\) and outbound traffic \(">"\)\.

For further troubleshooting, enable the IKE traceoptions \(for more information, see the preceding section about IKE\)\. 

## Tunnel<a name="TunnelTroubleshooting"></a>

First, double\-check that you have the necessary firewall rules in place\. For a list of the rules, see [Configuring a Firewall Between the Internet and Your Customer Gateway Device](Introduction.md#FirewallRules)\.

If your firewall rules are set up correctly, then continue troubleshooting with the following command:

```
user@router> show interfaces st0.1
```

```
 Logical interface st0.1 (Index 70) (SNMP ifIndex 126)
    Flags: Point-To-Point SNMP-Traps Encapsulation: Secure-Tunnel
    Input packets : 8719
    Output packets: 41841
    Security: Zone: Trust
    Allowed host-inbound traffic : bgp ping ssh traceroute
    Protocol inet, MTU: 9192
      Flags: None
      Addresses, Flags: Is-Preferred Is-Primary
      Destination: 169.254.255.0/30, Local: 169.254.255.2
```

Make sure that the `Security: Zone` is correct, and that the `Local` address matches the customer gateway tunnel inside address\.

Next, use the following command, replacing `169.254.255.1` with the inside IP address of your virtual private gateway\. Your results should look like the response shown here\.

```
user@router> ping 169.254.255.1 size 1382 do-not-fragment
```

```
PING 169.254.255.1 (169.254.255.1): 1410 data bytes
64 bytes from 169.254.255.1: icmp_seq=0 ttl=64 time=71.080 ms
64 bytes from 169.254.255.1: icmp_seq=1 ttl=64 time=70.585 ms
```

For further troubleshooting, review the configuration\.

## BGP<a name="BGPTroubleshooting"></a>

Use the following command:

```
user@router> show bgp summary
```

```
Groups: 1 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0                 2          1          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
169.254.255.1          7224          9         10       0       0        1:00 1/1/1/0              0/0/0/0
169.254.255.5          7224          8          9       0       0          56 0/1/1/0              0/0/0/0
```

For further troubleshooting, use the following command, replacing `169.254.255.1` with the inside IP address of your virtual private gateway\. 

```
user@router> show bgp neighbor 169.254.255.1
```

```
Peer: 169.254.255.1+179 AS 7224 Local: 169.254.255.2+57175 AS 65000
  Type: External    State: Established    Flags: <ImportEval Sync>
  Last State: OpenConfirm   Last Event: RecvKeepAlive
  Last Error: None
  Export: [ EXPORT-DEFAULT ] 
  Options: <Preference HoldTime PeerAS LocalAS Refresh>
  Holdtime: 30 Preference: 170 Local AS: 65000 Local System AS: 0
  Number of flaps: 0
  Peer ID: 169.254.255.1    Local ID: 10.50.0.10       Active Holdtime: 30
  Keepalive Interval: 10         Peer index: 0   
  BFD: disabled, down
  Local Interface: st0.1                            
  NLRI for restart configured on peer: inet-unicast
  NLRI advertised by peer: inet-unicast
  NLRI for this session: inet-unicast
  Peer supports Refresh capability (2)
  Restart time configured on the peer: 120
  Stale routes from peer are kept for: 300
  Restart time requested by this peer: 120
  NLRI that peer supports restart for: inet-unicast
  NLRI that restart is negotiated for: inet-unicast
  NLRI of received end-of-rib markers: inet-unicast
  NLRI of all end-of-rib markers sent: inet-unicast
  Peer supports 4 byte AS extension (peer-as 7224)
  Table inet.0 Bit: 10000
    RIB State: BGP restart is complete
    Send state: in sync
    Active prefixes:              1
    Received prefixes:            1
    Accepted prefixes:            1
    Suppressed due to damping:    0
    Advertised prefixes:          1
Last traffic (seconds): Received 4    Sent 8    Checked 4   
Input messages:  Total 24     Updates 2       Refreshes 0     Octets 505
Output messages: Total 26     Updates 1       Refreshes 0     Octets 582
Output Queue[0]: 0
```

Here you should see `Received prefixes` and `Advertised prefixes` listed at 1 each\. This should be within the `Table inet.0` section\.

If the `State` is not `Established`, check the `Last State` and `Last Error` for details of what is required to correct the problem\.

If the BGP peering is up, verify that your customer gateway router is advertising the default route \(0\.0\.0\.0/0\) to the VPC\. 

```
user@router> show route advertising-protocol bgp 169.254.255.1
```

```
inet.0: 10 destinations, 11 routes (10 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 0.0.0.0/0               Self                                    I
```

Additionally, make sure that you're receiving the prefix corresponding to your VPC from the virtual private gateway\.

```
user@router> show route receive-protocol bgp 169.254.255.1
```

```
inet.0: 10 destinations, 11 routes (10 active, 0 holddown, 0 hidden)
  Prefix                  Nexthop              MED     Lclpref    AS path
* 10.110.0.0/16           169.254.255.1        100                7224 I
```

## Virtual Private Gateway Attachment<a name="VGWTroubleshooting"></a>

Make sure that your virtual private gateway is attached to your VPC\. Your integration team does this with the AWS Management Console\.

If you have questions or need further assistance, use the [Amazon VPC forum](https://forums.aws.amazon.com/forum.jspa?forumID=58)\. 