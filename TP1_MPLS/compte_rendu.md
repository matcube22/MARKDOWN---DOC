# Configuration routeur
## Commandes
```bash
Router(config)#hostname Router-28
Router-28(config)#int GigabitEthernet 0/0/0
Router-28(config-if)#ip addr 100.0.28.28 255.255.255.0 
Router-28(config)#int GigabitEthernet 0/0 
Router-28(config)#int GigabitEthernet 0/0/1 
Router-28(config-if)#ip addr 10.28.28.254 255.255.255.0 
Router-28(config)#int loopback 28 
Router-28(config-if)#ip addr 10.10.10.28 255.255.255.255 
Router-28(config)#ip routing 
Router-28(config)#router ospf 1 
Router-28(config-router)#router-id 10.10.10.28 
Router-28(config-router)#redistribute connected subnets 
Router-28(config-router)#network 100.0.28.0 0.0.0.255 area 6 
Router-28(config)#router ospf 1 
Router-28(config-router)#network 10.28.28.0 0.0.0.255 area 6 
Router-28(config-router)#exit 
Router-28(config)#
```
## SH RUN Routeur
```bash
Router-28(config)#do sh run
Building configuration...


Current configuration : 1891 bytes

version 15.5
service timestamps debug datetime msec
service timestamps log datetime msec
no platform punt-keepalive disable-kernel-core

hostname Router-28

boot-start-marker
boot-end-marker

vrf definition Mgmt-intf
 
 address-family ipv4
 exit-address-family
 
 address-family ipv6
 exit-address-family

no aaa new-model

subscriber templating

multilink bundle-name authenticated

voice-card 0/4
 no watchdog

license udi pid ISR4331/K9 sn FDO223234MV
license boot level appxk9

spanning-tree extend system-id

redundancy
 mode none

vlan internal allocation policy ascending

interface Loopback28
 ip address 10.10.10.28 255.255.255.255

interface GigabitEthernet0/0/0
 ip address 100.0.28.28 255.255.255.0
 no ip proxy-arp
 negotiation auto

interface GigabitEthernet0/0/1
 ip address 10.28.28.254 255.255.255.0
 no ip proxy-arp
 negotiation auto

interface GigabitEthernet0/0/2
 no ip address
 no ip proxy-arp
 shutdown
 negotiation auto

interface Serial0/1/0
 no ip address
 shutdown

interface Service-Engine0/4/0

interface GigabitEthernet0
 vrf forwarding Mgmt-intf
 no ip address
 negotiation auto

interface Vlan1
 no ip address
 shutdown 

router ospf 1
 router-id 10.10.10.28
 redistribute connected subnets
 network 10.28.28.0 0.0.0.255 area 6
 network 100.0.28.0 0.0.0.255 area 6

ip forward-protocol nd
no ip http server
no ip http secure-server
ip tftp source-interface GigabitEthernet0/0/0

control-plane

mgcp behavior rsip-range tgcp-only
mgcp behavior comedia-role none
mgcp behavior comedia-check-media-src disable
mgcp behavior comedia-sdp-force disable

mgcp profile default


line con 0
 exec-timeout 0 0
 stopbits 1
line aux 0
 stopbits 1
line vty 0 4
 login

end
```
# Configuration switch
## Commandes
```bash
Switch(config)#hostname Switch-28
Switch-28(config)#monitor session 1 source interface FastEthernet 0/1 both 
Switch-28(config)#monitor session 1 source interface FastEthernet 0/24 both 
Switch-28(config)#monitor session 1 destination interface FastEthernet 0/23 
```
## SH RUN SWITCH
```bash
Switch-28(config)#do sh run
Building configuration...

Current configuration : 1337 bytes

version 15.0
no service pad
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption

hostname Switch-28

boot-start-marker
boot-end-marker

no aaa new-model
system mtu routing 1500
 
spanning-tree mode pvst
spanning-tree extend system-id

vlan internal allocation policy ascending


interface FastEthernet0/1


interface Vlan1
 no ip address

ip http server
ip http secure-server
logging esm config

line con 0
line vty 5 15

monitor session 1 source interface Fa0/1 , Fa0/24
monitor session 1 destination interface Fa0/23
end

```
# OSPF 1
## Table de routage
```bash
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
O        10.0.12.0/24 [110/3] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        10.0.23.0/24 [110/3] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        10.0.34.0/24 [110/2] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        10.0.41.0/24 [110/2] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        10.10.10.1/32 
           [110/3] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        10.10.10.2/32 
           [110/4] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        10.10.10.3/32 
           [110/3] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        10.10.10.4/32 
           [110/2] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O E2     10.10.10.25/32 
           [110/20] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O E2     10.10.10.26/32 
           [110/20] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
C        10.10.10.28/32 is directly connected, Loopback28
O E2     10.25.25.0/24 
           [110/20] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O E2     10.26.26.0/24 
           [110/20] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
C        10.28.28.0/24 is directly connected, GigabitEthernet0/0/1
L        10.28.28.254/32 is directly connected, GigabitEthernet0/0/1
      100.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
O        100.0.25.0/24 
           [110/12] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        100.0.26.0/24 
           [110/3] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
O        100.0.27.0/24 
           [110/3] via 100.0.28.254, 00:02:11, GigabitEthernet0/0/0
C        100.0.28.0/24 is directly connected, GigabitEthernet0/0/0
L        100.0.28.28/32 is directly connected, GigabitEthernet0/0/0
```
## Reponse aux questions
DR: 100.0.28.28 => Routeur ISR
BDR: 100.0.28.254 => LSR 4
DB decription: 100.0.28.28 et 100.0.28.254
LS update :100.0.28.254 (c'est lui qui nous l'envoie)

