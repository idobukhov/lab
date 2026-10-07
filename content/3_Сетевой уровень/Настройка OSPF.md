---
tags:
  - ospf
  - point-to-point
  - stub
  - lab
---

# Лабораторная работа: Настройка OSPF

## Краткое описание

**Цель работы:** Изучение и практическая настройка протокола OSPF, включая настройку пассивных интерфейсов, типа сети point-to-point, тупиковых областей (stub) и маршрута по умолчанию.

### Задачи:
- [ ] Включить OSPF на интерфейсах маршрутизаторов R1, R2, R3, INET-1 с использованием метода интерфейса.
- [ ] Включить пассивные интерфейсы в сегментах LAN и интерфейсах обратной связи.
- [ ] Включить OSPF на интерфейсах маршрутизаторов Remote-1, Remote-2 с использованием глобального метода.
- [ ] Настроить тип сети OSPF «точка-точка» на всех интерфейсах WAN, назначенных области 0.
- [ ] Настроить маршрутизаторы Remote-1 и Remote-2 как тупиковые области OSPF.
- [ ] Объявить все сегменты LAN, подключенные подсети WAN и интерфейсы обратной связи.
- [ ] Объявить маршрут по умолчанию от INET-1 всем соседним маршрутизаторам.
- [ ] Проверить конфигурацию лабораторной работы и таблицы маршрутизации OSPF.

### Топология сети

![[topology_ospf_base.png]]

---

