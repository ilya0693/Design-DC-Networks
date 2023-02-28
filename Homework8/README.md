# VxLAN. Оптимизация таблиц маршрутизации

## Цель
Реализовать передачу суммарных префиксов через EVPN route-type 5

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Анонсируете суммарные префиксы клиентов в Overlay сеть;
2. Настроите маршрутизацию между клиентами через суммарный префикс;
3. План работы, адресное пространство, схема сети, настройки - зафиксированы в документации.

## Общая информация

<details>

<summary> Схема стенда </summary>
  
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework8/EVE_NG%20Scheme.png "Схема стенда EVE-NG")

</details>

<details>

<summary> Конфигурация серверов </summary>

Конфигурация **_"Server-1"_**
```sh
hostname Server-1
 !
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.100
 encapsulation dot1Q 100
 ip address 10.123.100.10 255.255.255.128
!
ip route 0.0.0.0 0.0.0.0 192.168.100.1
```

Конфигурация **_"Server-2"_**
```sh
hostname Server-2
 !
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.100
 encapsulation dot1Q 100
 ip address 10.123.100.11 255.255.255.128
!
ip route 0.0.0.0 0.0.0.0 192.168.100.1
```
  
Конфигурация **_"Server-3"_**

```sh
hostname Server-3
 !
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.300
 encapsulation dot1Q 300
 ip address 10.123.253.10 255.255.255.0
!
ip route 0.0.0.0 0.0.0.0 10.123.253.1
```

Конфигурация **_"Server-4"_**

```sh
hostname Server-4
 !
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.400
 encapsulation dot1Q 400
 ip address 10.123.254.10 255.255.255.0
!
ip route 0.0.0.0 0.0.0.0 10.123.254.1
```
</details>

<details> 
<summary> Начальная конфигурация коммутаторов NX-OS (включая Underlay маршрутизацию) </summary>

Конфигурация коммутатора **_Leaf-1_**
```sh
hostname Leaf-1

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab
  
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback0 loopback1
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  ip address 10.123.1.1/31
  no shutdown
  
interface ethernet 1/7
  description Server-1
  switchport
  no shutdown
  switchport mode access
  switchport access vlan 100

interface loopback0
  description RID
  ip address 10.123.0.11/32

interface loopback1
  description VTEP
  ip address 10.123.0.12/32
  
boot nxos bootflash:nxos.9.3.10.bin

cli alias name wr copy running-config startup-config
  
router bgp 4200100011
  router-id 10.123.0.11
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 64
  template peer Spines_Underlay
    remote-as 4200100000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.0
    inherit peer Spines_Underlay
    description Spine-1
```

Конфигурация коммутатора **_Leaf-2_**
```sh
hostname Leaf-2

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers

route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback0 loopback1
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  ip address 10.123.1.5/31
  no shutdown
  
interface ethernet 1/7
  description Server-2
  no shutdown
  switchport
  switchport mode access
  switchport access vlan 100

interface loopback0
  description RID
  ip address 10.123.0.21/32

interface loopback1
  description VTEP
  ip address 10.123.0.22/32
  
boot nxos bootflash:nxos.9.3.10.bin

cli alias name wr copy running-config startup-config
  
router bgp 4200100022
  router-id 10.123.0.21
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 64
  template peer Spines_Underlay
    remote-as 4200100000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.4
    inherit peer Spines_Underlay
    description Spine-1
```

Конфигурация коммутатора **_Leaf-3_**
```sh
hostname Leaf-3

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 200
  name Servers
vlan 300
  name Servers-v300
  
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback0 loopback1  
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  ip address 10.123.1.9/31
  no shutdown
  
interface ethernet 1/6
  description Server-3
  switchport
  no shutdown
  switchport mode access
  switchport access vlan 200
  
interface ethernet 1/7
  description Server-4
  switchport
  no shutdown
  switchport mode access
  switchport access vlan 300

interface loopback0
  description RID
  ip address 10.123.0.31/32

interface loopback1
  description VTEP
  ip address 10.123.0.32/32
  
boot nxos bootflash:nxos.9.3.10.bin

cli alias name wr copy running-config startup-config
  
router bgp 4200100033
  router-id 10.123.0.31
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 64
  template peer Spines_Underlay
    remote-as 4200100000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.8
    inherit peer Spines_Underlay
    description Spine-1
```