# Mise en place de la commutation de label
## Commandes
```bash
Router-28(config)#ip cef distributed

# Commande 1
Router-28(config)#do sh ip cef 10.0.12.0 detail
10.0.12.0/24, epoch 1
  nexthop 100.0.28.254 GigabitEthernet0/0/0

# Commande 2
Router-28(config)#do sh ip cef 10.0.34.0 detail
10.0.34.0/24, epoch 1
  nexthop 100.0.28.254 GigabitEthernet0/0/0

```
## Reponse aux questions
Ils ont la même adresse de nexthop et que la valeur de epoch vaut 1

## Activation de mpls
```bash
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 19 subnets, 2 masks
O        10.0.12.0/24 [110/3] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        10.0.23.0/24 [110/3] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        10.0.34.0/24 [110/2] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        10.0.41.0/24 [110/2] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        10.10.10.1/32 
           [110/3] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        10.10.10.2/32 
           [110/4] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        10.10.10.3/32 
           [110/3] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        10.10.10.4/32 
           [110/2] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O E2     10.10.10.24/32 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O E2     10.10.10.25/32 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O E2     10.10.10.26/32 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O E2     10.10.10.27/32 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
C        10.10.10.28/32 is directly connected, Loopback28
O E2     10.24.24.0/24 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O E2     10.25.25.0/24 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O E2     10.26.26.0/24 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O E2     10.27.27.0/24 
           [110/20] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
C        10.28.28.0/24 is directly connected, GigabitEthernet0/0/1
L        10.28.28.254/32 is directly connected, GigabitEthernet0/0/1
      100.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O        100.0.24.0/24 
           [110/12] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        100.0.25.0/24 
           [110/12] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        100.0.26.0/24 
           [110/3] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
O        100.0.27.0/24 
           [110/3] via 100.0.28.254, 00:02:24, GigabitEthernet0/0/0
C        100.0.28.0/24 is directly connected, GigabitEthernet0/0/0
L        100.0.28.28/32 is directly connected, GigabitEthernet0/0/0
```

Wireshark

