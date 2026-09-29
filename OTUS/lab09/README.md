# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №9
BGP для маршрутизации IPv6 unicast

## Цель работы
Настроить eBGP/iBGP IPv6 unicast для всех сегментов сети по аналогичной логике с настройкой eBGP/iBGP IPv4 unicast.


- Настроен eBGP IPv6 unicast между офисом Москва и провайдерами Киторн и Ламас
- Настроен eBGP IPv6 unicast между провайдерами Киторн и Ламас
- Настроен eBGP IPv6 unicast между Ламас и Триада
- Настроен eBGP IPv6 unicast между офисом С.-Петербург и провайдером Триада
- Организована IPv6 unicast связность между пограничными роутерами Москва и СПб
- Настроен iBGP IPv6 unicast в офисе Москва (R14 ↔ R15)
- Настроен iBGP IPv6 unicast в Триаде с Route Reflector (R24)
- Все сети IPv6 имеют связность между собой

## Выполненные работы

## 1️⃣ eBGP IPv6 между офисом Москва и провайдерами

### R14 ↔ R22 (Киторн, AS 101)
```cisco
router bgp 1001
 neighbor 2001:DB8:0:20::1 remote-as 101
 address-family ipv6
  network 2001:DB8:55::/64
  network 2001:DB8:62::/64
  neighbor 2001:DB8:0:20::1 activate
  neighbor 2001:DB8:0:20::1 next-hop-self
 exit-address-family
 ```
 ### R15 ↔ R21 (Ламас, AS 301)
 ```
router bgp 1001
 neighbor 2001:DB8:0:21::1 remote-as 301
 address-family ipv6
  network 2001:DB8:55::/64
  network 2001:DB8:62::/64
  neighbor 2001:DB8:0:21::1 activate
  neighbor 2001:DB8:0:21::1 next-hop-self
 exit-address-family
 ```

## 2️⃣ eBGP IPv6 между Киторн и Ламас

### R22 (Киторн) ↔ R21 (Ламас)
```
! На R22
router bgp 101
 neighbor 2001:DB8:0:32::2 remote-as 301
 address-family ipv6
  neighbor 2001:DB8:0:32::2 activate
 exit-address-family

! На R21
router bgp 301
 neighbor 2001:DB8:0:32::1 remote-as 101
 address-family ipv6
  neighbor 2001:DB8:0:32::1 activate
 exit-address-family
```

## 3️⃣ eBGP IPv6 между Ламас и Триада
### R21 (Ламас) ↔ R24 (Триада)
```
! На R21
router bgp 301
 neighbor 2001:DB8:1::2 remote-as 520
 address-family ipv6
  neighbor 2001:DB8:1::2 activate
 exit-address-family

! На R24
router bgp 520
 neighbor 2001:DB8:1::1 remote-as 301
 address-family ipv6
  neighbor 2001:DB8:1::1 activate
  neighbor 2001:DB8:1::1 next-hop-self
 exit-address-family
```
## 4️⃣ eBGP IPv6 между офисом СПб и Триадой
### R18 ↔ R24 и R18 ↔ R26
```
router bgp 2042
 bgp router-id 10.255.255.18
 neighbor 2001:DB8:0:52::1 remote-as 520
 neighbor 2001:DB8:0:47::1 remote-as 520
 address-family ipv6
  network 2001:DB8:30::/64
  network 2001:DB8:40::/64
  neighbor 2001:DB8:0:52::1 activate
  neighbor 2001:DB8:0:52::1 next-hop-self
  neighbor 2001:DB8:0:47::1 activate
  neighbor 2001:DB8:0:47::1 next-hop-self
 exit-address-family
```

## 5️⃣ iBGP IPv6 в офисе Москва (R14 ↔ R15)
```
! На R14
router bgp 1001
 neighbor 2001:DB8:0:1::2 remote-as 1001
 address-family ipv6
  neighbor 2001:DB8:0:1::2 activate
  neighbor 2001:DB8:0:1::2 next-hop-self
 exit-address-family

! На R15
router bgp 1001
 neighbor 2001:DB8:0:1::1 remote-as 1001
 address-family ipv6
  neighbor 2001:DB8:0:1::1 activate
  neighbor 2001:DB8:0:1::1 next-hop-self
 exit-address-family
```