Конфигурация коммутатора **_Spine-1_**  
```sh
hostname Spine-1
  
feature bgp

no ip domain-lookup
ip domain-name dc.lab
  
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback0
route-map LEAFS_AS permit 10
  match as-path LEAFS_AS
  
interface Ethernet1/1
  description to_Leaf-1
  no switchport
  ip address 10.123.1.0/31
  no shutdown

interface Ethernet1/2
  description to_Leaf-2
  no switchport
  ip address 10.123.1.4/31
  no shutdown

interface Ethernet1/3
  description to_Leaf-3
  no switchport
  ip address 10.123.1.8/31
  no shutdown

interface loopback0
  description RID
  ip address 10.123.0.41/32

boot nxos bootflash:nxos.9.3.10.bin
  
cli alias name wr copy running-config startup-config
  
ip as-path access-list LEAFS_AS seq 10 permit "^42001000[1-9]{2}$"
  
router bgp 4200100000
  router-id 10.123.0.41
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 64
  neighbor 10.123.1.0/24 remote-as route-map LEAFS_AS
    timers 3 9
    address-family ipv4 unicast
```

</details>

<details> 

<summary>Полезные команды </summary>

```
show nve peers
show nve vni
show vxlan interface
show bgp l2vpn evpn summary
show bgp l2vpn evpn
show l2route evpn mac all
show l2route evpn mac-ip all
```

</details>

<details> 

<summary> Используемые технологии/протоколы </summary>

**_EVPN (Ethernet over VPN)_** - технология, обеспечивающая передачу L2 трафика через WAN каналы связи (не имеющие прямой связности L2). Данная технология реализуется с использованием взаимодействия следующих протоколов:
–	BGP - информация о месторасположении узла-оконечного устройства с точки зрения EVPN;
–	VXLAN - передача L2 сегментов в инкапсулированном виде через IP/UDP;

**_BGP_** — Протокол динамической маршрутизации. Протокол BGP имеет уникальную (относительно иных протоколов маршрутизации) возможность передавать расширенную информацию в данных Multiprotocol BGP (MP-BGP). В сети ЦОДа данная возможность используется для обеспечения работы технологии EVPN: через протокол BGP между коммутаторами происходит обмен следующими данными по местонахождению узлов-клиентов EVPN:
  *	IP адреса устройств в EVPN;
  *	MAC адреса устройств в EVPN;
  *	За каким устройством (с точки зрения топологии BGP) находятся узлы-клиенты сети EVPN, на какое устройство следует отправлять инкапсулированные VXLAN пакеты для целевого узла.

**_VxLAN_** — технология инкапсуляции L2 фреймов в IP пакеты (UDP протокол). Технология стандартизирована в RFC7348. VXLAN не обеспечивает шифрования данных. Каждый VXLAN идентифицируется уникальным VNID (Virtual Network ID). VXLAN предоставляет следующие возможности:
  *	24 бит VNID (более 16 млн уникальных ID);
  *	Передача L2 трафика через L3 сети (MAC in UDP);
  *	Совместно с EVPN, BGP обеспечивается целевая доставка L2 фреймов получателю: минимизируется количество широковещательных фреймов через WAN каналы связи;
  *	Если L3 Underlay сеть обеспечивает балансировку между несколькими WAN каналами связи, то VXLAN (аналогично любому другому UDP трафику) может использовать возможности балансировки трафика для обеспечения равномерной нагрузки каналов связи;
  *	Использует один порт UDP/4789 для входящих пакетов (для более равномерной балансировки инкапсулированного трафика транспортная сеть [Underlay] должна обеспечивать балансировку по UDP порту источника, назначения).

</details>

## Выполнение домашнего задания
