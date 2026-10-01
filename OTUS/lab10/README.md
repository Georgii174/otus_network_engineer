# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №9
Основные протоколы сети интернет

## Цель:
настроить DHCP и синхронизацию времени в офисе Москва, а также NAT в офисах Москва, C.-Перетбруг и Чокурдах.

* Настроите NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
* Настроите NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
* Настроите статический NAT для R20.
* Настроите NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
* 5*. Настроите статический NAT(PAT) для офиса Чокурдах.
* Настроите для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
* Настроите NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.
* Все офисы в лабораторной работе должны иметь IP связность.

## Выполненные работы

## 1️⃣ NAT(PAT) на R14 и R15 (Москва)

### R14 (Киторн, AS 101)
ip nat inside вешаем на внутренние линки
```
interface Ethernet0/1
 description Link to R13 (Zone 10)
 ip address 10.10.25.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address 2001:DB8:10:2::1/64
 ipv6 ospf 1 area 10
```
и  ip nat outside на внешний линк
```
interface Ethernet0/2
 description UPLink R22-R14
 ip address 172.0.20.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address 2001:DB8:0:20::2/64
```
NAT:
```
PAT для всех внутренних сетей
access-list 10 permit 10.0.0.0 0.255.255.255
access-list 10 permit 100.64.0.0 0.63.255.255
access-list 10 permit 192.168.0.0 0.0.255.255
ip nat inside source list 10 interface Ethernet0/2 overload

Port forwarding для R19 (Telnet)
ip nat inside source static tcp 10.132.35.2 23 interface Ethernet0/2 23

Маршрут по умолчанию — через R15 (весь трафик через R15)
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```
### R15 (Ламас, AS 301)
Делаем аналогичко на на R14
```
interface Ethernet0/1
 description Link to R12 (Zone 10)
 ip address 10.22.33.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address 2001:DB8:10:4::1/64
 ipv6 ospf 1 area 10

interface Ethernet0/2
 description UPLink R21-R15
 ip address 188.0.0.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 ipv6 address 2001:DB8:0:21::2/64

```
NAT:
```
PAT для всех внутренних сетей
access-list 10 permit 10.0.0.0 0.255.255.255
access-list 10 permit 100.64.0.0 0.63.255.255
access-list 10 permit 192.168.0.0 0.0.255.255
ip nat inside source list 10 interface Ethernet0/2 overload

Port forwarding для R19 (Telnet)
ip nat inside source static tcp 10.132.35.2 23 interface Ethernet0/2 23

Статический NAT для R20
ip nat inside source static 10.30.11.2 188.0.0.10

Маршрут по умолчанию — через R21 (Ламас)
ip route 0.0.0.0 0.0.0.0 188.0.0.1

Маршрут до R19 (для port forwarding)
ip route 10.132.35.0 255.255.255.252 10.0.0.1
```
## 2️⃣ NAT(PAT) на R18 (СПб) — пул из 5 адресов

### На R18:
```
interface Ethernet0/1
 description UPlink R18-R17
 ip address 10.10.1.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ipv6 address 2001:DB8:1:1::1/64
!
interface Ethernet0/2
 description link R24-R18
 ip address 52.10.5.2 255.255.255.252
 ip nat outside(У нас два внешних порта, ip nat outside вешаем на оба порат)
 ip virtual-reassembly in
 ipv6 address 2001:DB8:0:52::2/64
!
interface Ethernet0/3
 description link R26-R18
 ip address 47.32.2.2 255.255.255.252
 ip nat outside(У нас два внешних порта, ip nat outside вешаем на оба порат)
 ip virtual-reassembly in
 ipv6 address 2001:DB8:0:47::2/64


```
NAT:
```
Пул из 5 адресов AS 2042
ip nat pool SPB_POOL 100.64.0.1 100.64.0.5 netmask 255.255.255.248

ACL для внутренних сетей
access-list 20 permit 10.0.0.0 0.255.255.255
access-list 20 permit 192.168.30.0 0.0.0.31
access-list 20 permit 192.168.40.0 0.0.0.31

PAT с использованием пула
ip nat inside source list 20 pool SPB_POOL overload
```
## 3️⃣ Статический NAT для R20 (Москва)

### На R15:
```
R20 имеет внутренний адрес 10.30.11.2
Публичный адрес: 188.0.0.10
ip nat inside source static 10.30.11.2 188.0.0.10

R15_Core#show ip nat translations | include 10.30.11.2
--- 188.0.0.10         10.30.11.2         ---                ---
R15_Core#

```
## 4️⃣ NAT для доступа к R19 (remote management)

### Логика: R19 доступен извне через port forwarding (Telnet) на внешних IP роутеров R14 и R15.