## 6️⃣ iBGP IPv6 в Триаде с Route Reflector (R24)
### R24 — Route Reflector
```
router bgp 520
 neighbor 2001:DB8::23 remote-as 520
 neighbor 2001:DB8::23 update-source Loopback0
 neighbor 2001:DB8::25 remote-as 520
 neighbor 2001:DB8::25 update-source Loopback0
 neighbor 2001:DB8::26 remote-as 520
 neighbor 2001:DB8::26 update-source Loopback0
 address-family ipv6
  neighbor 2001:DB8::23 activate
  neighbor 2001:DB8::23 route-reflector-client
  neighbor 2001:DB8::23 next-hop-self
  neighbor 2001:DB8::25 activate
  neighbor 2001:DB8::25 route-reflector-client
  neighbor 2001:DB8::25 next-hop-self
  neighbor 2001:DB8::26 activate
  neighbor 2001:DB8::26 route-reflector-client
  neighbor 2001:DB8::26 next-hop-self
 exit-address-family
```
### R23, R25, R26 — клиенты RR
```
router bgp 520
 neighbor 2001:DB8::24 remote-as 520
 neighbor 2001:DB8::24 update-source Loopback0
 address-family ipv6
  neighbor 2001:DB8::24 activate
 exit-address-family
 ```

## 7️⃣ Организация IPv6-связности между офисами
### Москва: OSPFv3 + redistribute BGP
```
! На R14 и R15
ipv6 router ospf 1
 router-id 10.255.255.X
 default-information originate always
 exit

! На R14 добавлен статический маршрут до СПб
ipv6 route 2001:DB8:30::/64 2001:DB8:0:1::2
ipv6 route 2001:DB8:40::/64 2001:DB8:0:1::2

! На R15 добавлен статический маршрут до СПб
ipv6 route 2001:DB8:30::/64 2001:DB8:0:21::1
ipv6 route 2001:DB8:40::/64 2001:DB8:0:21::1
```

### Триада: IS-IS + iBGP
```
router isis
 net 49.0024.0000.0000.0024.00
 is-type level-2-only
 metric-style wide
 address-family ipv6
  multi-topology
 exit-address-family
```
### СПб: EIGRP IPv6 + redistribute BGP
```
! На R18
router eigrp EIGRP_SPB
 address-family ipv6 unicast autonomous-system 2042
  topology base
   redistribute bgp 2042 metric 1000000 100 255 1 1500
  exit-af-topology
 exit-address-family

! Статические маршруты до Москвы
ipv6 route 2001:DB8:55::/64 2001:DB8:0:52::1
ipv6 route 2001:DB8:62::/64 2001:DB8:0:52::1

! На R16 и R17 активирован EIGRP для IPv6
router eigrp EIGRP_SPB
 address-family ipv6 unicast autonomous-system 2042
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  af-interface Ethernet0/0.270
   no passive-interface
  exit-af-interface
  af-interface Loopback0
   no passive-interface
  exit-af-interface
 exit-address-family
```
### Ламас: статические маршруты
```
! На R21
ipv6 route 2001:DB8:30::/64 2001:DB8:1::2
ipv6 route 2001:DB8:40::/64 2001:DB8:1::2
ipv6 route 2001:DB8:55::/64 2001:DB8:0:21::2
ipv6 route 2001:DB8:62::/64 2001:DB8:0:21::2
```

