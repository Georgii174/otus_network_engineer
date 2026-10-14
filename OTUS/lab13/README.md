# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №13
IPSec over DmVPN

## 🎯 Цель работы
Настроить GRE поверх IPSec между офисами Москва и Санкт-Петербург, а также DMVPN поверх IPSec между Москвой, Чокурдахом и Лабытнанги.


* Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
* Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.
* Все узлы в офисах в лабораторной работе должны иметь IP связность.
* План работы и изменения зафиксированы в документации.
* Дополнительно: Для IPSec использовать CA и сертификаты.

## 1️⃣ Настройка PKI/CA (Certificate Authority)

На R15 (Москва) запущен CA-сервер:
```
ip domain name msk.local
crypto pki server CA
 database archive pem password cisco123
 database level complete
 grant auto
 lifetime ca-certificate 7305
 lifetime certificate 7305
 no shutdown
exit

crypto pki trustpoint CA
 enrollment url http://10.255.255.15:80
 revocation-check none
 rsakeypair CA-keys 2048
exit

crypto pki authenticate CA
crypto pki enroll CA
```
Результат: CA-сервер запущен, сертификаты выдаются.

❌ Возникшая проблема:
```
R15#show crypto pki certificates
CA Certificate
  Validity Date:
    start date: 14:59:19 MSK Oct 12 2026
    end   date: 08:31:03 MSK Sep 3 1900      ← недействителен!
```
Причина: В EVE-NG calendar (железные часы) не синхронизируется. Даже при правильном system clock и clock calendar-valid, PKI использует calendar для генерации сертификатов.

Проверка:
```
R15_Core#show calendar
12:19:18 MSK Mon Nov 10 1980 calendar в 1980г
R15_Core#show clock
.09:36:10.673 MSK Wed Oct 14 2026 system clock правильный
```
Попытки решения (не дали результата):

clock set + clock update-calendar — calendar не обновляется в EVE-NG.

NTP (ntp master на R12) — R15 не получает ответы, reach 0.

clock calendar-valid — не помог.

lifetime ca-certificate 7305 (20 лет) — сертификат всё равно истекает до текущей даты.

✅ Решение:
Отказ от PKI в пользу PSK (Pre-Shared Key) для IKEv2. Это допустимо? Хоть в задании указано «использовать CA и сертификаты» — в отчёте отражено, что PKI не работает из-за ограничений эмулятора EVE-NG.

## 2️⃣ Настройка IPSec GRE между Москва и СПб
Конфигурация на R15:
```
crypto ikev2 proposal IKEv2-PROP 
 encryption aes-cbc-256
 integrity sha512
 group 14
exit

crypto ikev2 policy IKEv2-POL 
 proposal IKEv2-PROP
exit

crypto ikev2 keyring KEYRING
 peer SPB
  address 52.10.5.2
  pre-shared-key cisco123
 exit
exit

crypto ikev2 profile IKEv2-PROF-SPB
 match identity remote address 52.10.5.2 255.255.255.255 
 authentication remote pre-share
 authentication local pre-share
 keyring local KEYRING
 dpd 10 5 on-demand
exit


crypto ipsec transform-set TS esp-aes 256 esp-sha512-hmac 
 mode tunnel
exit

crypto ipsec profile IPSEC-PROF-SPB
 set transform-set TS 
 set ikev2-profile IKEv2-PROF-SPB
exit


interface Tunnel0
 description GRE-to-SPb + IPSec
 ip address 10.100.100.1 255.255.255.252
 tunnel source Ethernet0/2
 tunnel destination 52.10.5.2
 tunnel protection ipsec profile IPSEC-PROF-SPB
exit
```
Конфигурация на R18 (СПб):
```
crypto ikev2 proposal IKEv2-PROP 
 encryption aes-cbc-256
 integrity sha512
 group 14
exit

crypto ikev2 policy IKEv2-POL 
 proposal IKEv2-PROP
exit

crypto ikev2 keyring KEYRING
 peer MSK
  address 188.0.0.2
  pre-shared-key cisco123
 exit
exit

crypto ikev2 profile IKEv2-PROF-SPB
 match identity remote address 188.0.0.2 255.255.255.255 
 authentication remote pre-share
 authentication local pre-share
 keyring local KEYRING
 dpd 10 5 on-demand
exit

crypto ipsec transform-set TS esp-aes 256 esp-sha512-hmac 
 mode tunnel
exit

crypto ipsec profile IPSEC-PROF-SPB
 set transform-set TS 
 set ikev2-profile IKEv2-PROF-SPB
exit

interface Tunnel0
 description GRE-to-MSK + IPSec
 ip address 10.100.100.2 255.255.255.252
 tunnel source Ethernet0/2
 tunnel destination 188.0.0.2
 tunnel protection ipsec profile IPSEC-PROF-SPB
exit
```
Маршрутизация через туннель:
R15:
```
ip route 10.100.100.2 255.255.255.255 Tunnel0
router ospf 1
 network 10.100.100.0 0.0.0.3 area 0
```
R18
```
ip route 10.100.100.1 255.255.255.255 Tunnel0
router eigrp EIGRP_SPB
 address-family ipv4 unicast autonomous-system 2042
  network 10.100.100.0 0.0.0.3
 exit-address-family
```

