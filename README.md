# VxLAN. EVPN L3
## Задание:
- Настроите каждого клиента в своем VNI
- Настроите маршрутизацию между клиентами.

- underlay - iBGP
![alt-dtp](https://github.com/vk1391/OTUS_DC_VXLAN_L3VNI/blob/main/vxlan_l3vni.jpg)

### Настроите каждого клиента в своем VNI
- конфигурация leaf1
```
vlan 10
!
vrf instance L3VNI
!
interface Ethernet1
   no switchport
   ip address 10.1.11.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.11.2/30
!
interface Ethernet3
   switchport access vlan 10
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 11.11.11.11/32
!
interface Loopback1
   ip address 101.11.11.11/32
!
interface Management1
!
interface Vlan10
   vrf L3VNI
   ip address 10.0.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100
   vxlan vrf L3VNI vni 1000
   vxlan flood vtep 101.13.13.13
!
ip routing
ip routing vrf L3VNI
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 11.11.11.11
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.11.1 remote-as 101
   neighbor 10.1.11.1 next-hop-self
   neighbor 10.1.11.1 bfd
   neighbor 10.2.11.1 remote-as 101
   neighbor 10.2.11.1 next-hop-self
   neighbor 10.2.11.1 bfd
   neighbor 101.12.12.12 remote-as 101
   neighbor 101.12.12.12 update-source Loopback1
   neighbor 101.12.12.12 send-community extended
   neighbor 101.13.13.13 remote-as 101
   neighbor 101.13.13.13 update-source Loopback1
   neighbor 101.13.13.13 send-community extended
   redistribute connected route-map redist
   !
   vlan 10
      rd 101.13.13.13:1
      route-target both 101:1
      redistribute learned
   !
   address-family evpn
      no neighbor 10.1.11.1 activate
      no neighbor 10.2.11.1 activate
      no neighbor 101.1.1.1 activate
      no neighbor 101.2.2.2 activate
      neighbor 101.12.12.12 activate
      neighbor 101.13.13.13 activate
      no neighbor 101.22.22.22 activate
      no neighbor 101.33.33.33 activate
   !
   address-family ipv4
      neighbor 10.1.11.1 activate
      neighbor 10.2.11.1 activate
      no neighbor 101.1.1.1 activate
      no neighbor 101.2.2.2 activate
      no neighbor 101.12.12.12 activate
      no neighbor 101.13.13.13 activate
   !
   vrf L3VNI
      rd 101.11.11.11:1000
      route-target import evpn 101:1000
      route-target export evpn 101:1000
!
end
```
- конфигурация leaf2:
```
vlan 11
!
vrf instance L3VNI
!
interface Ethernet1
   no switchport
   ip address 10.1.12.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.12.2/30
!
interface Ethernet3
   switchport access vlan 11
   ip address 11.0.0.1/30
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 12.12.12.12/32
!
interface Loopback1
   ip address 101.12.12.12/32
!
interface Management1
!
interface Vlan11
   vrf L3VNI
   ip address 11.0.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 11 vni 101
   vxlan vrf L3VNI vni 1000
!
ip routing
ip routing vrf L3VNI
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 12.12.12.12
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.12.1 remote-as 101
   neighbor 10.1.12.1 bfd
   neighbor 10.2.12.1 remote-as 101
   neighbor 10.2.12.1 bfd
   neighbor 101.11.11.11 remote-as 101
   neighbor 101.11.11.11 update-source Loopback1
   neighbor 101.11.11.11 send-community extended
   redistribute connected route-map redist
   !
   vlan 11
      rd 101.11.11.11:101
      route-target both 101:101
      redistribute learned
   !
   address-family evpn
      neighbor 101.11.11.11 activate
   !
   address-family ipv4
      no neighbor 101.11.11.11 activate
   !
   vrf L3VNI
      rd 101.11.11.11:1000
      route-target import evpn 101:1000
      route-target export evpn 101:1000
```
- конфигурация leaf3:
```
vlan 10,12
!
vrf instance L3VNI
!
interface Ethernet1
   no switchport
   ip address 10.1.13.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.13.2/30
!
interface Ethernet3
   switchport access vlan 10
!
interface Ethernet4
   switchport access vlan 12
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 13.13.13.13/32
!
interface Loopback1
   ip address 101.13.13.13/32
!
interface Management1
!
interface Vlan10
   vrf L3VNI
   ip address 12.0.0.1/30
!
interface Vlan11
!
interface Vlan12
   vrf L3VNI
   ip address 13.0.0.1/30
!
interface Vxlan1
   vxlan source-interface Loopback1
   vxlan udp-port 4789
   vxlan vlan 10 vni 100
   vxlan vlan 12 vni 102
   vxlan vrf L3VNI vni 1000
   vxlan flood vtep 101.11.11.11
!
ip routing
ip routing vrf L3VNI
!
route-map redist permit 10
   match interface Loopback0
!
route-map redist permit 20
   match interface Loopback1
!
router bgp 101
   router-id 13.13.13.13
   timers bgp 3 9
   maximum-paths 2
   neighbor 10.1.13.1 remote-as 101
   neighbor 10.1.13.1 bfd
   neighbor 10.2.13.1 remote-as 101
   neighbor 10.2.13.1 bfd
   neighbor 101.11.11.11 remote-as 101
   neighbor 101.11.11.11 update-source Loopback1
   neighbor 101.11.11.11 send-community extended
   redistribute connected route-map redist
   !
   vlan 10
      rd 101.11.11.11:1
      route-target both 101:1
      redistribute learned
   !
   vlan 12
      rd 101.11.11.11:102
      route-target both 101:102
      redistribute learned
   !
   address-family evpn
      neighbor 101.11.11.11 activate
   !
   address-family ipv4
      no neighbor 101.11.11.11 activate
   !
   vrf L3VNI
      rd 101.13.13.13:1000
      route-target import evpn 101:1000
      route-target export evpn 101:1000
!
end
```
### Настроите маршрутизацию между клиентами
- leaf1 vni:
```
localhost#sh vxlan vni
VNI to VLAN Mapping for Vxlan1
VNI       VLAN       Source       Interface       802.1Q Tag
--------- ---------- ------------ --------------- ----------
100       10         static       Ethernet3       untagged  
                                  Vxlan1          10        

VNI to dynamic VLAN Mapping for Vxlan1
VNI        VLAN       VRF         Source       
---------- ---------- ----------- ------------ 
1000       4094       L3VNI       evpn
```
- leaf 1 bgp evpn:
```
localhost#sh bgp evpn sum
BGP summary information for VRF default
Router identifier 11.11.11.11, local AS number 101
Neighbor Status Codes: m - Under maintenance
  Neighbor     V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  101.12.12.12 4 101           162188    162118    0    0 17:36:58 Estab   1      1
  101.13.13.13 4 101           194358    194269    0    0    2d02h Estab   4      4
```
leaf1 таблица маршрутизации L3vni:
```
localhost#sh ip route vrf L3VNI

VRF: L3VNI
Codes: C - connected, S - static, K - kernel, 
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort is not set

 C        10.0.0.0/30 is directly connected, Vlan10
 B I      11.0.0.2/32 [200/0] via VTEP 101.12.12.12 VNI 1000 router-mac 50:3a:6c:db:0b:b5 local-interface Vxlan1
 B I      12.0.0.2/32 [200/0] via VTEP 101.13.13.13 VNI 1000 router-mac 50:37:d4:2d:ff:13 local-interface Vxlan1
 B I      13.0.0.2/32 [200/0] via VTEP 101.13.13.13 VNI 1000 router-mac 50:37:d4:2d:ff:13 local-interface Vxlan1
```
- ping ПК leaf1 - ПК leaf2:
```
VPCS> sh ip

NAME        : VPCS[1]
IP/MASK     : 10.0.0.2/30
GATEWAY     : 10.0.0.1
DNS         : 
MAC         : 00:50:79:66:68:06
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 11.0.0.2

84 bytes from 11.0.0.2 icmp_seq=1 ttl=62 time=32.862 ms
84 bytes from 11.0.0.2 icmp_seq=2 ttl=62 time=36.065 ms
84 bytes from 11.0.0.2 icmp_seq=3 ttl=62 time=33.047 ms
84 bytes from 11.0.0.2 icmp_seq=4 ttl=62 time=36.981 ms
```
-  ping ПК leaf1 - ПК leaf3:
```
VPCS> ping 12.0.0.2

84 bytes from 12.0.0.2 icmp_seq=1 ttl=62 time=32.429 ms
84 bytes from 12.0.0.2 icmp_seq=2 ttl=62 time=45.173 ms
84 bytes from 12.0.0.2 icmp_seq=3 ttl=62 time=35.958 ms
^C
VPCS> ping 13.0.0.2

84 bytes from 13.0.0.2 icmp_seq=1 ttl=62 time=36.828 ms
84 bytes from 13.0.0.2 icmp_seq=2 ttl=62 time=36.511 ms
84 bytes from 13.0.0.2 icmp_seq=3 ttl=62 time=42.333 ms
84 bytes from 13.0.0.2 icmp_seq=4 ttl=62 time=34.148 ms
```