## 8️⃣ IPv6-связность
```
R14_Core#show bgp ipv6 unicast summary
BGP router identifier 10.255.255.14, local AS number 1001
BGP table version is 445, main routing table version 445
6 network entries using 1008 bytes of memory
10 path entries using 1040 bytes of memory
6/4 BGP path/bestpath attribute entries using 912 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3080 total bytes of memory
BGP activity 43/18 prefixes, 70/34 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:0:1::2 4         1001   11173   11107      445    0    0 6d22h           5
2001:DB8:0:20::1
                4          101   11179   11110      445    0    0 6d22h           2


R15_Core#show bgp ipv6 unicast summary
BGP router identifier 10.255.255.15, local AS number 1001
BGP table version is 740, main routing table version 740
6 network entries using 1008 bytes of memory
8 path entries using 832 bytes of memory
5/4 BGP path/bestpath attribute entries using 760 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2696 total bytes of memory
BGP activity 35/10 prefixes, 57/23 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:0:1::1 4         1001   11107   11174      740    0    0 6d22h           3
2001:DB8:0:21::1
                4          301   12685   12562      740    0    0 1w0d            2


R18#show ipv6 route 2001:DB8:0:52::1
Routing entry for 2001:DB8:0:52::/64
  Known via "connected", distance 0, metric 0, type connected
  Route count is 1/1, share count 0
  Routing paths:
    directly connected via Ethernet0/2
      Last updated 2w3d ago

R18#ping ipv6 2001:DB8:0:52::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:0:52::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms
R18#ping ipv6 2001:DB8:0:47::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:0:47::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/5 ms

R24#show bgp ipv6 unicast summary
BGP router identifier 10.255.255.24, local AS number 520
BGP table version is 459, main routing table version 459
6 network entries using 1008 bytes of memory
6 path entries using 624 bytes of memory
2/2 BGP path/bestpath attribute entries using 304 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2056 total bytes of memory
BGP activity 33/8 prefixes, 40/14 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8::23    4          520    9304    9433      459    0    0 5d20h           0
2001:DB8::25    4          520       0       0        1    0    0 never    Idle
2001:DB8::26    4          520    9304    9434      459    0    0 5d20h           0
2001:DB8:0:52::2
                4         2042       0       0        1    0    0 5d18h    Idle
2001:DB8:1::1   4          301    9461    9325      459    0    0 5d20h           4
R24#show ipv6 route bgp
IPv6 Routing Table - default - 21 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8::14/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5020, Ethernet0/0
B   2001:DB8::15/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5020, Ethernet0/0


R21#show bgp ipv6 unicast summary
BGP router identifier 10.255.255.21, local AS number 301
BGP table version is 702, main routing table version 702
6 network entries using 1008 bytes of memory
10 path entries using 1040 bytes of memory
5/4 BGP path/bestpath attribute entries using 760 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2928 total bytes of memory
BGP activity 37/12 prefixes, 49/18 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:0:21::2
                4         1001   12634   12757      702    0    0 1w0d            4
2001:DB8:0:32::1
                4          101   28716   28719      702    0    0 2w3d            4
2001:DB8:1::2   4          520    9326    9462      702    0    0 5d20h           2
R21#
R21#show ipv6 route bgp
IPv6 Routing Table - default - 14 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8::14/128 [20/0]
     via FE80::A8BB:CCFF:FE00:F020, Ethernet0/0
B   2001:DB8::15/128 [20/0]
     via FE80::A8BB:CCFF:FE00:F020, Ethernet0/0
B   2001:DB8:55::/64 [20/20]
     via FE80::A8BB:CCFF:FE00:F020, Ethernet0/0
B   2001:DB8:62::/64 [20/20]
     via FE80::A8BB:CCFF:FE00:F020, Ethernet0/0
R21#
```
### Ping с Москвы до СПБ и обратно 
```
R12#ping ipv6 2001:DB8:30::1 source 2001:DB8:55::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:30::1, timeout is 2 seconds:
Packet sent with a source address of 2001:DB8:55::1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/6 ms
R12#
R17#ping ipv6 2001:DB8:55::1 source 2001:DB8:30::1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:55::1, timeout is 2 seconds:
Packet sent with a source address of 2001:DB8:30::1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R17#
```