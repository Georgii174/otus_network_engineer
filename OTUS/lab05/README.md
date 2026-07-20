# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №5
EIGRP

Цель:
настроить EIGRP named-mode для IPv4 и IPv6 в офисе С.-Петербург.


Описание/Пошаговая инструкция выполнения домашнего задания:
В офисе С.-Петербург настроить EIGRP.
R32 получает только маршрут по умолчанию.
R16-17 анонсируют только суммарные префиксы.
Использовать EIGRP named-mode для настройки сети.
Настройка осуществляется одновременно для IPv4 и IPv6.

## Выполнение:

R16 (суммаризация + фильтрация)

```
router eigrp EIGRP_SPB
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  af-interface Ethernet0/0
   summary-address 192.168.40.0 255.255.255.0
  exit-af-interface
  !
  topology base
   distribute-list 1 out Ethernet0/3
  exit-af-topology
  network 10.10.2.0 0.0.0.3
  network 10.10.3.0 0.0.0.3
  network 10.10.8.0 0.0.0.3
  network 10.255.255.16 0.0.0.0
  eigrp router-id 10.255.255.16
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 2042
  !
  af-interface Ethernet0/0
   summary-address 2001:DB8:270::/64
  exit-af-interface
  !
  topology base
   distribute-list prefix-list FILTER_IPv6 out Ethernet0/3
  exit-af-topology
  eigrp router-id 10.255.255.16
 exit-address-family

```
ACL для фильтрации (разрешает только default):
```
access-list 1 permit 0.0.0.0
access-list 1 deny any
```
R17 (суммаризация)

```
router eigrp EIGRP_SPB
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  af-interface Ethernet0/0
   summary-address 192.168.30.0 255.255.255.0
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 10.10.1.0 0.0.0.3
  network 10.10.7.0 0.0.0.3
  network 10.255.255.17 0.0.0.0
  eigrp router-id 10.255.255.17
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 2042
  !
  af-interface Ethernet0/0
   summary-address 2001:DB8:260::/64
  exit-af-interface
  !
  topology base
  exit-af-topology
  eigrp router-id 10.255.255.17
 exit-address-family
!
```
R18 (перераспределение default)
```
router eigrp EIGRP_SPB
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  topology base
   redistribute static
  exit-af-topology
  network 10.10.1.0 0.0.0.3
  network 10.10.2.0 0.0.0.3
  network 10.255.255.18 0.0.0.0
  eigrp router-id 10.255.255.18
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 2042
  !
  topology base
   redistribute static
  exit-af-topology
  eigrp router-id 10.255.255.18
 exit-address-family
!
```
R32 (stub)
```
router eigrp EIGRP_SPB
 !
 address-family ipv4 unicast autonomous-system 2042
  !
  topology base
  exit-af-topology
  network 10.10.3.0 0.0.0.3
  network 10.255.255.32 0.0.0.0
  eigrp router-id 10.255.255.32
  eigrp stub receive-only
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 2042
  !
  topology base
  exit-af-topology
  eigrp router-id 10.255.255.32
  eigrp stub receive-only
 exit-address-family
!
```

### Итого:

Проверка R32 (получает только default)
 ```
 
Gateway of last resort is 10.10.3.1 to network 0.0.0.0

D*EX  0.0.0.0/0 [170/2048000] via 10.10.3.1, 00:19:54, Ethernet0/0
```
Проверка EIGRP-соседств:
```
R16# show ip eigrp neighbors
EIGRP-IPv4 VR(EIGRP_SPB) Address-Family Neighbors for AS(2042)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.10.3.2               Et0/3                    10 01:36:00    7   100  0  30
0   10.10.2.1               Et0/1                    12 01:36:04    5   100  0  52
```

