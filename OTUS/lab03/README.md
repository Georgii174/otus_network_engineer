# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №3
OSPF

Цель:
настроить OSPF в офисе Москва, разделить сеть на зоны и настроить фильтрацию между зонами.


Описание/Пошаговая инструкция выполнения домашнего задания:
Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.
Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.
Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.
Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.
Настройка для IPv6 повторяет логику IPv4.
План работы и изменения зафиксированы в документации.


## Выполнение:

Маршрутизаторы R14-R15 является Core layer(Ядром Моссковского офиса), она же и зоне 0 - backbone

Настройка R14:

 Настроили ospf. В ospf указываем сети которые необходимо анонсировать, у казываем зоны этих сетей. Добавляем статические маршруты по умолчанию. Так-же настраиваем ospfv3 для ipv6

```
router ospf 1
 router-id 10.255.255.14
 network 10.0.0.0 0.0.0.3 area 0
 network 10.10.25.0 0.0.0.3 area 10
 network 10.132.35.0 0.0.0.3 area 101
 network 10.255.255.14 0.0.0.0 area 0
 network 100.40.55.0 0.0.0.3 area 10
 default-information originate always
!
ip route 0.0.0.0 0.0.0.0 172.0.20.1
!
ipv6 route ::/0 2001:DB8:0:20::1
ipv6 router ospf 1
 router-id 10.255.255.14
 default-information originate always
```
На R15:

По мимо настройки ospf, настраиваем и фильтрацияю для зоны 101
```
router ospf 1
 router-id 10.255.255.15
 area 102 filter-list prefix Filter_101 in
 network 10.0.0.0 0.0.0.3 area 0
 network 10.22.33.0 0.0.0.3 area 10
 network 10.30.11.0 0.0.0.3 area 102
 network 10.40.21.0 0.0.0.3 area 10
 network 10.255.255.15 0.0.0.0 area 0
 default-information originate always
!
ip route 0.0.0.0 0.0.0.0 188.0.0.1
!
!
ip prefix-list Filter_101 seq 5 deny 10.132.35.0/30
ip prefix-list Filter_101 seq 10 deny 10.132.35.0/30 le 32
ip prefix-list Filter_101 seq 15 permit 0.0.0.0/0 le 32
ipv6 route ::/0 2001:DB8:0:21::1
ipv6 router ospf 1
 router-id 10.255.255.15
 area 102 filter-list prefix Filter_101v6 out
 default-information originate always
!
!
!
ipv6 prefix-list Filter_101v6 seq 5 deny 2001:DB8:101::/64
ipv6 prefix-list Filter_101v6 seq 10 deny 2001:DB8:101::/64 le 128
ipv6 prefix-list Filter_101v6 seq 15 permit ::/0 le 128
!
```

Маршрутизаторы R12-R13 будут являтся Distribution layer(Распределителям), и входят в зону 10. Настроки у них примерно по хожу, по этому тут будет пример только с R12
~~~
router ospf 1
 router-id 10.255.255.12
 network 10.22.33.0 0.0.0.3 area 10
 network 10.100.71.0 0.0.0.15 area 10
 network 10.255.255.12 0.0.0.0 area 10
 network 100.40.55.0 0.0.0.3 area 10
!
ipv6 router ospf 1
 router-id 10.255.255.12
~~~

Настраиваем зону 101 R19
~~~
router ospf 1
 router-id 10.255.255.19
 network 10.132.35.0 0.0.0.3 area 101
 network 10.255.255.19 0.0.0.0 area 101
!
ipv6 router ospf 1
 router-id 10.255.255.19
~~~
Настраиваем зону 102 R20
~~~
router ospf 1
 router-id 10.255.255.20
 network 10.30.11.0 0.0.0.3 area 102
 network 10.255.255.20 0.0.0.0 area 102
!
ipv6 router ospf 1
 router-id 10.255.255.20
~~~

## Итог:

R14 имеет соседство с R15, R12, R13, R19
```
R14_Core#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.255.15     1   FULL/DR         00:00:33    10.0.0.2        Ethernet1/0
10.255.255.12     1   FULL/BDR        00:00:39    100.40.55.2     Ethernet0/0
10.255.255.13     1   FULL/BDR        00:00:34    10.10.25.2      Ethernet0/1
10.255.255.19     1   FULL/DR         00:00:39    10.132.35.2     Ethernet0/3
R14_Core# show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.255.255.14/32   1     LOOP  0/0
Et1/0        1     0               10.0.0.1/30        10    BDR   1/1
Et0/0        1     10              100.40.55.1/30     10    DR    1/1
Et0/1        1     10              10.10.25.1/30      10    DR    1/1
Et0/3        1     101             10.132.35.1/30     10    BDR   1/1

```
R15 имеет соседство с R14, R12, R13, R20
```
R15_Core#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.255.255.14     1   FULL/BDR        00:00:37    10.0.0.1        Ethernet1/0
10.255.255.13     1   FULL/BDR        00:00:36    10.40.21.2      Ethernet0/0
10.255.255.12     1   FULL/BDR        00:00:38    10.22.33.2      Ethernet0/1
10.255.255.20     1   FULL/DR         00:00:38    10.30.11.2      Ethernet0/3
R15_Core#show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Lo0          1     0               10.255.255.15/32   1     LOOP  0/0
Et1/0        1     0               10.0.0.2/30        10    DR    1/1
Et0/0        1     10              10.40.21.1/30      10    DR    1/1
Et0/1        1     10              10.22.33.1/30      10    DR    1/1
Et0/3        1     102             10.30.11.1/30      10    BDR   1/1

Проверяем
R15_Core#show ip route ospf
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

      10.0.0.0/8 is variably subnetted, 18 subnets, 3 masks
O        10.10.25.0/30 [110/20] via 10.40.21.2, 00:00:51, Ethernet0/0
O        10.80.8.0/28 [110/20] via 10.40.21.2, 00:00:51, Ethernet0/0
O        10.100.71.0/28 [110/20] via 10.22.33.2, 00:00:51, Ethernet0/1
O IA     10.132.35.0/30 [110/20] via 10.0.0.1, 00:00:51, Ethernet1/0
O        10.255.255.12/32 [110/11] via 10.22.33.2, 00:00:51, Ethernet0/1
O        10.255.255.13/32 [110/11] via 10.40.21.2, 00:00:51, Ethernet0/0
O        10.255.255.14/32 [110/11] via 10.0.0.1, 00:00:51, Ethernet1/0
O IA     10.255.255.19/32 [110/21] via 10.0.0.1, 00:00:51, Ethernet1/0
O        10.255.255.20/32 [110/11] via 10.30.11.2, 00:00:51, Ethernet0/3
      100.0.0.0/30 is subnetted, 1 subnets
O        100.40.55.0 [110/20] via 10.22.33.2, 00:00:51, Ethernet0/1
R15_Core#
R15_Core#
R15_Core#show ip prefix-list Filter_101
ip prefix-list Filter_101: 3 entries
   seq 5 deny 10.132.35.0/30
   seq 10 deny 10.132.35.0/30 le 32
   seq 15 permit 0.0.0.0/0 le 32

```
По условиян задачи на R20 зона 102 не должна видить маршрут зоны 101

```
R20#
R20#show ip route 10.132.35.0
% Subnet not in table
R20#show ip rou
R20#show ip route os
R20#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.30.11.1 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.30.11.1, 00:04:21, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 14 subnets, 3 masks
O IA     10.0.0.0/30 [110/20] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.10.25.0/30 [110/30] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.22.33.0/30 [110/20] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.40.21.0/30 [110/20] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.80.8.0/28 [110/30] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.100.71.0/28 [110/30] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.255.255.12/32 [110/21] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.255.255.13/32 [110/21] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.255.255.14/32 [110/21] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.255.255.15/32 [110/11] via 10.30.11.1, 00:04:17, Ethernet0/0
O IA     10.255.255.19/32 [110/31] via 10.30.11.1, 00:04:17, Ethernet0/0
      100.0.0.0/30 is subnetted, 1 subnets
O IA     100.40.55.0 [110/30] via 10.30.11.1, 00:04:17, Ethernet0/0
R20#

```
