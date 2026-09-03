# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №7
# Настройка iBGP, Route Reflector и оптимизация трафика

## Выполненные работы:

### 1. iBGP в офисе Москва (R14-R15)
* Настроено iBGP-соседство между R14 и R15 (AS 1001).
* Использован атрибут `next-hop-self` для корректной внутренней маршрутизации.
* Анонсированы локальные сети Москвы.
```
R14_Core#show running-config | section bgp
router bgp 1001
 bgp router-id 10.255.255.14
 bgp log-neighbor-changes
 neighbor 10.0.0.2 remote-as 1001
 neighbor 172.0.20.1 remote-as 101
 !
 address-family ipv4
  network 10.0.0.0 mask 255.255.255.0
  network 10.10.25.0 mask 255.255.255.252
  network 10.30.11.0 mask 255.255.255.252
  network 10.80.8.0 mask 255.255.255.240
  network 10.100.71.0 mask 255.255.255.240
  network 10.132.35.0 mask 255.255.255.252
  network 10.255.255.14 mask 255.255.255.255
  network 192.168.55.0 mask 255.255.255.224
  network 192.168.62.0 mask 255.255.255.224
  neighbor 10.0.0.2 activate
  neighbor 10.0.0.2 next-hop-self
  neighbor 172.0.20.1 activate
  neighbor 172.0.20.1 next-hop-self
 exit-address-family

R15_Core#show running-config | section bgp
 redistribute bgp 1001 subnets
router bgp 1001
 bgp router-id 10.255.255.15
 bgp log-neighbor-changes
 neighbor 10.0.0.1 remote-as 1001
 neighbor 188.0.0.1 remote-as 301
 !
 address-family ipv4
  network 10.0.0.0 mask 255.255.255.0
  network 10.10.25.0 mask 255.255.255.252
  network 10.30.11.0 mask 255.255.255.252
  network 10.80.8.0 mask 255.255.255.240
  network 10.100.71.0 mask 255.255.255.240
  network 10.132.35.0 mask 255.255.255.252
  network 10.255.255.15 mask 255.255.255.255
  network 192.168.55.0 mask 255.255.255.224
  network 192.168.62.0 mask 255.255.255.224
  neighbor 10.0.0.1 activate
  neighbor 10.0.0.1 next-hop-self
  neighbor 188.0.0.1 activate
  neighbor 188.0.0.1 next-hop-self
  neighbor 188.0.0.1 route-map PREFER_LAMAS in
 exit-address-family

```

### 2. iBGP в Триаде с Route Reflector (RR)
* Маршрутизатор R24 (AS 520) назначен как **Route Reflector**.
* Клиентами RR назначены R23, R25, R26.
* Настроены eBGP-сессии между Триадой и смежными AS (Киторн, Ламас, СПб).
```
router bgp 520
 bgp router-id 10.255.255.24
 bgp log-neighbor-changes
 neighbor 10.0.175.3 remote-as 520
 neighbor 10.0.177.2 remote-as 520
 neighbor 10.0.179.7 remote-as 520
 neighbor 31.0.0.1 remote-as 301
 neighbor 52.10.5.2 remote-as 2042
 !
 address-family ipv4
  network 31.0.0.0 mask 255.255.255.252
  network 52.10.5.0 mask 255.255.255.252
  neighbor 10.0.175.3 activate
  neighbor 10.0.175.3 route-reflector-client
  neighbor 10.0.175.3 next-hop-self
  neighbor 10.0.177.2 activate
  neighbor 10.0.177.2 route-reflector-client
  neighbor 10.0.177.2 next-hop-self
  neighbor 10.0.179.7 activate
  neighbor 10.0.179.7 route-reflector-client
  neighbor 10.0.179.7 next-hop-self
  neighbor 31.0.0.1 activate
  neighbor 52.10.5.2 activate
  neighbor 52.10.5.2 next-hop-self
 exit-address-family
!
```

### 3. Приоритет провайдера Ламас для Москвы
* Создан Route-map `PREFER_LAMAS` с `set local-preference 200`.
* Route-map применен на вход (in) от соседа Ламас (R21) на R15.
* Исходящий трафик Москвы теперь предпочтительно идет через Ламас.
```
  R15
  neighbor 188.0.0.1 next-hop-self
  neighbor 188.0.0.1 route-map PREFER_LAMAS in
```