```
Label Distribution Protocol
    Version: 1
    PDU Length: 662
    LSR ID: 10.10.10.28
    Label Space ID: 0
    Address Message
        0... .... = U bit: Unknown bit not set
        Message Type: Address Message (0x300)
        Message Length: 22
        Message ID: 0x00000077
        Address List
            00.. .... = TLV Unknown bits: Known TLV, do not Forward (0x0)
            TLV Type: Address List (0x101)
            TLV Length: 14
            Address Family: IPv4 (1)
            Addresses
                Address 1: 100.0.28.28
                Address 2: 10.28.28.254
                Address 3: 10.10.10.28
    Label Mapping Message
```
## Table de routage du LSR4
```bash
LSR4#sh ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     100.0.0.0/24 is subnetted, 5 subnets
C       100.0.28.0 is directly connected, FastEthernet0/0
O       100.0.24.0 [110/11] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
O       100.0.25.0 [110/11] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
O       100.0.26.0 [110/2] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
O       100.0.27.0 [110/2] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
     10.0.0.0/8 is variably subnetted, 18 subnets, 2 masks
O       10.0.12.0/24 [110/2] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
O       10.10.10.2/32 [110/3] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
                      [110/3] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
O       10.10.10.3/32 [110/2] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
O       10.28.28.0/24 [110/2] via 100.0.28.28, 00:13:38, FastEthernet0/0
O E2    10.27.27.0/24 [110/20] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
O E2    10.26.26.0/24 [110/20] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
O E2    10.25.25.0/24 [110/20] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
O E2    10.24.24.0/24 [110/20] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
O       10.10.10.1/32 [110/2] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
C       10.10.10.4/32 is directly connected, Loopback4
O E2    10.10.10.26/32 [110/20] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
O E2    10.10.10.27/32 [110/20] via 10.0.34.3, 00:13:48, FastEthernet0/0/0
O E2    10.10.10.24/32 [110/20] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
O E2    10.10.10.25/32 [110/20] via 10.0.41.1, 00:16:10, FastEthernet0/1/0
O E2    10.10.10.28/32 [110/20] via 100.0.28.28, 00:13:38, FastEthernet0/0
O       10.0.23.0/24 [110/2] via 10.0.34.3, 00:16:10, FastEthernet0/0/0
C       10.0.41.0/24 is directly connected, FastEthernet0/1/0
C       10.0.34.0/24 is directly connected, FastEthernet0/0/0

```

## MPLS forwarding-table du routeur 28
```bash
Router-28(config-if)#do sh mpls for
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         18         10.0.12.0/24     0             Gi0/0/0    100.0.28.254
17         19         10.0.23.0/24     0             Gi0/0/0    100.0.28.254
18         Pop Label  10.0.34.0/24     0             Gi0/0/0    100.0.28.254
19         Pop Label  10.0.41.0/24     0             Gi0/0/0    100.0.28.254
20         17         10.10.10.1/32    0             Gi0/0/0    100.0.28.254
21         16         10.10.10.2/32    0             Gi0/0/0    100.0.28.254
22         20         10.10.10.3/32    0             Gi0/0/0    100.0.28.254
23         Pop Label  10.10.10.4/32    0             Gi0/0/0    100.0.28.254
24         33         10.10.10.24/32   0             Gi0/0/0    100.0.28.254
25         28         10.10.10.25/32   0             Gi0/0/0    100.0.28.254
26         25         10.10.10.26/32   0             Gi0/0/0    100.0.28.254
27         35         10.10.10.27/32   0             Gi0/0/0    100.0.28.254
28         34         10.24.24.0/24    0             Gi0/0/0    100.0.28.254
29         29         10.25.25.0/24    0             Gi0/0/0    100.0.28.254
30         26         10.26.26.0/24    0             Gi0/0/0    100.0.28.254
31         22         10.27.27.0/24    0             Gi0/0/0    100.0.28.254
32         32         100.0.24.0/24    0             Gi0/0/0    100.0.28.254
33         27         100.0.25.0/24    0             Gi0/0/0    100.0.28.254
34         24         100.0.26.0/24    0             Gi0/0/0    100.0.28.254
35         21         100.0.27.0/24    0             Gi0/0/0    100.0.28.254
```


