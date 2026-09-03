# Репозиторий лабораторных работ курса "Сетевой инженер" в OTUS.ru

## Домашнее задание №8
 BGP. Фильтрация

## Выполненные работы:
### 1. Фильтрация транзитного трафика в офисе Москва (AS 1001)
* На R14 и R15 настраеваем фильстрацию
```
R14:
Создаем AS-Path ACL для блокировки транзита
ip as-path access-list 1 permit ^$
ip as-path access-list 1 deny ^(101|301)_
Создаем Route-map для исходящего трафика
route-map BLOCK_TRAN permit 10
 match as-path 1
! добавляем в router bgp 1001
  neighbor 172.0.20.1 route-map BLOCK_TRAN out
!
```
На R15 делаем аналошично.
Проверяем
```
R14_Core#show ip bgp
BGP table version is 40, local router ID is 10.255.255.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.10.25.0/30    10.0.0.2                20    100      0 i
 *>                   0.0.0.0                  0         32768 i
 * i 10.30.11.0/30    10.0.0.2                 0    100      0 i
 *>                   10.0.0.2                20         32768 i
 * i 10.80.8.0/28     10.0.0.2                20    100      0 i
 *>                   10.10.25.2              20         32768 i
 * i 10.100.71.0/28   10.0.0.2                20    100      0 i
 *>                   100.40.55.2             20         32768 i
 * i 10.132.35.0/30   10.0.0.2                20    100      0 i
 *>                   0.0.0.0                  0         32768 i
 *>  10.255.255.14/32 0.0.0.0                  0         32768 i
 r>i 10.255.255.15/32 10.0.0.2                 0    100      0 i
 r>i 192.168.30.0/27  10.0.0.2                 0    200      0 301 520 2042 i
 r>i 192.168.40.0/27  10.0.0.2                 0    200      0 301 520 2042 i
     Network          Next Hop            Metric LocPrf Weight Path
 * i 192.168.55.0/27  10.0.0.2                20    100      0 i
 *>                   100.40.55.2             20         32768 i
 * i 192.168.62.0/27  10.0.0.2                20    100      0 i
 *>                   10.10.25.2              20         32768 i

```
### 2. Фильтрация транзитного трафика в офисе СПб (AS 2042)
* На R18 настроен Prefix-list, который разрешает только сети 192.168.30.0/27 и 192.168.40.0/27.
* Маршруты, полученные от Триады, не анонсируются обратно.
```
Создаем Prefix-list, разрешающий только локальные сети
ip prefix-list LOCAL_NET seq 5 permit 192.168.30.0/27
ip prefix-list LOCAL_NET seq 10 permit 192.168.40.0/27
Создаем Route-map для исходящего трафика
route-map ONLY_LOCAL permit 10
 match ip address prefix-list LOCAL_NET
Применяем фильтр к внешнему соседу (Триада, IP: 52.10.5.1)
  neighbor 52.10.5.1 route-map ONLY_LOCAL out
```
Проверяем

```
R18#show ip bgp
BGP table version is 36, local router ID is 10.255.255.18
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  5.83.32.0/30     52.10.5.1                              0 520 301 i
 *>  10.10.25.0/30    52.10.5.1                              0 520 301 101 1001 i
 *>  10.30.11.0/30    52.10.5.1                              0 520 301 101 1001 i
 *>  10.80.8.0/28     52.10.5.1                              0 520 301 101 1001 i
 *>  10.100.71.0/28   52.10.5.1                              0 520 301 101 1001 i
 *>  10.132.35.0/30   52.10.5.1                              0 520 301 101 1001 i
 *>  10.182.0.0/30    52.10.5.1                              0 520 301 101 i
 *>  10.255.255.14/32 52.10.5.1                              0 520 301 101 1001 i
 *>  10.255.255.15/32 52.10.5.1                              0 520 301 101 1001 i
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.255.255.21/32 52.10.5.1                              0 520 301 i
 *>  10.255.255.22/32 52.10.5.1                              0 520 301 101 i
 *>  31.0.0.0/30      52.10.5.1                0             0 520 i
 *   52.10.5.0/30     52.10.5.1                0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.0.20.0/30    52.10.5.1                              0 520 301 101 i
 *>  188.0.0.0/30     52.10.5.1                              0 520 301 i
 *>  192.168.30.0/27  10.10.1.2          1536000         32768 i
 *>  192.168.40.0/27  10.10.2.2          1536000         32768 i
 *>  192.168.55.0/27  52.10.5.1                              0 520 301 101 1001 i
 *>  192.168.62.0/27  52.10.5.1                              0 520 301 101 1001 i

```
### 3. Фильтрация на провайдере Киторн (AS 101)
* Настроен Prefix-list, разрешающий только 0.0.0.0/0.
* Маршрут по умолчанию отдается в офис Москва.
```
Создаем Prefix-list, разрешающий только маршрут по умолчанию
ip prefix-list ONLY_DEF seq 5 permit 0.0.0.0/0
Создаем Route-map для исходящего трафика
route-map ONLY_DEF out
 match ip address prefix-list ONLY_DEF
Применяем фильтр к соседу Москвы (например, IP: 172.0.20.2)
 neighbor 172.0.20.1 route-map ONLY_DEF out
```
### 4. Фильтрация на провайдере Ламас (AS 301)
* Настроен Prefix-list, разрешающий 0.0.0.0/0, 192.168.30.0/27 и 192.168.40.0/27.
* Маршрут по умолчанию и сети СПб отдаются в офис Москва.
```
Создаем Prefix-list
ip prefix-list DEF_AND_SPB seq 5 permit 0.0.0.0/0
ip prefix-list DEF_AND_SPB seq 10 permit 192.168.30.0/27
ip prefix-list DEF_AND_SPB seq 15 permit 192.168.40.0/27
Создаем Route-map
route-map DEF_AND_SPB permit 10
 match ip address prefix-list DEF_AND_SPB
Применяем фильтр к соседу Москвы
 neighbor 188.0.0.2 route-map DEF_AND_SPB out
```
### 5. IP связность
* Проверена связность между VPC8 (СПб) и VPC7 (Москва).
* Достигнута полная связность. 

```
VPCS> ping 192.168.55.22

84 bytes from 192.168.55.22 icmp_seq=1 ttl=58 time=10.444 ms
84 bytes from 192.168.55.22 icmp_seq=2 ttl=58 time=2.903 ms
84 bytes from 192.168.55.22 icmp_seq=3 ttl=58 time=6.450 ms
84 bytes from 192.168.55.22 icmp_seq=4 ttl=58 time=4.557 ms
84 bytes from 192.168.55.22 icmp_seq=5 ttl=58 time=4.720 ms


VPCS> ping 192.168.30.15

84 bytes from 192.168.30.15 icmp_seq=1 ttl=57 time=10.683 ms
84 bytes from 192.168.30.15 icmp_seq=2 ttl=57 time=3.648 ms
84 bytes from 192.168.30.15 icmp_seq=3 ttl=57 time=4.292 ms
84 bytes from 192.168.30.15 icmp_seq=4 ttl=57 time=3.408 ms
84 bytes from 192.168.30.15 icmp_seq=5 ttl=57 time=4.093 ms
```