### 4. Балансировка трафика в СПб
* На R18 настроен `maximum-paths 2` и `bgp bestpath as-path multipath-relax`.
* Трафик до внешних офисов распределяется по двум линкам в Триаду одновременно (через R24 и R26).
```
router bgp 2042
 bgp router-id 10.255.255.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 neighbor 47.32.2.1 remote-as 520
 neighbor 52.10.5.1 remote-as 520
 !
 address-family ipv4
  network 10.10.0.0 mask 255.255.255.0
  network 52.10.5.0 mask 255.255.255.252
  network 192.168.30.0 mask 255.255.255.224
  network 192.168.40.0 mask 255.255.255.224
  neighbor 47.32.2.1 activate
  neighbor 52.10.5.1 activate
  neighbor 52.10.5.1 next-hop-self
  maximum-paths 2
 exit-address-family
!
```

### 5. IP-связность
* Проверена таблица маршрутизации на всех граничных роутерах.
* Выполнены тесты `ping` между сетевыми устройствами.
* Полная связность достигнута.


## Проверка:
* `show ip bgp summary` - проверка соседств (должны быть Established).
* `show ip bgp` - проверка наличия всех маршрутов.
* `show ip route` - проверка полученных маршрутов в таблице.
* `ping` - проверка сквозной IP-связности.

