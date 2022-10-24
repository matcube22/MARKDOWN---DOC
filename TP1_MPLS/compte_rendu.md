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

interface FastEthernet0/2

interface FastEthernet0/3

interface FastEthernet0/4

interface FastEthernet0/5

interface FastEthernet0/6

interface FastEthernet0/7
 
interface FastEthernet0/8

interface FastEthernet0/9

interface FastEthernet0/10

interface FastEthernet0/11

interface FastEthernet0/12

interface FastEthernet0/13

interface FastEthernet0/14

interface FastEthernet0/15

interface FastEthernet0/16

interface FastEthernet0/17

interface FastEthernet0/18

interface FastEthernet0/19
 
interface FastEthernet0/20

interface FastEthernet0/21

interface FastEthernet0/22

interface FastEthernet0/23

interface FastEthernet0/24

interface GigabitEthernet0/1

interface GigabitEthernet0/2

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
