# cisco XR - juniper MX interworking scenarios L3VPN, L2VPN H-VPLS

![image info](./topology.jpg)

```
summary:
reachability in vrf A is ok between cisco xr and vmx routers in the domain.
vmx domain is seemless mpls, core (isis) and aggregation (ospf) have no redistribution between them.
core ASBR (vmx4) is bgp RR towards the aggregation. :)

it is an example config for L3VPN and L2VPN H-VPLS.
mpls is simple ldp not SR-MPLS.

L3VPN: 
  every xr,vmx has a vrf loopback (xr:lo100, juni:lo0.100), they can reach each-other.

H-VPLS:
                           ce2
                            |
 ce1 - xr4 - xr2 - vmx2 - vmx4 - vmx6 - ce3
        |-------------------|------|
                xconn         VPLS

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
H-VPLS:

RP/0/RP0/CPU0:xrv4#sh l2vpn xconnect 
Sun Dec  7 19:56:11.295 UTC
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

root@vmx4> show vpls connections               
Layer-2 VPN connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NC -- interface encapsulation not CCC/TCC/VPLS
EM -- encapsulation mismatch     WE -- interface and instance encaps not same
VC-Dn -- Virtual circuit down    NP -- interface hardware not present 
CM -- control-word mismatch      -> -- only outbound connection is up
CN -- circuit not provisioned    <- -- only inbound connection is up
OR -- out of range               Up -- operational
OL -- no outgoing label          Dn -- down                      
LD -- local site signaled down   CF -- call admission control failure      
RD -- remote site signaled down  SC -- local and remote site ID collision
LN -- local site not designated  LM -- local site ID not minimum designated
RN -- remote site not designated RM -- remote site ID not minimum designated
XX -- unknown connection status  IL -- no incoming label
MM -- MTU mismatch               MI -- Mesh-Group ID not available
BK -- Backup connection          ST -- Standby connection
PF -- Profile parse failure      PB -- Profile busy
RS -- remote site standby        SN -- Static Neighbor
LB -- Local site not best-site   RB -- Remote site not best-site
VM -- VLAN ID mismatch           HS -- Hot-standby Connection

Legend for interface status 
Up -- operational                       
Dn -- down

Instance: VPLS1
Edge protection: Not-Primary
  BGP-VPLS State
  Local site: SITE2 (2)
    connection-site           Type  St     Time last up          # Up trans
    3                         rmt   Up     Dec  7 19:32:45 2025           1
      Remote PE: 1.1.56.6, Negotiated control-word: No
      Incoming label: 27, Outgoing label: 19
      Local interface: lsi.1048577, Status: Up, Encapsulation: VPLS
        Description: Intf - vpls VPLS1 local site 2 remote site 3
      Flow Label Transmit: No, Flow Label Receive: No
  LDP-VPLS State
  VPLS-id: 123
  Mesh-group connections: L2CIRCUITS
    Neighbor                  Type  St     Time last up          # Up trans
    1.1.1.4(vpls-id 1232)     rmt   Up     Dec  7 19:23:57 2025           1
      Remote PE: 1.1.1.4, Negotiated control-word: No
      Incoming label: 35, Outgoing label: 24005
      Negotiated PW status TLV: No
      Local interface: lsi.1048576, Status: Up, Encapsulation: ETHERNET
        Description: Intf - vpls VPLS1 neighbor 1.1.1.4 vpls-id 1232
      Flow Label Transmit: No, Flow Label Receive: No
    1.1.1.4(vpls-id 123)      rmt   OL   

root@vmx4> 

root@vmx6> show vpls connections 
Layer-2 VPN connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NC -- interface encapsulation not CCC/TCC/VPLS
EM -- encapsulation mismatch     WE -- interface and instance encaps not same
VC-Dn -- Virtual circuit down    NP -- interface hardware not present 
CM -- control-word mismatch      -> -- only outbound connection is up
CN -- circuit not provisioned    <- -- only inbound connection is up
OR -- out of range               Up -- operational
OL -- no outgoing label          Dn -- down                      
LD -- local site signaled down   CF -- call admission control failure      
RD -- remote site signaled down  SC -- local and remote site ID collision
LN -- local site not designated  LM -- local site ID not minimum designated
RN -- remote site not designated RM -- remote site ID not minimum designated
XX -- unknown connection status  IL -- no incoming label
MM -- MTU mismatch               MI -- Mesh-Group ID not available
BK -- Backup connection          ST -- Standby connection
PF -- Profile parse failure      PB -- Profile busy
RS -- remote site standby        SN -- Static Neighbor
LB -- Local site not best-site   RB -- Remote site not best-site
VM -- VLAN ID mismatch           HS -- Hot-standby Connection

Legend for interface status 
Up -- operational                       
Dn -- down

Instance: VPLS1
Edge protection: Not-Primary
  BGP-VPLS State
  Local site: SITE3 (3)
    connection-site           Type  St     Time last up          # Up trans
    2                         rmt   Up     Dec  7 19:32:45 2025           1
      Remote PE: 1.1.11.4, Negotiated control-word: No
      Incoming label: 19, Outgoing label: 27
      Local interface: lsi.1048576, Status: Up, Encapsulation: VPLS
        Description: Intf - vpls VPLS1 local site 3 remote site 2
      Flow Label Transmit: No, Flow Label Receive: No
  LDP-VPLS State
  VPLS-id: 123

root@vmx6> 

ce1#sh ip int brief | inc 22.22.22
FastEthernet2/0        22.22.22.1      YES manual up                    up      

ce1#ping 22.22.22.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 22.22.22.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 68/72/80 ms
ce1#ping 22.22.22.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 22.22.22.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 64/69/76 ms
ce1#

ce1#sh arp 
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  22.22.22.1              -   ca0d.ff1e.0038  ARPA   FastEthernet2/0
Internet  22.22.22.2             22   ca01.fdf1.0000  ARPA   FastEthernet2/0
Internet  22.22.22.3             22   ca13.0a28.0038  ARPA   FastEthernet2/0
ce1#

root@vmx4> show vpls mac-table 

MAC flags       (S -static MAC, D -dynamic MAC, L -locally learned, C -Control MAC
    O -OVSDB MAC, SE -Statistics enabled, NM -Non configured MAC, R -Remote PE MAC, P -Pinned MAC)

Routing instance : VPLS1
 Bridging domain : __VPLS1__, VLAN : NA
   MAC                 MAC      Logical          NH     MAC         active
   address             flags    interface        Index  property    source
   ca:01:fd:f1:00:00   D        ge-0/0/2.0      
   ca:0d:ff:1e:00:38   D        lsi.1048576     
   ca:13:0a:28:00:38   D        lsi.1048577     

root@vmx6> show vpls mac-table 

MAC flags       (S -static MAC, D -dynamic MAC, L -locally learned, C -Control MAC
    O -OVSDB MAC, SE -Statistics enabled, NM -Non configured MAC, R -Remote PE MAC, P -Pinned MAC)

Routing instance : VPLS1
 Bridging domain : __VPLS1__, VLAN : NA
   MAC                 MAC      Logical          NH     MAC         active
   address             flags    interface        Index  property    source
   ca:01:fd:f1:00:00   D        lsi.1048576     
   ca:0d:ff:1e:00:38   D        lsi.1048576     
   ca:13:0a:28:00:38   D        ge-0/0/2.0      

```