На R14:
```
ip nat inside source static tcp 10.132.35.2 23 interface Ethernet0/2 23
Доступ: telnet 172.0.20.2
```
На R15:
```
ip nat inside source static tcp 10.132.35.2 23 interface Ethernet0/2 23
Доступ: telnet 188.0.0.2
```
Настройка Telnet на R19:
```
enable secret class
line vty 0 4
 password cisco
 login
 transport input telnet
```
Важно добавить маршруты для обратного трафика:
```
На R15
ip route 10.132.35.0 255.255.255.252 10.0.0.1

На R21 (для доступа из Чокурдах)
ip route 32.0.5.0 255.255.255.252 31.0.0.2
```
Проверка:
```
R24#telnet 188.0.0.2
Trying 188.0.0.2 ... Open


User Access Verification

Password:
```
## 5️⃣ Статический NAT(PAT) для офиса Чокурдах

### На R25: 
На внешнии линки вешаем ip nat outside 
на внетрении ip nat inside
```
interface Ethernet0/2
 description link R25-R26 (Zona 26)
 ip address 10.0.180.4 255.255.255.240
 ip router isis
 ip nat outside
 ip virtual-reassembly in
 ipv6 address 2001:DB8:180::4/64
 ipv6 router isis
 isis network point-to-point

interface Ethernet0/3
 description Link R25-R28
 ip address 32.0.5.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
```
## 6️⃣ DHCP-сервер на R12 и R13 (Москва)

### На R12 и R13 (VLAN 280 VLAN 290):
```
service dhcp (включаем DHCP‑сервер)

ip dhcp excluded-address 192.168.55.1 192.168.55.5 (исключаем из пула зарезервированные адреса)

ip dhcp pool VLAN280 (создаем пул и вешаем его на vlan)
 network 192.168.55.0 255.255.255.224
 default-router 192.168.55.1
 dns-server 8.8.8.8 8.8.4.4
 domain-name mck.local
 lease 7
```
Сразу можно проверить:
```
R12#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.55.6        0100.5079.6668.01       Oct 07 2026 07:58 AM    Automatic
R12#show ip dhcp pool

Pool vlan280 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 30
 Leased addresses               : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.55.7         192.168.55.1     - 192.168.55.30     1
R12#
```
Включаем DHCP на VPC и проверяем :
```
VPCS> ip dhcp
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.55.6/27
GATEWAY     : 192.168.55.1
DNS         : 8.8.8.8  8.8.4.4
DHCP SERVER : 192.168.55.1
DHCP LEASE  : 515198, 604800/302400/529200
DOMAIN NAME : mck.local
MAC         : 00:50:79:66:68:01
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```
## 7️⃣ NTP-сервер на R12 и R13 (Москва)

### На R12 и R13:
```
NTP-мастер (stratum 2)
ntp master 2

Часовой пояс
clock timezone MSK 3 0
clock calendar-valid
```
На клиентах офис Москва:
```
ntp server 10.255.255.12
ntp server 10.255.255.13

clock timezone MSK 3 0
clock calendar-valid
```
Проверка:
```
R20#show ntp status
Clock is synchronized, stratum 3, reference is 10.255.255.13
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz, precision is 2**10
ntp uptime is 8829600 (1/100 of seconds), resolution is 4000
reference time is EE686F54.D5C291A8 (08:34:12.835 MSK Thu Oct 1 2026)
clock offset is -0.5000 msec, root delay is 1.00 msec
root dispersion is 24.15 msec, peer dispersion is 2.02 msec
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is 0.000000058 s/s
system poll interval is 1024, last update was 1228 sec ago.
R20#show ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
+~10.255.255.12   127.127.1.1      2    258   1024   377  1.000  -0.500  1.975
*~10.255.255.13   127.127.1.1      2    169   1024   377  1.000  -0.500  2.028
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
R20#show clock
08:54:41.411 MSK Thu Oct 1 2026
R20#
```
## 8️⃣ IP-связность между всеми офисами

### Проверка связности:
```
PING с Мостквы до СПБ
VPCS> ping 192.168.62.1
84 bytes from 192.168.62.1 icmp_seq=1 ttl=253 time=5.666 ms
84 bytes from 192.168.62.1 icmp_seq=2 ttl=253 time=6.917 ms
84 bytes from 192.168.62.1 icmp_seq=3 ttl=253 time=2.393 ms
VPCS> ping 192.168.62.6

84 bytes from 192.168.62.6 icmp_seq=1 ttl=61 time=13.553 ms
84 bytes from 192.168.62.6 icmp_seq=2 ttl=61 time=10.184 ms
84 bytes from 192.168.62.6 icmp_seq=3 ttl=61 time=8.299 ms
PING с СПБ до Мостквы

VPCS> ping 192.168.55.6

84 bytes from 192.168.55.6 icmp_seq=1 ttl=58 time=15.623 ms
84 bytes from 192.168.55.6 icmp_seq=2 ttl=58 time=5.131 ms
84 bytes from 192.168.55.6 icmp_seq=3 ttl=58 time=10.488 ms
84 bytes from 192.168.55.6 icmp_seq=4 ttl=58 time=10.125 ms
84 bytes from 192.168.55.6 icmp_seq=5 ttl=58 time=10.773 ms

VPCS>

```