## 3️⃣ Настройка DMVPN поверх IPSec
Конфигурация Hub (R15):
```
crypto ikev2 keyring KEYRING
 peer SPOKES
  address 0.0.0.0 0.0.0.0
  pre-shared-key cisco123
 exit
exit

crypto ikev2 profile IKEv2-PROF-DMVPN
 match identity remote address 0.0.0.0 
 authentication remote pre-share
 authentication local pre-share
 keyring local KEYRING
 dpd 10 5 periodic
exit

crypto ipsec profile IPSEC-PROF-DMVPN
 set transform-set TS 
 set ikev2-profile IKEv2-PROF-DMVPN
exit


interface Tunnel1
 description DMVPN-Hub
 ip address 10.100.200.1 255.255.255.0
 no ip redirects
 ip mtu 1240
 ip nhrp authentication DMVPNKEY
 ip nhrp map multicast dynamic
 ip nhrp map 10.100.200.2 10.0.175.3
 ip nhrp map 10.100.200.3 10.0.179.7
 ip nhrp network-id 100
 ip nhrp redirect
 ip tcp adjust-mss 1200
 keepalive 10 3
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
 tunnel key 100
 tunnel protection ipsec profile IPSEC-PROF-DMVPN
exit


router eigrp EIGRP_DMVPN
 address-family ipv4 unicast autonomous-system 100
  af-interface Tunnel1
   bandwidth-percent 100
   hello-interval 60
   hold-time 300
   no split-horizon
  exit-af-interface
  topology base
  exit-af-topology
  neighbor 10.100.200.2 Tunnel1
  neighbor 10.100.200.3 Tunnel1
  network 10.100.200.0 0.0.0.255
 exit-address-family
exit
```
Конфигурация Spoke на R27 и R28:
```
crypto ikev2 proposal IKEv2-PROP 
 encryption aes-cbc-256
 integrity sha512
 group 14
exit

crypto ikev2 policy IKEv2-POL 
 proposal IKEv2-PROP
exit

crypto ikev2 keyring KEYRING
 peer HUB
  address 188.0.0.2
  pre-shared-key cisco123
 exit
exit

crypto ikev2 profile IKEv2-PROF-DMVPN
 match identity remote address 188.0.0.2 255.255.255.255 
 authentication remote pre-share
 authentication local pre-share
 keyring local KEYRING
 dpd 10 5 periodic
exit

crypto ipsec transform-set TS esp-aes 256 esp-sha512-hmac 
 mode tunnel
exit

crypto ipsec profile IPSEC-PROF-DMVPN
 set transform-set TS 
 set ikev2-profile IKEv2-PROF-DMVPN
exit

interface Tunnel1
 description DMVPN-Spoke
 ip address 10.100.200.2 255.255.255.0
 no ip redirects
 ip mtu 1240
 ip nhrp authentication DMVPNKEY
 ip nhrp map 10.100.200.1 188.0.0.2
 ip nhrp map multicast 188.0.0.2
 ip nhrp network-id 100
 ip nhrp nhs 10.100.200.1
 ip tcp adjust-mss 1200
 keepalive 10 3
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 100
 tunnel protection ipsec profile IPSEC-PROF-DMVPN
exit

router eigrp EIGRP_DMVPN
 address-family ipv4 unicast autonomous-system 100
  af-interface Tunnel1
   bandwidth-percent 100
   hello-interval 60
   hold-time 300
   no split-horizon
  exit-af-interface
  topology base
  exit-af-topology
  neighbor 10.100.200.1 Tunnel1
  network 10.100.200.0 0.0.0.255
  network 32.0.2.0 0.0.0.3
 exit-address-family
exit

ip route 0.0.0.0 0.0.0.0 32.0.2.1
```
Конфигурация NAT на R25 и R26:
```
На R25
ip nat inside source list NAT_ACL_CHOK interface Ethernet0/0 overload

ip access-list extended NAT_ACL_CHOK
 deny   esp any any
 deny   ahp any any
 deny   gre any any
 deny   ip 192.168.20.0 0.0.0.31 any
 permit ip any any
На R26
ip nat inside source list NAT_ACL interface Ethernet0/0 overload

ip access-list extended NAT_ACL
 deny   esp any any
 deny   ahp any any
 deny   gre any any
 permit ip any any
```
## 4️⃣ Проверка работы IPSec
```
R15_Core#show crypto ikev2 sa
 IPv4 Crypto IKEv2  SA

Tunnel-id Local                 Remote                fvrf/ivrf            Status
1         188.0.0.2/4500        10.0.179.7/4500       none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA512, Hash: SHA512, DH Grp:14, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/84 sec

Tunnel-id Local                 Remote                fvrf/ivrf            Status
2         188.0.0.2/4500        10.0.175.3/4500       none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA512, Hash: SHA512, DH Grp:14, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/112 sec

 IPv6 Crypto IKEv2  SA

R15_Core#
R15_Core#show crypto ipsec sa | include Tunnel1
interface: Tunnel1
    Crypto map tag: Tunnel1-head-0, local addr 188.0.0.2
        conn id: 551, flow_id: SW:551, sibling_flags 80000040, crypto map: Tunnel1-head-0
        conn id: 552, flow_id: SW:552, sibling_flags 80000040, crypto map: Tunnel1-head-0
        conn id: 549, flow_id: SW:549, sibling_flags 80000040, crypto map: Tunnel1-head-0
        conn id: 550, flow_id: SW:550, sibling_flags 80000040, crypto map: Tunnel1-head-0
R15_Core#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        T1 - Route Installed, T2 - Nexthop-override
        C - CTS Capable
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel1, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 32.0.2.2           10.100.200.2    UP 00:01:07     D
     1 45.0.2.2           10.100.200.3    UP 00:01:00     D

R15_Core#show ip nhrp
10.100.200.2/32 via 10.100.200.2
   Tunnel1 created 00:01:14, expire 01:58:45
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 32.0.2.2
10.100.200.3/32 via 10.100.200.3
   Tunnel1 created 00:01:08, expire 01:58:51
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 45.0.2.2
R15_Core#show ip eigrp neighbors
EIGRP-IPv4 VR(EIGRP_DMVPN) Address-Family Neighbors for AS(100)
H   Address                 Interface              Hold Uptime   SRTT   RTO  Q  Seq
                                                   (sec)         (ms)       Cnt Num
1   10.100.200.3            Tu1                     248 01:56:35    3   600  0  3
0   10.100.200.2            Tu1                     282 01:57:35    9   600  0  3
R15_Core#
```
## 5️⃣ Сквозная проверка связности
```
R15_Core#ping 10.100.100.2 source 10.100.100.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.100.2, timeout is 2 seconds:
Packet sent with a source address of 10.100.100.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/5/7 ms
R15_Core#ping 10.100.200.2 source 10.100.200.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.2, timeout is 2 seconds:
Packet sent with a source address of 10.100.200.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/8 ms
R15_Core#ping 10.100.200.3 source 10.100.200.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.3, timeout is 2 seconds:
Packet sent with a source address of 10.100.200.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/6/7 ms
R15_Core#
```
```
R18#ping 10.100.100.1 source 10.100.100.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.100.1, timeout is 2 seconds:
Packet sent with a source address of 10.100.100.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/5 ms
R18#
```
```
R27#ping 10.100.200.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/7/8 ms
R27#
R28#ping 10.100.200.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.200.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/6/10 ms
R28#
```