## MPLS forwarding-table du LSR4
```bash
LSR4#sh mpls for
Local  Outgoing      Prefix            Bytes Label   Outgoing   Next Hop    
Label  Label or VC   or Tunnel Id      Switched      interface              
16     17            10.10.10.2/32     0             Fa0/0/0    10.0.34.3   
       16            10.10.10.2/32     0             Fa0/1/0    10.0.41.1   
17     Pop Label     10.10.10.1/32     0             Fa0/1/0    10.0.41.1   
18     Pop Label     10.0.12.0/24      0             Fa0/1/0    10.0.41.1   
19     Pop Label     10.0.23.0/24      0             Fa0/0/0    10.0.34.3   
20     Pop Label     10.10.10.3/32     0             Fa0/0/0    10.0.34.3   
21     Pop Label     100.0.27.0/24     2058          Fa0/0/0    10.0.34.3   
22     22            10.27.27.0/24     918           Fa0/0/0    10.0.34.3   
24     Pop Label     100.0.26.0/24     5785          Fa0/0/0    10.0.34.3   
25     21            10.10.10.26/32    0             Fa0/0/0    10.0.34.3   
26     24            10.26.26.0/24     3980          Fa0/0/0    10.0.34.3   
27     Pop Label     100.0.25.0/24     0             Fa0/1/0    10.0.41.1   
28     28            10.10.10.25/32    0             Fa0/1/0    10.0.41.1   
29     29            10.25.25.0/24     0             Fa0/1/0    10.0.41.1   
30     Pop Label     10.28.28.0/24     784           Fa0/0      100.0.28.28 
31     Pop Label     10.10.10.28/32    882           Fa0/0      100.0.28.28 
32     Pop Label     100.0.24.0/24     0             Fa0/1/0    10.0.41.1   
33     32            10.10.10.24/32    0             Fa0/1/0    10.0.41.1   
34     33            10.24.24.0/24     11373         Fa0/1/0    10.0.41.1   
35     34            10.10.10.27/32    918           Fa0/0/0    10.0.34.3
```

## CEF du routeur 28
```bash
Router-28(config-if)#do sh ip cef 10.0.12.0 det
10.0.12.0/24, epoch 1
  local label info: global/16
  nexthop 100.0.28.254 GigabitEthernet0/0/0 label 18

Router-28(config-if)#do sh ip cef 10.0.34.0 det
10.0.34.0/24, epoch 1
  local label info: global/18
  nexthop 100.0.28.254 GigabitEthernet0/0/0
```


## CEF du LSR4

```bash
LSR4#sh ip cef 10.0.12.0 det
10.0.12.0/24, epoch 0
  local label info: global/18
  nexthop 10.0.41.1 FastEthernet0/1/0

LSR4#sh ip cef 10.0.34.0 det
10.0.34.0/32, epoch 0, flags receive
  Dependent covered prefix type cover need deagg cover 10.0.34.0/24
  Interface source: FastEthernet0/0/0
  receive for FastEthernet0/0/0
```
Notre label local global est 18, et notre interface sortie Fe0/1/0 avec le next-hop 10.0.41.1


## MPLS LDP detail du routeur 28
```bash
Router-28(config-if)#do sh mpls ldp bindings 10.0.12.0 24 detail
  lib entry: 10.0.12.0/24, rev 2, chkpt: none
        local binding:  label: 16 (owner LDP)
          Advertised to:
          10.10.10.4:0           
        remote binding: lsr: 10.10.10.4:0, label: 18 checkpointed
```

On possède le label 16, et notre prochain label est **18** @IP10.10.10.4 pour le saut.

# Ping poste 27
On observe au dessus de la couche 2 du modèle, MPLS, on lit:
```
MultiProtocol Label Switching Header, Label: 21, Exp: 0, S: 1, TTL: 63
    0000 0000 0000 0001 0101 .... .... .... = MPLS Label: 21 (0x00015)
    .... .... .... .... .... 000. .... .... = MPLS Experimental Bits: 0
    .... .... .... .... .... ...1 .... .... = MPLS Bottom Of Label Stack: 1
    .... .... .... .... .... .... 0011 1111 = MPLS TTL: 63

```

L'avant dernier retire le lale
```
Frame 42: 98 bytes on wire (784 bits), 98 bytes captured (784 bits) on interface \Device\NPF_{0146BE60-F92F-48FB-84BD-320E6FABE536}, id 0
Ethernet II, Src: Cisco_f2:65:70 (00:13:60:f2:65:70), Dst: Cisco_af:dd:00 (00:b1:e3:af:dd:00)
    Destination: Cisco_af:dd:00 (00:b1:e3:af:dd:00)
    Source: Cisco_f2:65:70 (00:13:60:f2:65:70)
    Type: IPv4 (0x0800)
Internet Protocol Version 4, Src: 100.0.27.27, Dst: 10.28.28.28
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
    Total Length: 84
    Identification: 0x7d27 (32039)
    Flags: 0x40, Don't fragment
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 253
    Protocol: ICMP (1)
    Header Checksum: 0x5b2e [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 100.0.27.27
    Destination Address: 10.28.28.28
Internet Control Message Protocol

```
![[ping_mpls.png]]

