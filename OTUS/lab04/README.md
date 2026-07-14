# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №4
IS-IS

Цель:
настроить протокол IS-IS для IPv4 и IPv6 в ISP Триада.


Описание/Пошаговая инструкция выполнения домашнего задания:
Настроите IS-IS в ISP Триада.
R23 и R25 находятся в зоне 2222.
R24 находится в зоне 24.
R26 находится в зоне 26.
Настройка осуществляется одновременно для IPv4 и IPv6.

## Выполнение:

По сколько на всех роутерах настройка одинакова, покажу все на примере R23 и R26:

```
R23
interface Ethernet0/1
 description link R23-R25 (Zona 2222)
 ip address 10.0.175.1 255.255.255.240
 ip router isis
 ipv6 address 2001:DB8:175::1/64
 ipv6 router isis
 isis network point-to-point
!
interface Ethernet0/2
 description link R23-R24 (Zona 24)
 ip address 10.0.177.2 255.255.255.240
 ip router isis
 ipv6 address 2001:DB8:177::2/64
 ipv6 router isis
 isis network point-to-point
!
router isis
 net 49.2222.0000.0000.0023.00
 metric-style wide
 log-adjacency-changes
 !
 address-family ipv6
  multi-topology
 exit-address-family
!

```
R26
```
!
interface Ethernet0/0
 description link R24-R26
 ip address 10.0.179.7 255.255.255.240
 ip router isis
 ipv6 address 2001:DB8:179::7/64
 ipv6 router isis
 isis network point-to-point
!
interface Ethernet0/2
 description link R26-R25 (Zona 2222)
 ip address 10.0.180.8 255.255.255.240
 ip router isis
 ipv6 address 2001:DB8:180::8/64
 ipv6 router isis
 isis network point-to-point
!
interface Ethernet0/3
 description link R26-R18
 ip address 47.32.2.1 255.255.255.252
!
router isis
 net 49.0026.0000.0000.0026.00
 metric-style wide
 log-adjacency-changes
 !
 address-family ipv6
  multi-topology
 exit-address-family
!
```

### Проверяем:

R23
```
R23# show isis neighbors

System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/2       10.0.177.5      UP    23       01
R25            L1L2 Et0/1       10.0.175.3      UP    21       01
R23#show isi
R23#show isis to
R23#show isis topology

IS-IS TID 0 paths to level-1 routers
System Id            Metric     Next-Hop             Interface   SNPA
R23                  --
R25                  10         R25                  Et0/1       aabb.cc01.9000

IS-IS TID 0 paths to level-2 routers
System Id            Metric     Next-Hop             Interface   SNPA
R23                  --
R24                  10         R24                  Et0/2       aabb.cc01.8020
R25                  10         R25                  Et0/1       aabb.cc01.9000
R26                  20         R24                  Et0/2       aabb.cc01.8020
                                R25                  Et0/1       aabb.cc01.9000
R23#show is
R23#show isis da
R23#show isis database

IS-IS Level-1 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R23.00-00           * 0x0000000C   0xD735        1104              1/0/0
R25.00-00             0x00000009   0xAA51        1146              1/0/0
IS-IS Level-2 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R23.00-00           * 0x0000000D   0xEA56        624               0/0/0
R24.00-00             0x00000005   0x99A9        1091              0/0/0
R25.00-00             0x0000000B   0xF809        1039              0/0/0
R26.00-00             0x00000003   0xFD1F        1008              0/0/0
R23#show ip ro
R23#show ip route is
R23#show ip route isis
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 12 subnets, 3 masks
i L2     10.0.179.0/28 [115/20] via 10.0.177.5, 00:21:36, Ethernet0/2
i L1     10.0.180.0/28 [115/20] via 10.0.175.3, 00:56:47, Ethernet0/1
i L2     10.255.255.24/32 [115/20] via 10.0.177.5, 00:21:36, Ethernet0/2
i L1     10.255.255.25/32 [115/20] via 10.0.175.3, 00:56:47, Ethernet0/1
i L2     10.255.255.26/32 [115/30] via 10.0.177.5, 00:15:28, Ethernet0/2
                          [115/30] via 10.0.175.3, 00:15:28, Ethernet0/1
R23#show ipv6 route isis
IPv6 Routing Table - default - 12 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
I1  2001:DB8::25/128 [115/20]
     via FE80::A8BB:CCFF:FE01:9000, Ethernet0/1
I2  2001:DB8::26/128 [115/30]
     via FE80::A8BB:CCFF:FE01:9000, Ethernet0/1
I2  2001:DB8:179::/64 [115/30]
     via FE80::A8BB:CCFF:FE01:9000, Ethernet0/1
I1  2001:DB8:180::/64 [115/20]
     via FE80::A8BB:CCFF:FE01:9000, Ethernet0/1
```

R26

```
R26#show isis neighbors

System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/0       10.0.179.6      UP    28       02
R25            L2   Et0/2       10.0.180.4      UP    25       02
R26#show is
R26#show isis to
R26#show isis topology

IS-IS TID 0 paths to level-1 routers
System Id            Metric     Next-Hop             Interface   SNPA
R26                  --

IS-IS TID 0 paths to level-2 routers
System Id            Metric     Next-Hop             Interface   SNPA
R23                  20         R24                  Et0/0       aabb.cc01.8010
                                R25                  Et0/2       aabb.cc01.9020
R24                  10         R24                  Et0/0       aabb.cc01.8010
R25                  10         R25                  Et0/2       aabb.cc01.9020
R26                  --
R26#sho
R26#show da
R26#show dat
R26#show is
R26#show isis da
R26#show isis database

IS-IS Level-1 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R26.00-00           * 0x00000003   0xF882        968               1/0/0
IS-IS Level-2 Link State Database:
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime      ATT/P/OL
R23.00-00             0x0000000D   0xEA56        516               0/0/0
R24.00-00             0x00000005   0x99A9        987               0/0/0
R25.00-00             0x0000000B   0xF809        935               0/0/0
R26.00-00           * 0x00000003   0xFD1F        908               0/0/0
R26#show ip
R26#show ip ro
R26#show ip route is
R26#show ip route isis
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
i L2     10.0.175.0/28 [115/20] via 10.0.180.4, 00:17:11, Ethernet0/2
i L2     10.0.177.0/28 [115/20] via 10.0.179.6, 00:17:11, Ethernet0/0
i L2     10.255.255.23/32 [115/30] via 10.0.180.4, 00:17:11, Ethernet0/2
                          [115/30] via 10.0.179.6, 00:17:11, Ethernet0/0
i L2     10.255.255.24/32 [115/20] via 10.0.179.6, 00:17:11, Ethernet0/0
i L2     10.255.255.25/32 [115/20] via 10.0.180.4, 00:17:11, Ethernet0/2
R26#show ipv6 route isis
IPv6 Routing Table - default - 10 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
I2  2001:DB8::23/128 [115/30]
     via FE80::A8BB:CCFF:FE01:9020, Ethernet0/2
I2  2001:DB8::25/128 [115/20]
     via FE80::A8BB:CCFF:FE01:9020, Ethernet0/2
I2  2001:DB8:175::/64 [115/20]
     via FE80::A8BB:CCFF:FE01:9020, Ethernet0/2
I2  2001:DB8:177::/64 [115/30]
     via FE80::A8BB:CCFF:FE01:9020, Ethernet0/2

```

