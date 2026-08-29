# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №6
BGP. Основы

Цель:
настроить BGP между автономными системами и организовать доступность между офисами Москва и С.-Петербург.


Описание/Пошаговая инструкция выполнения домашнего задания:
Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
Настроите eBGP между провайдерами Киторн и Ламас.
Настроите eBGP между Ламас и Триада.
Настроите eBGP между офисом С.-Петербург и провайдером Триада.
Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.
План работы и изменения зафиксированы в документации.

## Выполнение:

Настраиваем eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас (R14 и R15):
```
R14_Core#show running-config | section bgp
router bgp 1001
 bgp router-id 10.255.255.14
 bgp log-neighbor-changes
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
  neighbor 172.0.20.1 activate
  neighbor 172.0.20.1 next-hop-self

 exit-address-family

R15_Core#show running-config | section bgp
 redistribute bgp 1001 subnets
router bgp 1001
 bgp router-id 10.255.255.15
 bgp log-neighbor-changes
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
  neighbor 188.0.0.1 activate
  neighbor 188.0.0.1 next-hop-self

 exit-address-family

```
Настраиваем eBGP между провайдерами Киторн и Ламас. (R22 и R21):
```
R22# show running-config | section bgp
router bgp 101
 bgp router-id 10.255.255.22
 bgp log-neighbor-changes
 neighbor 5.83.32.2 remote-as 301
 neighbor 172.0.20.2 remote-as 1001
 !
 address-family ipv4
  network 5.83.32.0 mask 255.255.255.252
  network 10.182.0.0 mask 255.255.255.252
  network 10.255.255.22 mask 255.255.255.255
  network 172.0.20.0 mask 255.255.255.252
  neighbor 5.83.32.2 activate
  neighbor 5.83.32.2 next-hop-self
  neighbor 172.0.20.2 activate
  neighbor 172.0.20.2 next-hop-self
 exit-address-family

R21# show running-config | section bgp
router bgp 301
 bgp router-id 10.255.255.21
 bgp log-neighbor-changes
 neighbor 5.83.32.1 remote-as 101
 neighbor 31.0.0.2 remote-as 520
 neighbor 188.0.0.2 remote-as 1001
 !
 address-family ipv4
  network 5.83.32.0 mask 255.255.255.252
  network 10.255.255.21 mask 255.255.255.255
  network 31.0.0.0 mask 255.255.255.252
  network 188.0.0.0 mask 255.255.255.252
  neighbor 5.83.32.1 activate
  neighbor 5.83.32.1 next-hop-self
  neighbor 31.0.0.2 activate
  neighbor 31.0.0.2 next-hop-self
  neighbor 188.0.0.2 activate
  neighbor 188.0.0.2 next-hop-self
 exit-address-family

```
Настраиваем eBGP между Ламас и Триада (R24):
```
R24#show running-config | section bgp
router bgp 520
 bgp router-id 10.255.255.24
 bgp log-neighbor-changes
 neighbor 31.0.0.1 remote-as 301
 neighbor 52.10.5.2 remote-as 2042
 !
 address-family ipv4
  network 31.0.0.0 mask 255.255.255.252
  network 52.10.5.0 mask 255.255.255.252
  neighbor 31.0.0.1 activate
  neighbor 31.0.0.1 next-hop-self
  neighbor 52.10.5.2 activate
  neighbor 52.10.5.2 next-hop-self
 exit-address-family

```
Настраиваем eBGP между офисом С.-Петербург и провайдером Триада (R18):
Для IP доступность между пограничным роутерами офисами Москва и С.-Петербург, анонсируем сети 192.168.30.0 и 192.168.40.0 со стороны R18 и делаем так-же со стороны R14 и R15
```
R18#show running-config | section bgp
router bgp 2042
 bgp router-id 10.255.255.18
 bgp log-neighbor-changes
 neighbor 52.10.5.1 remote-as 520
 !
 address-family ipv4
  network 10.10.0.0 mask 255.255.255.0
  network 52.10.5.0 mask 255.255.255.252
  network 192.168.30.0 mask 255.255.255.224
  network 192.168.40.0 mask 255.255.255.224
  neighbor 52.10.5.1 activate
  neighbor 52.10.5.1 next-hop-self
 exit-address-family
```
Итог:

