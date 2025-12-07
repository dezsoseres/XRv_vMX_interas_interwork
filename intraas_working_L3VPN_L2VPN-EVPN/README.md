# cisco XR - juniper MX interworking scenarios L3VPN, L2VPN EVPN

![image info](./topology.jpg)

```
summary:
reachability in vrf A is ok between cisco xr and vmx routers in the domain.
vmx domain is seemless mpls, core (isis) and aggregation (ospf) have no redistribution between them.
core ASBR (vmx4) is bgp RR towards the aggregation. :)

it is an example config for L3VPN and L2VPN EVPN.
mpls is simple ldp not SR-MPLS.

L3VPN: 
  every xr,vmx has a vrf loopback (xr:lo100, juni:lo0.100), they can reach each-other.

EVPN:
                           ce2
                            |
 ce1 - xr4 - xr2 - vmx2 - vmx4 - vmx6 - ce3
        |-------------------|------|
                xconn         EVPN

 all ce routers can ping each-others

```

```
L3VPN:

RP/0/RP0/CPU0:xrv4#sh route vrf A
Sun Dec  7 19:35:04.015 UTC

Codes: C - connected, S - static, R - RIP, B - BGP, (>) - Diversion path
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - ISIS, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, su - IS-IS summary null, * - candidate default
       U - per-user static route, o - ODR, L - local, G  - DAGR, l - LISP
       A - access/subscriber, a - Application route
       M - mobile route, r - RPL, t - Traffic Engineering, (!) - FRR Backup path

Gateway of last resort is not set

B    11.11.11.2/32 [200/0] via 1.1.1.2 (nexthop in vrf default), 00:26:38
L    11.11.11.4/32 is directly connected, 01:49:36, Loopback100
B    12.12.12.2/32 [200/0] via 1.1.11.2 (nexthop in vrf default), 00:25:24
B    12.12.12.4/32 [200/0] via 1.1.11.4 (nexthop in vrf default), 00:25:25
B    12.12.12.6/32 [200/1] via 1.1.11.4 (nexthop in vrf default), 00:20:54
RP/0/RP0/CPU0:xrv4#ping vrf A 11.11.11.2
Sun Dec  7 19:35:12.785 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 11.11.11.2 timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms
RP/0/RP0/CPU0:xrv4#ping vrf A 11.11.11.4
Sun Dec  7 19:35:14.593 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 11.11.11.4 timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
RP/0/RP0/CPU0:xrv4#ping vrf A 12.12.12.2
Sun Dec  7 19:35:20.823 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 12.12.12.2 timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
RP/0/RP0/CPU0:xrv4#ping vrf A 12.12.12.4
Sun Dec  7 19:35:27.080 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 12.12.12.4 timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
RP/0/RP0/CPU0:xrv4#ping vrf A 12.12.12.6
Sun Dec  7 19:35:28.664 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 12.12.12.6 timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/4 ms
RP/0/RP0/CPU0:xrv4#
```


