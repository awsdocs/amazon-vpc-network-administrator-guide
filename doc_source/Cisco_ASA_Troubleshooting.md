# Troubleshooting Cisco ASA Customer Gateway Connectivity<a name="Cisco_ASA_Troubleshooting"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

When you troubleshoot the connectivity of a Cisco customer gateway, consider three things: IKE, IPsec, and routing\. You can troubleshoot these areas in any order, but we recommend that you start with IKE \(at the bottom of the network stack\) and move up\.

**Important**  
Some Cisco ASAs only support Active/Standby mode\. When you use these Cisco ASAs, you can have only one active tunnel at a time\. The other standby tunnel becomes active only if the first tunnel becomes unavailable\. The standby tunnel may produce the following error in your log files, which can be ignored: `Rejecting IPSec tunnel: no matching crypto map entry for remote proxy 0.0.0.0/0.0.0.0/0/0 local proxy 0.0.0.0/0.0.0.0/0/0 on interface outside` 

## IKE<a name="ASA_IKE"></a>

Use the following command\. The response shows a customer gateway with IKE configured correctly\.

```
ciscoasa# show crypto isakmp sa
```

```
   Active SA: 2
   Rekey SA: 0 (A tunnel will report 1 Active and 1 Rekey SA during rekey)
Total IKE SA: 2

1   IKE Peer: AWS_ENDPOINT_1
    Type    : L2L             Role    : initiator
    Rekey   : no              State   : MM_ACTIVE
```

You should see one or more lines containing an `src` value for the remote gateway specified in the tunnels\. The `state` value should be `MM_ACTIVE` and `status` should be `ACTIVE`\. The absence of an entry, or any entry in another state, indicates that IKE is not configured properly\.

For further troubleshooting, run the following commands to enable log messages that provide diagnostic information\.

```
router# term mon
router# debug crypto isakmp
```

To disable debugging, use the following command:

```
router# no debug crypto isakmp
```

## IPsec<a name="ASA_IPsec"></a>

Use the following command\. The response shows a customer gateway with IPsec configured correctly\.

```
ciscoasa# show crypto ipsec sa
```

```
interface: outside
    Crypto map tag: VPN_crypto_map_name, seq num: 2, local addr: 172.25.50.101

      access-list integ-ppe-loopback extended permit ip any vpc_subnet subnet_mask
      local ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/0/0)
      remote ident (addr/mask/prot/port): (vpc_subnet/subnet_mask/0/0)
      current_peer: integ-ppe1

      #pkts encaps: 0, #pkts encrypt: 0, #pkts digest: 0
      #pkts decaps: 0, #pkts decrypt: 0, #pkts verify: 0
      #pkts compressed: 0, #pkts decompressed: 0
      #pkts not compressed: 0, #pkts comp failed: 0, #pkts decomp failed: 0
      #pre-frag successes: 0, #pre-frag failures: 0, #fragments created: 0
      #PMTUs sent: 0, #PMTUs rcvd: 0, #decapsulated frgs needing reassembly: 0
      #send errors: 0, #recv errors: 0

      local crypto endpt.: 172.25.50.101, remote crypto endpt.: AWS_ENDPOINT_1

      path mtu 1500, ipsec overhead 74, media mtu 1500
      current outbound spi: 6D9F8D3B
      current inbound spi : 48B456A6

    inbound esp sas:
      spi: 0x48B456A6 (1219778214)
         transform: esp-aes esp-sha-hmac no compression
         in use settings ={L2L, Tunnel, PFS Group 2, }
         slot: 0, conn_id: 4710400, crypto-map: VPN_cry_map_1
         sa timing: remaining key lifetime (kB/sec): (4374000/3593)
         IV size: 16 bytes
         replay detection support: Y
         Anti replay bitmap:
          0x00000000 0x00000001
    outbound esp sas:
      spi: 0x6D9F8D3B (1839172923)
         transform: esp-aes esp-sha-hmac no compression
         in use settings ={L2L, Tunnel, PFS Group 2, }
         slot: 0, conn_id: 4710400, crypto-map: VPN_cry_map_1
         sa timing: remaining key lifetime (kB/sec): (4374000/3593)
         IV size: 16 bytes
         replay detection support: Y
         Anti replay bitmap:
         0x00000000 0x00000001
```

For each tunnel interface, you should see both `inbound esp sas` and `outbound esp sas`\. This assumes that an SA is listed \(for example, `spi: 0x48B456A6`\), and IPsec is configured correctly\.

In Cisco ASA, the IPsec only comes up after "interesting traffic" is sent\. To always keep the IPsec active, we recommend configuring SLA monitor\. SLA monitor continues to send interesting traffic, keeping the IPsec active\.

You can also use the following ping command to force your IPsec to start negotiation and go up:

```
ping ec2_instance_ip_address
```

```
Pinging ec2_instance_ip_address with 32 bytes of data:

Reply from ec2_instance_ip_address: bytes=32 time<1ms TTL=128
Reply from ec2_instance_ip_address: bytes=32 time<1ms TTL=128
Reply from ec2_instance_ip_address: bytes=32 time<1ms TTL=128

Ping statistics for 10.0.0.4:
Packets: Sent = 3, Received = 3, Lost = 0 (0% loss),

Approximate round trip times in milliseconds:
Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

For further troubleshooting, use the following command to enable debugging:

```
router# debug crypto ipsec
```

To disable debugging, use the following command:

```
router# no debug crypto ipsec
```

## Routing<a name="ASA_Tunnel"></a>

Ping the other end of the tunnel\. If this is working, then your IPsec should be up and running fine\. If this is not working, check your access lists, and refer the previous IPsec section\.

If you are not able to reach your instances, check the following:

1. Verify that the access\-list is configured to allow traffic that is associated with the crypto map\.

   You can do this using the following command:

   ```
   ciscoasa# show run crypto
   ```

   ```
   crypto ipsec transform-set transform-amzn esp-aes esp-sha-hmac
   crypto map VPN_crypto_map_name 1 match address access-list-name
   crypto map VPN_crypto_map_name 1 set pfs
   crypto map VPN_crypto_map_name 1 set peer AWS_ENDPOINT_1 AWS_ENDPOINT_2
   crypto map VPN_crypto_map_name 1 set transform-set transform-amzn
   crypto map VPN_crypto_map_name 1 set security-association lifetime seconds 3600
   ```

1. Next, check the access list as follows:

   ```
   ciscoasa# show run access-list access-list-name
   ```

   ```
   access-list access-list-name extended permit ip any vpc_subnet subnet_mask
   ```

   For example:

   ```
   access-list access-list-name extended permit ip any 10.0.0.0 255.255.0.0
   ```

1. Verify that this access list is correct\. The example access list in the previous step allows all internal traffic to the VPC subnet 10\.0\.0\.0/16\.

1. Run a traceroute from the Cisco ASA device, to see if it reaches the Amazon routers \(for example, *AWS\_ENDPOINT\_1*/*AWS\_ENDPOINT\_2*\)\.

   If this reaches the Amazon router, then check the static routes that you added in the AWS Management Console, and also the security groups for the particular instances\.

1. For further troubleshooting, review the configuration\.