Проверяем ip доступность 
```
R14_Core#             ping 192.168.30.15 source 172.0.20.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.30.15, timeout is 2 seconds:
Packet sent with a source address of 172.0.20.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/5 ms
R14_Core#
R14_Core#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 172.0.20.1 to network 0.0.0.0

      5.0.0.0/30 is subnetted, 1 subnets
B        5.83.32.0 [20/0] via 172.0.20.1, 2d02h
      10.0.0.0/8 is variably subnetted, 20 subnets, 3 masks
B        10.182.0.0/30 [20/0] via 172.0.20.1, 2d02h
B        10.255.255.21/32 [20/0] via 172.0.20.1, 2d02h
B        10.255.255.22/32 [20/0] via 172.0.20.1, 2d02h
      31.0.0.0/30 is subnetted, 1 subnets
B        31.0.0.0 [20/0] via 172.0.20.1, 2d02h
      52.0.0.0/30 is subnetted, 1 subnets
B        52.10.5.0 [20/0] via 172.0.20.1, 2d02h
      188.0.0.0/30 is subnetted, 1 subnets
B        188.0.0.0 [20/0] via 172.0.20.1, 2d02h
      192.168.30.0/27 is subnetted, 1 subnets
B        192.168.30.0 [20/0] via 172.0.20.1, 19:56:10
      192.168.40.0/27 is subnetted, 1 subnets
B        192.168.40.0 [20/0] via 172.0.20.1, 19:55:40

R15_Core#show ip route bgp
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

      5.0.0.0/30 is subnetted, 1 subnets
B        5.83.32.0 [20/0] via 188.0.0.1, 2d18h
      10.0.0.0/8 is variably subnetted, 21 subnets, 3 masks
B        10.182.0.0/30 [20/0] via 188.0.0.1, 2d18h
B        10.255.255.21/32 [20/0] via 188.0.0.1, 2d02h
B        10.255.255.22/32 [20/0] via 188.0.0.1, 2d18h
      31.0.0.0/30 is subnetted, 1 subnets
B        31.0.0.0 [20/0] via 188.0.0.1, 2d18h
      52.0.0.0/30 is subnetted, 1 subnets
B        52.10.5.0 [20/0] via 188.0.0.1, 2d18h
      172.0.0.0/30 is subnetted, 1 subnets
B        172.0.20.0 [20/0] via 188.0.0.1, 2d18h
      192.168.30.0/27 is subnetted, 1 subnets
B        192.168.30.0 [20/0] via 188.0.0.1, 19:57:43
      192.168.40.0/27 is subnetted, 1 subnets
B        192.168.40.0 [20/0] via 188.0.0.1, 19:57:12
R15_Core#ping 192.168.30.15 source 188.0.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.30.15, timeout is 2 seconds:
Packet sent with a source address of 188.0.0.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/4 ms

R14_Core# show ip bgp
BGP table version is 17, local router ID is 10.255.255.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  5.83.32.0/30     172.0.20.1               0             0 101 i
 *>  10.10.25.0/30    0.0.0.0                  0         32768 i
 *>  10.30.11.0/30    10.0.0.2                20         32768 i
 *>  10.80.8.0/28     10.10.25.2              20         32768 i
 *>  10.100.71.0/28   100.40.55.2             20         32768 i
 *>  10.132.35.0/30   0.0.0.0                  0         32768 i
 *>  10.182.0.0/30    172.0.20.1               0             0 101 i
 *>  10.255.255.14/32 0.0.0.0                  0         32768 i
 *>  10.255.255.21/32 172.0.20.1                             0 101 301 i
 *>  10.255.255.22/32 172.0.20.1               0             0 101 i
 *>  31.0.0.0/30      172.0.20.1                             0 101 301 i
 *>  52.10.5.0/30     172.0.20.1                             0 101 301 520 i
 r>  172.0.20.0/30    172.0.20.1               0             0 101 i
 *>  188.0.0.0/30     172.0.20.1                             0 101 301 i
 *>  192.168.30.0/27  172.0.20.1                             0 101 301 520 2042 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.40.0/27  172.0.20.1                             0 101 301 520 2042 i


R15_Core#show ip bgp
BGP table version is 17, local router ID is 10.255.255.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  5.83.32.0/30     188.0.0.1                0             0 301 i
 *>  10.10.25.0/30    10.40.21.2              20         32768 i
 *>  10.30.11.0/30    0.0.0.0                  0         32768 i
 *>  10.80.8.0/28     10.40.21.2              20         32768 i
 *>  10.100.71.0/28   10.22.33.2              20         32768 i
 *>  10.132.35.0/30   10.0.0.1                20         32768 i
 *>  10.182.0.0/30    188.0.0.1                              0 301 101 i
 *>  10.255.255.15/32 0.0.0.0                  0         32768 i
 *>  10.255.255.21/32 188.0.0.1                0             0 301 i
 *>  10.255.255.22/32 188.0.0.1                              0 301 101 i
 *>  31.0.0.0/30      188.0.0.1                0             0 301 i
 *>  52.10.5.0/30     188.0.0.1                              0 301 520 i
 *>  172.0.20.0/30    188.0.0.1                              0 301 101 i
 r>  188.0.0.0/30     188.0.0.1                0             0 301 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.30.0/27  188.0.0.1                              0 301 520 2042 i
 *>  192.168.40.0/27  188.0.0.1                              0 301 520 2042 i

```