## Шаг 1: Настройка OSPF методом интерфейса (R1, R2, R3, INET-1)
Включите протокол OSPF на интерфейсах, подключенных к маршрутизатору, используя метод настройки интерфейса. Настройте пассивные интерфейсы в сегментах локальной сети и интерфейсах обратной связи.
### R1
```cisco
interface Loopback0
 description Management Interface
 ip address 192.168.255.1 255.255.255.255
 ip ospf 1 area 0
interface GigabitEthernet0/0/0
 description link to R2 router
 ip address 192.168.1.2 255.255.255.252
 ip ospf 1 area 1
 no shut
interface GigabitEthernet0/0/1
 description link to INET-1 router
 ip address 192.168.1.9 255.255.255.252
 ip ospf 1 area 0
 no shut
interface GigabitEthernet0/0/2
 description link to R3 router
 ip address 192.168.1.5 255.255.255.252
 ip ospf 1 area 0
 no shut
router ospf 1
 passive-interface Loopback0
```
### R2
```cisco
interface Loopback0
 description Management Interface
 ip address 192.168.255.2 255.255.255.255
 ip ospf 1 area 1
interface GigabitEthernet0/0/0
 description link to R1 router
 ip address 192.168.1.1 255.255.255.252
 ip ospf 1 area 1
 no shut
interface GigabitEthernet0/0/1
 description LAN (172.16.1.0/24)
 ip address 172.16.1.254 255.255.255.0
 ip ospf 1 area 1
 no shut
router ospf 1
 passive-interface default
 no passive-interface GigabitEthernet0/0/0
```
### R3
```cisco
interface Loopback0
 description Management Interface
 ip address 192.168.255.3 255.255.255.255
 ip ospf 1 area 0
interface GigabitEthernet0/0/0
 description link to R1 router
 ip address 192.168.1.6 255.255.255.252
 ip ospf 1 area 0
 no shut
interface GigabitEthernet0/0/1
 description link to INET-1 router
 ip address 192.168.1.13 255.255.255.252
 ip ospf 1 area 0
 no shut
interface GigabitEthernet0/0/2
 description link to Remote-1
 ip address 192.168.1.17 255.255.255.252
 ip ospf 1 area 2
 no shut
router ospf 1
 passive-interface Loopback0
```
### INET-1
```cisco
interface Loopback0
 description Management Interface
 ip address 192.168.255.4 255.255.255.255
 ip ospf 1 area 0
interface GigabitEthernet0/0
 description link to R1 router
 ip address 192.168.1.10 255.255.255.252
 ip ospf 1 area 0
 no shut
interface GigabitEthernet1/0
 description link to R3 router
 ip address 192.168.1.14 255.255.255.252
 ip ospf 1 area 0
 no shut
interface GigabitEthernet2/0
 description link to ISP router
 ip address 172.33.1.1 255.255.255.252
 no shut
interface GigabitEthernet3/0
 description link to Remote-2 router
 ip address 192.168.1.21 255.255.255.252
 ip ospf 1 area 3
 no shut
router ospf 1
 passive-interface GigabitEthernet2/0
 passive-interface Loopback0
```
---
## Шаг 2: Настройка OSPF глобальным методом (Remote-1, Remote-2)
Включите протокол OSPF на интерфейсах маршрутизаторов Remote-1 и Remote-2, используя метод глобальной конфигурации. Настройте пассивные интерфейсы в сегментах LAN и интерфейсах обратной связи.
### Remote-1
```cisco
interface Loopback0
 description Management Interface
 ip address 192.168.255.5 255.255.255.255
interface GigabitEthernet1/0/1
 description link to R3 router
 no switchport
 ip address 192.168.1.18 255.255.255.252
 no shut
router ospf 1
 passive-interface Loopback0
 passive-interface Vlan10
 passive-interface Vlan11
 passive-interface Vlan12
 network 172.16.0.0 0.0.255.255 area 2
 network 192.168.1.16 0.0.0.3 area 2
 network 192.168.255.5 0.0.0.0 area 2
```
### Remote-2
```cisco
interface Loopback0
 description Management Interface
 ip address 192.168.255.6 255.255.255.255
interface GigabitEthernet0/0/0
 description link to INET-1 router
 ip address 192.168.1.22 255.255.255.252
 no shut
interface GigabitEthernet0/0/1
 description LAN (172.16.2.0/26)
 ip address 172.16.2.62 255.255.255.192
 no shut
router ospf 1
 passive-interface default
 no passive-interface GigabitEthernet0/0/0
 network 172.16.2.0 0.0.0.63 area 3
 network 192.168.1.20 0.0.0.3 area 3
 network 192.168.255.6 0.0.0.0 area 3
```
---
## Шаг 3: Настройка типа сети «точка-точка» (Point-to-Point)
Настройте сетевой тип OSPF «точка-точка» на всех интерфейсах WAN, назначенных области 0. Это предотвратит выбор DR/BDR и ускорит сходимость для каналов WAN.
### R1
```cisco
interface GigabitEthernet0/0/1
 ip ospf network point-to-point
interface GigabitEthernet0/0/2
 ip ospf network point-to-point
```
### R3
```cisco
interface GigabitEthernet0/0/0
 ip ospf network point-to-point
interface GigabitEthernet0/0/1
 ip ospf network point-to-point
```
### INET-1
```cisco
interface GigabitEthernet0/0
 ip ospf network point-to-point
interface GigabitEthernet1/0
 ip ospf network point-to-point
```
---
## Шаг 4: Настройка тупиковых областей (Stub)
Настройте маршрутизаторы Remote-1 и Remote-2 как тупиковые области OSPF, чтобы уменьшить размер базы данных состояния каналов (LSDB) для маршрутизаторов.
> [!info] Важно
> Настройку stub-области необходимо выполнять на всех маршрутизаторах, подключенных к данной области.
### Remote-1 (Area 2)
```cisco
router ospf 1
 area 2 stub
```
### R3 (Area 2)
```cisco
router ospf 1
 area 2 stub
```
### Remote-2 (Area 3)
```cisco
router ospf 1
 area 3 stub
```
### INET-1 (Area 3)
```cisco
router ospf 1
 area 3 stub
```
---
## Шаг 5: Маршрут по умолчанию
Объявите маршрут по умолчанию от INET-1 всем соседним устройствам, находящимся ниже по потоку, для обеспечения доступа в интернет.
### INET-1
```cisco
ip route 0.0.0.0 0.0.0.0 172.33.1.2
router ospf 1
 default-information originate
```
---
## Шаг 6: Проверка конфигурации
Проверьте смежность соседних узлов, таблицы маршрутизации, тип сети, тупиковые области и связность.
### Команды для проверки
- [ ] `show ip ospf neighbor` (R1, INET-1, R3)
- [ ] `show ip route` (R1, INET-1, R3)
- [ ] `show ip protocols` (R1)
- [ ] `show ip ospf interface brief` (R1, INET-1, Remote-1)
- [ ] `show ip ospf interface` (R1, INET-1, Remote-1)
### Проверка связности (Ping)
- [ ] `ping 172.16.2.1` (LAN-2)
- [ ] `ping 172.33.1.2` (ISP)
- [ ] `ping 172.16.10.254` (Remote-1)
- [ ] `ping 172.16.11.254` (Remote-1)
- [ ] `ping 172.16.12.254` (Remote-1)