Проверяем доступность сквозной IP-связности
```
VPCS> ping 192.168.55.22

84 bytes from 192.168.55.22 icmp_seq=1 ttl=58 time=8.371 ms
84 bytes from 192.168.55.22 icmp_seq=2 ttl=58 time=3.208 ms
84 bytes from 192.168.55.22 icmp_seq=3 ttl=58 time=3.631 ms
84 bytes from 192.168.55.22 icmp_seq=4 ttl=58 time=4.490 ms
84 bytes from 192.168.55.22 icmp_seq=5 ttl=58 time=6.222 ms

VPCS> ping 192.168.62.19

84 bytes from 192.168.62.19 icmp_seq=1 ttl=58 time=7.033 ms
84 bytes from 192.168.62.19 icmp_seq=2 ttl=58 time=3.430 ms
84 bytes from 192.168.62.19 icmp_seq=3 ttl=58 time=4.906 ms
84 bytes from 192.168.62.19 icmp_seq=4 ttl=58 time=6.324 ms
84 bytes from 192.168.62.19 icmp_seq=5 ttl=58 time=4.840 ms

VPCS> ping 192.168.30.15

84 bytes from 192.168.30.15 icmp_seq=1 ttl=61 time=12.649 ms
84 bytes from 192.168.30.15 icmp_seq=2 ttl=61 time=2.935 ms
84 bytes from 192.168.30.15 icmp_seq=3 ttl=61 time=2.713 ms
84 bytes from 192.168.30.15 icmp_seq=4 ttl=61 time=3.071 ms
84 bytes from 192.168.30.15 icmp_seq=5 ttl=61 time=3.881 ms

VPCS>
```
Проверка маршрутов iBGP в офисе Москва
```
R15_Core#show ip bgp summary
BGP router identifier 10.255.255.15, local AS number 1001
BGP table version is 20, main routing table version 20
19 network entries using 2736 bytes of memory
26 path entries using 2080 bytes of memory
12/7 BGP path/bestpath attribute entries using 1824 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 6736 total bytes of memory
BGP activity 19/0 prefixes, 30/4 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.1        4         1001    1113    1111       20    0    0 16:43:37        8
188.0.0.1       4          301    1118    1110       20    0    0 16:43:36       10
R15_Core#show ip bgp
BGP table version is 20, local router ID is 10.255.255.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  5.83.32.0/30     188.0.0.1                0    200      0 301 i
 * i 10.10.25.0/30    10.0.0.1                 0    100      0 i
 *>                   10.40.21.2              20         32768 i
 * i 10.30.11.0/30    10.0.0.1                20    100      0 i
 *>                   0.0.0.0                  0         32768 i
 * i 10.80.8.0/28     10.0.0.1                20    100      0 i
 *>                   10.40.21.2              20         32768 i
 * i 10.100.71.0/28   10.0.0.1                20    100      0 i
 *>                   10.22.33.2              20         32768 i
 * i 10.132.35.0/30   10.0.0.1                 0    100      0 i
 *>                   10.0.0.1                20         32768 i
 *>  10.182.0.0/30    188.0.0.1                     200      0 301 101 i
 r>i 10.255.255.14/32 10.0.0.1                 0    100      0 i
 *>  10.255.255.15/32 0.0.0.0                  0         32768 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.255.255.21/32 188.0.0.1                0    200      0 301 i
 *>  10.255.255.22/32 188.0.0.1                     200      0 301 101 i
 *>  31.0.0.0/30      188.0.0.1                0    200      0 301 i
 *>  52.10.5.0/30     188.0.0.1                     200      0 301 520 i
 *>  172.0.20.0/30    188.0.0.1                     200      0 301 101 i
 r>  188.0.0.0/30     188.0.0.1                0    200      0 301 i
 *>  192.168.30.0/27  188.0.0.1                     200      0 301 520 2042 i
 *>  192.168.40.0/27  188.0.0.1                     200      0 301 520 2042 i
 * i 192.168.55.0/27  10.0.0.1                20    100      0 i
 *>                   10.22.33.2              20         32768 i
 * i 192.168.62.0/27  10.0.0.1                20    100      0 i
 *>                   10.40.21.2              20         32768 i
R15_Core#show ip rou
R15_Core#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 188.0.0.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 188.0.0.1
      5.0.0.0/30 is subnetted, 1 subnets
B        5.83.32.0 [20/0] via 188.0.0.1, 16:43:28
      10.0.0.0/8 is variably subnetted, 21 subnets, 3 masks
C        10.0.0.0/30 is directly connected, Ethernet1/0
L        10.0.0.2/32 is directly connected, Ethernet1/0
O        10.10.25.0/30 [110/20] via 10.40.21.2, 16:43:38, Ethernet0/0
C        10.22.33.0/30 is directly connected, Ethernet0/1
L        10.22.33.1/32 is directly connected, Ethernet0/1
C        10.30.11.0/30 is directly connected, Ethernet0/3
L        10.30.11.1/32 is directly connected, Ethernet0/3
C        10.40.21.0/30 is directly connected, Ethernet0/0
L        10.40.21.1/32 is directly connected, Ethernet0/0
O        10.80.8.0/28 [110/20] via 10.40.21.2, 16:43:48, Ethernet0/0
O        10.100.71.0/28 [110/20] via 10.22.33.2, 16:43:48, Ethernet0/1
O IA     10.132.35.0/30 [110/20] via 10.0.0.1, 16:43:48, Ethernet1/0
B        10.182.0.0/30 [20/0] via 188.0.0.1, 16:43:28
O        10.255.255.12/32 [110/11] via 10.22.33.2, 16:43:48, Ethernet0/1
O        10.255.255.13/32 [110/11] via 10.40.21.2, 16:43:48, Ethernet0/0
O        10.255.255.14/32 [110/11] via 10.0.0.1, 16:43:48, Ethernet1/0
C        10.255.255.15/32 is directly connected, Loopback0
O IA     10.255.255.19/32 [110/21] via 10.0.0.1, 16:43:48, Ethernet1/0
O        10.255.255.20/32 [110/11] via 10.30.11.2, 16:43:48, Ethernet0/3
B        10.255.255.21/32 [20/0] via 188.0.0.1, 16:43:28
B        10.255.255.22/32 [20/0] via 188.0.0.1, 16:43:28
      31.0.0.0/30 is subnetted, 1 subnets
B        31.0.0.0 [20/0] via 188.0.0.1, 16:43:28
      52.0.0.0/30 is subnetted, 1 subnets
B        52.10.5.0 [20/0] via 188.0.0.1, 16:42:57
      100.0.0.0/30 is subnetted, 1 subnets
O        100.40.55.0 [110/20] via 10.22.33.2, 16:43:38, Ethernet0/1
      172.0.0.0/30 is subnetted, 1 subnets
B        172.0.20.0 [20/0] via 188.0.0.1, 16:43:28
      188.0.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        188.0.0.0/30 is directly connected, Ethernet0/2
L        188.0.0.2/32 is directly connected, Ethernet0/2
      192.168.30.0/27 is subnetted, 1 subnets
B        192.168.30.0 [20/0] via 188.0.0.1, 16:42:57
      192.168.40.0/27 is subnetted, 1 subnets
B        192.168.40.0 [20/0] via 188.0.0.1, 16:42:57
      192.168.55.0/27 is subnetted, 1 subnets
O        192.168.55.0 [110/20] via 10.22.33.2, 16:43:48, Ethernet0/1
      192.168.62.0/27 is subnetted, 1 subnets
O        192.168.62.0 [110/20] via 10.40.21.2, 00:03:21, Ethernet0/0

```