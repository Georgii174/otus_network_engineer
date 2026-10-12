# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №12

## 🎯 Цель работы
Настроить BGP free core (ядро без BGP) в офисах Москвы и Санкт-Петербурга с использованием MPLS LDP.

* Настроите BGP free core в офисе Москвы.
* Настроите BGP free core в офисе Санкт-Петербурга.
* План работы и изменения зафиксированы в документации.

## 1️⃣ BGP Free Core в офисе Москвы
```

На P-роутерах (R12, R13, R19, R20)
MPLS на интерфейсах
interface Ethernet0/1
 mpls label protocol ldp
 mpls ip
exit
!
LDP Router-ID
mpls ldp router-id Loopback0 force
!
BGP не настраивается

На PE-роутерах (R14, R15):
router bgp 1001
 neighbor 10.255.255.15 remote-as 1001
 neighbor 10.255.255.15 update-source Loopback0
 address-family ipv4
  neighbor 10.255.255.15 activate
  neighbor 10.255.255.15 next-hop-self
  neighbor 10.255.255.15 send-label
 exit-address-family
exit
```

## 2️⃣ BGP Free Core в офисе Санкт-Петербурга
```
На P-роутерах (R16, R17, R32):
interface Ethernet0/1
 mpls label protocol ldp
 mpls ip
exit
!
mpls ldp router-id Loopback0 force
! BGP не настраивается

На PE-роутере (R18):
router bgp 2042
 ! eBGP с Триадой и Ламасом
 neighbor 52.10.5.1 remote-as 520
 neighbor 47.32.2.1 remote-as 520
 address-family ipv4
  network 192.168.30.0 mask 255.255.255.224
  network 192.168.40.0 mask 255.255.255.224
  neighbor 52.10.5.1 activate
  neighbor 52.10.5.1 next-hop-self
  neighbor 47.32.2.1 activate
 exit-address-family
exit
```

## 3️⃣ Проверка работы MPLS LDP

```
Проверка со стороны МСК:

R12#show ip route | include b
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
      10.0.0.0/8 is variably subnetted, 19 subnets, 3 masks
C        10.255.255.12/32 is directly connected, Loopback0
      100.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
      192.168.55.0/24 is variably subnetted, 2 subnets, 2 masks
      192.168.62.0/27 is subnetted, 1 subnets
R12#show mpls ldp neighbor
    Peer LDP Ident: 10.255.255.15:0; Local LDP Ident 10.255.255.12:0
        TCP connection: 10.255.255.15.64749 - 10.255.255.12.646
        State: Oper; Msgs sent/rcvd: 4975/4986; Downstream
        Up time: 2d23h
        LDP discovery sources:
          Ethernet0/3, Src IP addr: 10.22.33.1
        Addresses bound to peer LDP Ident:
          10.40.21.1      10.255.255.15   10.22.33.1      188.0.0.2
          10.30.11.1      10.0.0.2        10.100.100.1    10.100.200.1
    Peer LDP Ident: 10.255.255.14:0; Local LDP Ident 10.255.255.12:0
        TCP connection: 10.255.255.14.61963 - 10.255.255.12.646
        State: Oper; Msgs sent/rcvd: 4958/4964; Downstream
        Up time: 2d23h
        LDP discovery sources:
          Ethernet0/2, Src IP addr: 100.40.55.1
        Addresses bound to peer LDP Ident:
          100.40.55.1     10.255.255.14   10.10.25.1      172.0.20.2
          10.132.35.1     10.0.0.1
R12#show mpls forwarding-table 10.255.255.15
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop
Label      Label      or Tunnel Id     Switched      interface
28         explicit-n 10.255.255.15/32 0             Et0/3      10.22.33.1
R12#ping 192.168.30.15 source 192.168.55.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.30.15, timeout is 2 seconds:
Packet sent with a source address of 192.168.55.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/5 ms
R12#traceroute 192.168.30.15 source 192.168.55.1
Type escape sequence to abort.
Tracing the route to 192.168.30.15
VRF info: (vrf in name/id, vrf out name/id)
  1 10.22.33.1 1 msec 1 msec 0 msec
  2 188.0.0.1 1 msec 1 msec 2 msec
  3 31.0.0.2 1 msec 2 msec 2 msec
  4 52.10.5.2 2 msec 1 msec 3 msec
  5 10.10.1.2 1 msec 5 msec 1 msec
  6 192.168.30.15 2 msec 7 msec 2 msec


Проверка со стороны СПБ:
R16#show ip route | include b
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
      5.0.0.0/30 is subnetted, 1 subnets
      10.0.0.0/8 is variably subnetted, 25 subnets, 3 masks
C        10.255.255.16/32 is directly connected, Loopback0
      31.0.0.0/30 is subnetted, 1 subnets
      52.0.0.0/30 is subnetted, 1 subnets
      172.0.0.0/30 is subnetted, 1 subnets
      188.0.0.0/30 is subnetted, 1 subnets
      192.168.30.0/27 is subnetted, 1 subnets
      192.168.40.0/24 is variably subnetted, 2 subnets, 2 masks
      192.168.55.0/27 is subnetted, 1 subnets
      192.168.62.0/27 is subnetted, 1 subnets
R16#show mpls l
R16#show mpls ldp ne
R16#show mpls ldp neighbor
    Peer LDP Ident: 10.255.255.18:0; Local LDP Ident 10.255.255.16:0
        TCP connection: 10.255.255.18.25664 - 10.255.255.16.646
        State: Oper; Msgs sent/rcvd: 4961/4949; Downstream
        Up time: 2d23h
        LDP discovery sources:
          Ethernet0/1, Src IP addr: 10.10.2.1
        Addresses bound to peer LDP Ident:
          10.10.2.1       10.255.255.18   10.10.1.1       52.10.5.2
          47.32.2.2       10.100.100.2
R16#show mpls fo
R16#show mpls forwarding-table 10.255.255.18
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop
Label      Label      or Tunnel Id     Switched      interface
30         Pop Label  10.255.255.18/32 0             Et0/1      10.10.2.1
R16#ping 192.168.55.6 source 192.168.40.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.55.6, timeout is 2 seconds:
Packet sent with a source address of 192.168.40.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/8/17 ms
R16#tr
R16#traceroute 192.168.55.6 so
R16#traceroute 192.168.55.6 source 192.168.40.1
Type escape sequence to abort.
Tracing the route to 192.168.55.6
VRF info: (vrf in name/id, vrf out name/id)
  1 10.10.2.1 [MPLS: Label 42 Exp 0] 0 msec 2 msec 1 msec
  2 52.10.5.1 2 msec 2 msec 0 msec
  3 31.0.0.1 2 msec 2 msec 0 msec
  4 5.83.32.1 3 msec 2 msec 1 msec
  5 172.0.20.2 1 msec 1 msec 2 msec
  6 100.40.55.2 2 msec 1 msec 1 msec
  7 192.168.55.6 4 msec 6 msec 3 msec
```