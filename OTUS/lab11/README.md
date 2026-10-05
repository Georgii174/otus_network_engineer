# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №11

## 🎯 Цель работы
Настроить GRE между офисами Москва и Санкт-Петербург, а также DMVPN между Москвой, Чокурдахом и Лабытнанги.

* Настроите GRE между офисами Москва и С.-Петербург.
* Настроите DMVMN между Москва и Чокурдах, Лабытнанги.
* Все узлы в офисах в лабораторной работе должны иметь IP связность.
* План работы и изменения зафиксированы в документации.

## 1️⃣ GRE между офисами Москва и СПб

### Настройка на R15 (Москва):
```
Создаем тунель
interface Tunnel0
 description GRE-to-SPb
 ip address 10.100.100.1 255.255.255.252
 tunnel source Ethernet0/2
 tunnel destination 52.10.5.2
 tunnel mode gre ip
 no shutdown


Маршрут до туннельной сети СПб
ip route 10.100.100.2 255.255.255.255 Tunnel0
```
### Настройка на R18 (СПб):
```
interface Tunnel0
 description GRE-to-Moscow
 ip address 10.100.100.2 255.255.255.252
 tunnel source Ethernet0/2
 tunnel destination 188.0.0.2
 tunnel mode gre ip
 no shutdown
exit

ip route 10.100.100.1 255.255.255.255 Tunnel0
exit
```
## 2️⃣ DMVPN между Москвой, Чокурдахом и Лабытнанги

### Настройка Hub — R15 (Москва):
```
Создаем туннель
interface Tunnel1
 description DMVPN-Hub
 ip address 10.100.200.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp authentication DMVPNKEY
 ip nhrp network-id 100
 ip nhrp map multicast dynamic
 ip nhrp redirect
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
 tunnel key 100
 no shutdown

EIGRP для DMVPN
router eigrp EIGRP_DMVPN
 address-family ipv4 unicast autonomous-system 100
  network 10.100.200.0 0.0.0.255
  af-interface Tunnel1
   no split-horizon
  exit-af-interface
 exit-address-family
```
### Настройка R27 и R28:
```
 Создаем туннель
interface Tunnel1
 description DMVPN-Spoke-to-Moscow
 ip address 10.100.200.2 255.255.255.0
 ip mtu 1400
 ip nhrp authentication DMVPNKEY
 ip nhrp network-id 100
 ip nhrp map 10.100.200.1 188.0.0.2
 ip nhrp map multicast 188.0.0.2
 ip nhrp nhs 10.100.200.1
 ip nhrp registration timeout 60
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 100
 no shutdown

EIGRP для DMVPN
router eigrp EIGRP_DMVPN
 address-family ipv4 unicast autonomous-system 100
  network 10.100.200.0 0.0.0.255
  network 32.0.2.0 0.0.0.3
 exit-address-family
exit
```

## 3️⃣ Проверка связности
```
R15_Core#ping 10.100.100.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.100.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/7/30 ms
R15_Core#ping 10.100.200.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R15_Core#ping 10.100.200.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms
R15_Core#

R18#ping 10.100.100.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.100.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
R18#
R27#ping 10.100.200.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R27#ping 10.100.200.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/5/9 ms
R27#
R28#ping 10.100.200.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms
R28#ping 10.100.200.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
R28#

```