## MPLS forwarding-table du routeur 27
```bash
Routeur27>sh mpls forwarding-table 
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         20         10.0.12.0/24     0             Gi0/0/0    100.0.27.254
17         Pop Label  10.0.23.0/24     0             Gi0/0/0    100.0.27.254
18         Pop Label  10.0.34.0/24     0             Gi0/0/0    100.0.27.254
19         19         10.0.41.0/24     0             Gi0/0/0    100.0.27.254
20         18         10.10.10.1/32    0             Gi0/0/0    100.0.27.254
21         17         10.10.10.2/32    0             Gi0/0/0    100.0.27.254
22         Pop Label  10.10.10.3/32    0             Gi0/0/0    100.0.27.254
23         16         10.10.10.4/32    0             Gi0/0/0    100.0.27.254
24         32         10.10.10.24/32   0             Gi0/0/0    100.0.27.254
25         27         10.10.10.25/32   0             Gi0/0/0    100.0.27.254
26         21         10.10.10.26/32   0             Gi0/0/0    100.0.27.254
27         30         10.10.10.28/32   0             Gi0/0/0    100.0.27.254
28         33         10.24.24.0/24    0             Gi0/0/0    100.0.27.254
29         28         10.25.25.0/24    0             Gi0/0/0    100.0.27.254
30         24         10.26.26.0/24    0             Gi0/0/0    100.0.27.254
31         29         10.28.28.0/24    0             Gi0/0/0    100.0.27.254
32         31         100.0.24.0/24    0             Gi0/0/0    100.0.27.254
33         25         100.0.25.0/24    0             Gi0/0/0    100.0.27.254
34         Pop Label  100.0.26.0/24    0             Gi0/0/0    100.0.27.254
35         26         100.0.28.0/24    0             Gi0/0/0    100.0.27.254
```

## Configuration du router CE
```
Router(config)#enable secret lannion
Router(config)#hostname CE-2600
CE-2600(config)#int FastEthernet 0/0
CE-2600(config-if)#ip addr 10.34.1.1 255.255.255.0
CE-2600(config-if)#no shut
CE-2600(config-if)#exit
CE-2600(config)#int FastEthernet 0/1
CE-2600(config-if)#ip addr 192.168.34.254 255.255.255.0
CE-2600(config-if)#no shut
CE-2600(config-if)#exit
CE-2600(config)#int Loopback34
CE-2600(config-if)#ip addr 34.34.34.34 255.255.255.255
CE-2600(config-if)#no shut
CE-2600(config-if)#exit
CE-2600(config)#router ospf 200
CE-2600(config-router)#redistribute connected subnets 
CE-2600(config-router)#network 10.34.1.0 0.0.0.255 area 6
```

## Configuration du vrf
```
Router-ISR-34(config)#hostname PE-ISR-34
PE-ISR-34(config)#int GigabitEthernet 0/0/1
PE-ISR-34(config-if)#ip vrf poste_33_34
PE-ISR-34(config-vrf)#rd 33:34
PE-ISR-34(config-vrf)#route-target both 33:34
PE-ISR-34(config-vrf)#exit
PE-ISR-34(config)#int GigabitEthernet 0/0/0
PE-ISR-34(config-if)#ip vrf forwarding poste_33_34
PE-ISR-34(config-if)#ip address 10.34.1.254 255.255.255.0
```

**TEST**
```
PE-ISR-34#ping vrf poste_33_34 10.34.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.34.1.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```

**SH RUN**
```
ip vrf poste_33_34
 rd 33:34
 route-target export 33:34
 route-target import 33:34
 
 
PE-ISR-34#sh ip route vrf poste_33_34
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.34.1.0/24 is directly connected, GigabitEthernet0/0/0
L        10.34.1.254/32 is directly connected, GigabitEthernet0/0/0
      34.0.0.0/32 is subnetted, 1 subnets
O E2     34.34.34.34 [110/20] via 10.34.1.1, 00:03:25, GigabitEthernet0/0/0
O E2  192.168.34.0/24 [110/20] via 10.34.1.1, 00:03:25, GigabitEthernet0/0/0
```