```
EVPN:
                           ce2
                            |
 ce1 - xr4 - xr2 - vmx2 - vmx4 - vmx6 - ce3
        |-------------------|------|
                xconn         EVPN

 all ce routers can ping each-other.


RP/0/RP0/CPU0:xrv4# sh l2vpn xconnect 
Sun Dec  7 21:58:11.938 UTC
Legend: ST = State, UP = Up, DN = Down, AD = Admin Down, UR = Unresolved,
        SB = Standby, SR = Standby Ready, (PP) = Partially Programmed,
        LU = Local Up, RU = Remote Up, CO = Connected, (SI) = Seamless Inactive

XConnect                   Segment 1                       Segment 2                
Group      Name       ST   Description            ST       Description            ST    
------------------------   -----------------------------   -----------------------------
XCONNS_UZLETI
           XCONN112   UP   Gi0/0/0/2              UP       1.1.11.4        1232   UP    
----------------------------------------------------------------------------------------
RP/0/RP0/CPU0:xrv4#


simple trick on vmx4 with LT logical tunnel interface (hairpin inside the box)

vmx4
set chassis fpc 0 pic 0 tunnel-services bandwidth 10g
set chassis fpc 0 lite-mode

set interfaces lt-0/0/0 unit 0 encapsulation ethernet-bridge
set interfaces lt-0/0/0 unit 0 peer-unit 1
set interfaces lt-0/0/0 unit 0 family bridge
set interfaces lt-0/0/0 unit 1 encapsulation ethernet-ccc
set interfaces lt-0/0/0 unit 1 peer-unit 0
set interfaces lt-0/0/0 unit 1 family ccc

set protocols l2circuit neighbor 1.1.1.4 interface lt-0/0/0.1 virtual-circuit-id 1232
set protocols l2circuit neighbor 1.1.1.4 interface lt-0/0/0.1 control-word

set routing-instances EVPN1 instance-type evpn
set routing-instances EVPN1 protocols evpn
set routing-instances EVPN1 forwarding-options family evpn flood input EVPN_FLOOD_FILTER
set routing-instances EVPN1 interface lt-0/0/0.0
set routing-instances EVPN1 interface ge-0/0/2.0
set routing-instances EVPN1 route-distinguisher 1.1.11.4:1001
set routing-instances EVPN1 vrf-target target:100:1001

root@vmx4> show l2circuit connections
Layer-2 Circuit Connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NP -- interface h/w not present   
MM -- mtu mismatch               Dn -- down                       
EM -- encapsulation mismatch     VC-Dn -- Virtual circuit Down    
CM -- control-word mismatch      Up -- operational                
VM -- vlan id mismatch           CF -- Call admission control failure
OL -- no outgoing label          IB -- TDM incompatible bitrate 
NC -- intf encaps not CCC/TCC    TM -- TDM misconfiguration 
BK -- Backup Connection          ST -- Standby Connection
CB -- rcvd cell-bundle size bad  SP -- Static Pseudowire
LD -- local site signaled down   RS -- remote site standby
RD -- remote site signaled down  HS -- Hot-standby Connection
XX -- unknown

Legend for interface status  
Up -- operational            
Dn -- down                   
Neighbor: 1.1.1.4 
    Interface                 Type  St     Time last up          # Up trans
    lt-0/0/0.1(vc 1232)       rmt   Up     Dec  7 21:54:36 2025           1
      Remote PE: 1.1.1.4, Negotiated control-word: No
      Incoming label: 54, Outgoing label: 24005
      Negotiated PW status TLV: No
      Local interface: lt-0/0/0.1, Status: Up, Encapsulation: ETHERNET
      Flow Label Transmit: No, Flow Label Receive: No

root@vmx4> 

root@vmx4> show evpn database    
Instance: EVPN1
VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
                ca:01:fd:f1:00:00  ge-0/0/2.0                     Dec 07 21:51:45  22.22.22.2
                ca:0d:ff:1e:00:38  lt-0/0/0.0                     Dec 07 21:54:38
                ca:13:0a:28:00:38  1.1.56.6                       Dec 07 21:51:45  22.22.22.3

root@vmx6> show evpn database 
Instance: EVPN1
VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
                ca:01:fd:f1:00:00  1.1.11.4                       Dec 07 21:51:45  22.22.22.2
                ca:0d:ff:1e:00:38  1.1.11.4                       Dec 07 21:54:38
                ca:13:0a:28:00:38  ge-0/0/2.0                     Dec 07 21:51:45  22.22.22.3

root@vmx4> show evpn mac-table       

MAC flags       (S -static MAC, D -dynamic MAC, L -locally learned, C -Control MAC
    O -OVSDB MAC, SE -Statistics enabled, NM -Non configured MAC, R -Remote PE MAC, P -Pinned MAC)

Routing instance : EVPN1
 Bridging domain : __EVPN1__, VLAN : NA
   MAC                 MAC      Logical          NH     MAC         active
   address             flags    interface        Index  property    source
   ca:01:fd:f1:00:00   D        ge-0/0/2.0      
   ca:0d:ff:1e:00:38   D        lt-0/0/0.0      
   ca:13:0a:28:00:38   DC                        1048575            1.1.56.6                      

root@vmx6> show evpn mac-table 

MAC flags       (S -static MAC, D -dynamic MAC, L -locally learned, C -Control MAC
    O -OVSDB MAC, SE -Statistics enabled, NM -Non configured MAC, R -Remote PE MAC, P -Pinned MAC)

Routing instance : EVPN1
 Bridging domain : __EVPN1__, VLAN : NA
   MAC                 MAC      Logical          NH     MAC         active
   address             flags    interface        Index  property    source
   ca:01:fd:f1:00:00   DC                        1048575            1.1.11.4                      
   ca:0d:ff:1e:00:38   DC                        1048575            1.1.11.4                      
   ca:13:0a:28:00:38   D        ge-0/0/2.0 

ce1#sh ip int brief | inc 22.22
FastEthernet2/0        22.22.22.1      YES manual up                    up      

ce1#sh arp 
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  22.22.22.1              -   ca0d.ff1e.0038  ARPA   FastEthernet2/0
Internet  22.22.22.2            128   ca01.fdf1.0000  ARPA   FastEthernet2/0
Internet  22.22.22.3            128   ca13.0a28.0038  ARPA   FastEthernet2/0

ce1#ping 22.22.22.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 22.22.22.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/8 ms
ce1#ping 22.22.22.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 22.22.22.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/13/28 ms
ce1#ping 22.22.22.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 22.22.22.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/13/28 ms

```




