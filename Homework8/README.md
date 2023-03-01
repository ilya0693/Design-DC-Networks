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
  
ip prefix-list LOOPBACKS seq 5 permit 10.123.0.0/24 eq 32

route-map REDISTRIBUTE_CONNECTED permit 10
  match ip address prefix-list LOOPBACKS
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  ip address 10.123.1.1/31
  no shutdown

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

ip prefix-list LOOPBACKS seq 5 permit 10.123.0.0/24 eq 32

route-map REDISTRIBUTE_CONNECTED permit 10
  match ip address prefix-list LOOPBACKS
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  ip address 10.123.1.5/31
  no shutdown

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

Конфигурация коммутатора **_Border_Leaf-1_**
```sh
hostname Border_Leaf-1

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab
  
ip prefix-list LOOPBACKS seq 5 permit 10.123.0.0/24 eq 32

route-map REDISTRIBUTE_CONNECTED permit 10
  match ip address prefix-list LOOPBACKS 
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  ip address 10.123.1.9/31
  no shutdown

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
  
ip prefix-list LOOPBACKS seq 5 permit 10.123.0.0/24 eq 32
ip as-path access-list LEAFS_AS seq 10 permit "^42001000[1-9]{2}$"
  
route-map LEAFS_AS permit 10
  match as-path LEAFS_AS
route-map REDISTRIBUTE_CONNECTED permit 10
  match ip address prefix-list LOOPBACKS
  
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
### 1. Конфигурация VxLAN EVPN и межсетевого взаимодействия между разными Tenant.

Данная домашняя работа подразумевает создание VxLAN EVPN фабрики с использованием 2х Tenant (или MAC-VRF). Данные Tenant'ы изолированы друг от друга и для обеспечения связности между ними нужно дополнительное устройство, у которого будет выход в оба Tenant'а. В нашем случае таким устройством является Firewall. Рассмотрим дизайн сети более подробно. L1-L3 схема сети и схема Overlay представлена на рисунках ниже.

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework8/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-L1-L3%20Firewall.drawio.png "L1-L3 схема сети ЦОД")
L1-L3 схема сети ЦОД
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework8/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-Overlay%2BFirewall.drawio.png "Схема Overlay ЦОД")
Схема Overlay ЦОД

В рамках данной домашней работы нам необходимо:
* обеспечить L2 связность между Server-1 и Server-2. Данные сервера будут находиться в одном Tenant'е и широковещательном домене.
* обеспечить L3 связность между Server-3 и Server-4. Данные сервера будут находиться в одном Tenant'е, но в разных широковещательных доменах.
* обеспечить связность между Server-1/Server-2 и Server-3/Server-4 через Firewall. Каждый Tenant должен быть доступен через суммарный префикс.

Несколько примечаний по выполнению ДЗ:
* Для решения данных задач будет использоваться технология VxLAN/EVPN. В рамках данного ДЗ межсетевое взаимодействие между Server-1 и Server-3 или между Server-2 и Server-4 рассматриваться не будет, так как для их взаимодействия необходимо настраивать VRF-Leaking, что выходит за рамки данного ДЗ.
* В качестве серверов и Firewall'а используются образы Cisco IOL. 
* Фильтрация трафика в данном ДЗ рассматриваться и выполняться не будет.

Рассмотрим подробно конфигурацию VxLAN/EVPN и способ обеспечения связности 2х разных Tenant'ов.

##### 1.1. Пример конфигурации EVPN на коммутаторах Spine

В данной ДЗ подробое описание команд/секций рассматриваться не будет, так как это уже сделано ранее. В данном подразделе будет приведен лишь синтаксис конфигурации (смотрите ниже).

```sh
nv overlay evpn

route-map NEXT-HOP-UNCH permit 10
  set ip next-hop unchanged
  
router bgp 4200100000
  address-family l2vpn evpn
    nexthop route-map NEXT-HOP-UNCH
    retain route-target all
  neighbor 10.123.0.0/24 remote-as route-map LEAFS_AS
    update-source loopback0
    ebgp-multihop 3
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map NEXT-HOP-UNCH out
```

##### 1.2. Пример конфигурации VxLAN/EVPN на коммутаторе _"Leaf-1"_

В данном подразделе рассмотрим конфигурацию Leaf коммутаторов более подробно. Первым шагом включаем функционал EVPN, VxLAN, возможность ассоциировать VLAN с VNI и fabric forwarding. Функционал fabric forwarding нужен для работы L3VNI.
```sh
nv overlay evpn
feature vn-segment-vlan-based
feature nv overlay
feature fabric forwarding
```

Ассоциируем VLAN'ы с VNI. В данной домашней работе для VNI зарезервировано значение 8XXXX, где XXXX - номер VLAN'а. Также создадим 2 VLAN'а для L3VNI, тем самым поместив их в разные Tenant'ы.
```sh
vlan 100
  name Servers_v100->VRF_"A"
  vn-segment 80100
vlan 300
  name Servers_v300->VRF_"B"
  vn-segment 80300
vlan 900
  name L3VNI_for_VRF_"A"
  vn-segment 80900
vlan 910
  name L3VNI_for_VRF_"B"
  vn-segment 80910
```

Настроим BGP в адресном семействе l2vpn evpn, тем самым создав Overlay сеть.
```sh
router bgp 4200100011
  template peer Spines_Overlay
    remote-as 4200100000
    update-source loopback0
    ebgp-multihop 3
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.123.0.41
    inherit peer Spines_Overlay
    description Spine-1
```

Следующим шагом создаим два VRF, предназначенные для L3VNI, и настроим в созданном нами VRF RD и RT в адресном семействе ipv4.
```sh
vrf context A
  vni 80900
  rd 10.123.0.12:900
  address-family ipv4 unicast
    route-target import 80900:900
    route-target import 80900:900 evpn
    route-target export 80900:900
    route-target export 80900:900 evpn

vrf context B
  vni 80910
  rd 10.123.0.12:910
  address-family ipv4 unicast
    route-target import 80910:910
    route-target import 80910:910 evpn
    route-target export 80910:910
    route-target export 80910:910 evpn
```

Так как между Server-1 и Server-2 используется L2VNI, сконфигурируем секцию evpn. Укажем, что у нас используется L2 VNI, настроим RD и RT для нашего VNI. Так как у нас используется eBGP, рекомендуется настраивать RD и RT вручную, чтобы они были одинаковыми в разных AS.
```sh
evpn
  vni 80100 l2
    rd 10.123.0.12:100
    route-target import 80100:100
    route-target export 80100:100
```

Создадим SVI интерфейс, ассоциируем данный SVI с созданным нами VRF и включим функционал anycast-gateway. Также настроим общий мак адрес для anycast gateway.
```sh
fabric forwarding anycast-gateway-mac 000a.000b.000c

interface vlan 100
  vrf member A
  ip address 10.123.100.1/25
  fabric forwarding mode anycast-gateway
  no shutdown
  
interface vlan 300
  vrf member B
  ip address 10.123.253.1/24
  fabric forwarding mode anycast-gateway
  no shutdown
```

Создадим SVI интерфейс для L3VNI обоих Tenant'ов. На нем не настраивается IP адрес, только форвардинг трафика.
```sh
interface vlan 900
  vrf member A
  ip forward
  no shutdown
  
interface vlan 910
  vrf member B
  ip forward
  no shutdown
```

Сконфигурируем nve интерфейс для работы VxLAN (как L2VNI, так L3VNI).
```sh
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 80100
    ingress-replication protocol bgp
  member vni 80300
    ingress-replication protocol bgp
  member vni 80900 associate-vrf
  member vni 80910 associate-vrf
```

В завершении пробросим VLAN 100 и VLAN 300 до соответствующих серверов.
```sh
interface Ethernet1/6
  switchport
  description Server-1
  switchport mode trunk
  switchport trunk allowed vlan 100
  
interface Ethernet1/7
  switchport
  description Server-3
  switchport mode trunk
  switchport trunk allowed vlan 300
```

На остальных Leaf коммутаторах (в том числе и Border_Leaf) VxLAN EVPN настраивается идентичным образом, с небольшими отличиями. С конфигурацией коммутаторов можно ознакомиться ниже.

<details>

<summary> Конфигурация VxLAN EVPN на коммутаторе Leaf-2 </summary>

```sh
nv overlay evpn
feature vn-segment-vlan-based
feature nv overlay
feature fabric forwarding
 
vlan 100
  name Servers_v100->VRF_"A"
  vn-segment 80100
vlan 400
  name Servers_v400->VRF_"B"
  vn-segment 80400
vlan 900
  name L3VNI_for_VRF_"A"
  vn-segment 80900
vlan 910
  name L3VNI_for_VRF_"B"
  vn-segment 80910
  
router bgp 4200100022
  template peer Spines_Overlay
    remote-as 4200100000
    update-source loopback0
    ebgp-multihop 3
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.123.0.41
    inherit peer Spines_Overlay
    description Spine-1

vrf context A
  vni 80900
  rd 10.123.0.22:900
  address-family ipv4 unicast
    route-target import 80900:900
    route-target import 80900:900 evpn
    route-target export 80900:900
    route-target export 80900:900 evpn

vrf context B
  vni 80910
  rd 10.123.0.22:910
  address-family ipv4 unicast
    route-target import 80910:910
    route-target import 80910:910 evpn
    route-target export 80910:910
    route-target export 80910:910 evpn

evpn
  vni 80100 l2
    rd 10.123.0.22:100
    route-target import 80100:100
    route-target export 80100:100

fabric forwarding anycast-gateway-mac 000a.000b.000c

interface vlan 100
  vrf member A
  ip address 10.123.100.1/25
  fabric forwarding mode anycast-gateway
  no shutdown
  
interface vlan 400
  vrf member B
  ip address 10.123.254.1/24
  fabric forwarding mode anycast-gateway
  no shutdown
  
interface vlan 900
  vrf member A
  ip forward
  no shutdown
  
interface vlan 910
  vrf member B
  ip forward
  no shutdown
  
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 80100
    ingress-replication protocol bgp
  member vni 80400
    ingress-replication protocol bgp
  member vni 80900 associate-vrf
  member vni 80910 associate-vrf
  
interface Ethernet1/6
  switchport
  description Server-2
  switchport mode trunk
  switchport trunk allowed vlan 100
  
interface Ethernet1/7
  switchport
  description Server-4
  switchport mode trunk
  switchport trunk allowed vlan 400
```
</details>

<details>

<summary> Конфигурация VxLAN EVPN на коммутаторе Border_Leaf-1 </summary>

```sh
nv overlay evpn
feature vn-segment-vlan-based
feature nv overlay
feature fabric forwarding
 
vlan 900
  name L3VNI_for_VRF_"A"
  vn-segment 80900
vlan 910
  name L3VNI_for_VRF_"B"
  vn-segment 80910
  
router bgp 4200100033
  template peer Spines_Overlay
    remote-as 4200100000
    update-source loopback0
    ebgp-multihop 3
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.123.0.41
    inherit peer Spines_Overlay
    description Spine-1

vrf context A
  vni 80900
  rd 10.123.0.32:900
  address-family ipv4 unicast
    route-target import 80900:900
    route-target import 80900:900 evpn
    route-target export 80900:900
    route-target export 80900:900 evpn

vrf context B
  vni 80910
  rd 10.123.0.32:910
  address-family ipv4 unicast
    route-target import 80910:910
    route-target import 80910:910 evpn
    route-target export 80910:910
    route-target export 80910:910 evpn

interface vlan 900
  vrf member A
  ip forward
  no shutdown
  
interface vlan 910
  vrf member B
  ip forward
  no shutdown
  
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 80900 associate-vrf
  member vni 80910 associate-vrf
```
</details>

Следует отметить, что у нас настроен симметричный L3VNI, так как VLAN L3VNI распространяется на каждый Leaf коммутатор.

##### 1.3. Конфигурация межсетевого взаимодействия между разными Tenant'ами.

Для обеспечения связности между разными Tenant'ами необходимо донастроить наш Border_Leaf коммутатор и настроить Firewall. Для начала, рассмотрим настройки Border_Leaf.

На 1 шаге необходимо настроить стыковочные интерфейсы между BR и FW. В нашем случае будут использоваться стыковочные сабинтерфейсы. Каждый сабинтерфейс назначается в соответствующий VRF.
 ```sh
interface ethernet 1/7
  no shutdown
  no switchport

interface Ethernet1/7.3010
  encapsulation dot1q 3010
  vrf member A
  ip address 10.123.1.11/31
  no shutdown

interface Ethernet1/7.3011
  encapsulation dot1q 3011
  vrf member B
  ip address 10.123.1.13/31
  no shutdown
```
Далее необходимо настроить BGP  сессии в адресном семействе ipv4 unicast между BR и FW
```sh
router bgp 4200100033
  vrf A
    address-family ipv4 unicast
      aggregate-address 10.123.100.0/24 summary-only
    neighbor 10.123.1.10
      remote-as 4200100055
      local-as 4200100034 no-prepend replace-as
      address-family ipv4 unicast
   vrf B
    address-family ipv4 unicast
      aggregate-address 10.123.253.0/23 summary-only
    neighbor 10.123.1.12
      remote-as 4200100055
      local-as 4200100035 no-prepend replace-as
      address-family ipv4 unicast
```
На этом конфигурацию BR можно считать завершенной. Настроим Firewall.
```sh
interface Ethernet0/0
 no ip address

interface Ethernet0/0.3010
 encapsulation dot1Q 3010
 ip address 10.123.1.10 255.255.255.254

interface GigabitEthernet0/0.200
 encapsulation dot1Q 200
 ip address 169.254.0.8 255.255.255.254
 
router bgp 4200100055
 bgp log-neighbor-changes
 neighbor 10.123.1.11 remote-as 4200100034
 neighbor 10.123.1.13 remote-as 4200100035
```

На этом конфигурацию VxLAN/EVPN фабрики ЦОД можно считать завершенной.

##### 1.4. Проверка работоспособности VxLAN EVPN
После настройки L2 и L3 связности с использованием технологий VxLAN EVPN проверим, что у нас VxLAN туннель работает, EVPN маршруты передаются, разные Tenant'ы могут общаться друг с другом. Так как в данном ДЗ упор идет на маршрутизацию между Tenant'ами, проверим работоспособность со стороны коммутатора **_Border_Leaf-1_**

1. Проверка состояния NVE интерфейса (VxLAN туннеля).
 ```sh
Border_Leaf-1# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      10.123.0.12                             Up    CP        00:14:54 5001.0000.1b08
nve1      10.123.0.22                             Up    CP        00:14:54 5002.0000.1b08

```
Как мы видим, интерфейс NVE в состоянии UP.

2. Более подробная информация об nve интерфейсе.
 ```sh
Border_Leaf-1# show nve vni
Codes: CP - Control Plane        DP - Data Plane
       UC - Unconfigured         SA - Suppress ARP
       SU - Suppress Unknown Unicast
       Xconn - Crossconnect
       MS-IR - Multisite Ingress Replication

Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      80900    n/a               Up    CP   L3 [A]
nve1      80910    n/a               Up    CP   L3 [B]

```
Из вывода команды видим, что на BR используется только L3 VNI и каждый VNI находится в своем VRF.

3. Проверка общей (суммарной) информации об установлении BGP соседства в адресном семействе L2VPN EVPN.
 ```sh
Border_Leaf-1# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.123.0.31, local AS number 4200100033
BGP table version is 67, L2VPN EVPN config peers 1, capable peers 1
13 network entries and 13 paths using 2692 bytes of memory
BGP attribute entries [13/2236], BGP AS path entries [5/58]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.123.0.41     4 4200100000
                            195     165       67    0    0 00:16:46 4
```

Как мы видим, BGP соседство установлено и они обмениваются EVPN маршрутами.

4. Просмотр таблицы BGP адресного семейства L2VPN EVPN.
 ```sh
Border_Leaf-1# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 67, Local Router ID is 10.123.0.31
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.123.0.11:33067
*>e[2]:[0]:[0]:[48]:[aabb.cc00.6000]:[32]:[10.123.253.10]/272
                      10.123.0.12                                    0 420010000
0 4200100011 i

Route Distinguisher: 10.123.0.12:100
*>e[2]:[0]:[0]:[48]:[aabb.cc00.5000]:[32]:[10.123.100.10]/272
                      10.123.0.12                                    0 420010000
0 4200100011 i

Route Distinguisher: 10.123.0.21:33167
*>e[2]:[0]:[0]:[48]:[aabb.cc00.8000]:[32]:[10.123.254.10]/272
                      10.123.0.22                                    0 420010000
0 4200100022 i

Route Distinguisher: 10.123.0.22:100
*>e[2]:[0]:[0]:[48]:[aabb.cc00.7000]:[32]:[10.123.100.11]/272
                      10.123.0.22                                    0 420010000
0 4200100022 i

Route Distinguisher: 10.123.0.32:900    (L3VNI 80900)
*>e[2]:[0]:[0]:[48]:[aabb.cc00.5000]:[32]:[10.123.100.10]/272
                      10.123.0.12                                    0 420010000
0 4200100011 i
*>e[2]:[0]:[0]:[48]:[aabb.cc00.7000]:[32]:[10.123.100.11]/272
                      10.123.0.22                                    0 420010000
0 4200100022 i
*>l[5]:[0]:[0]:[23]:[10.123.252.0]/224
                      10.123.0.32                                    0 420010005
5 4200100035 i
*>l[5]:[0]:[0]:[24]:[10.123.100.0]/224
                      10.123.0.32                       100      32768 i
*>l[5]:[0]:[0]:[32]:[10.123.254.10]/224
                      10.123.0.32                                    0 420010005
5 4200100035 4200100000 4200100022 i

Route Distinguisher: 10.123.0.32:910    (L3VNI 80910)
*>e[2]:[0]:[0]:[48]:[aabb.cc00.6000]:[32]:[10.123.253.10]/272
                      10.123.0.12                                    0 420010000
0 4200100011 i
*>e[2]:[0]:[0]:[48]:[aabb.cc00.8000]:[32]:[10.123.254.10]/272
                      10.123.0.22                                    0 420010000
0 4200100022 i
*>l[5]:[0]:[0]:[23]:[10.123.252.0]/224
                      10.123.0.32                       100      32768 i
*>l[5]:[0]:[0]:[24]:[10.123.100.0]/224
                      10.123.0.32                                    0 420010005
5 4200100034 i

```
Из вывода команды видно, чте в таблице BGP есть EVPN маршруты до удаленных серверов (маршруты 2 типа), маршруты до удаленных VTEP (маршруты 3 типа) и маршруты до сетей, находящихся в других Tenant (маршруты 5 типа). 

5. Просмотр L2 маршрутов EVPN.
 ```sh
Border_Leaf-1# show l2route evpn mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Pf):Permanently-Frozen, (Orp): Orphan

Topology    Mac Address    Prod   Flags         Seq No     Next-Hops

----------- -------------- ------ ------------- ---------- ---------------------
------------------
900         5001.0000.1b08 VXLAN  Rmac          0          10.123.0.12

900         5002.0000.1b08 VXLAN  Rmac          0          10.123.0.22

910         5001.0000.1b08 VXLAN  Rmac          0          10.123.0.12

910         5002.0000.1b08 VXLAN  Rmac          0          10.123.0.22


```
Из вывода команды видно, что мак адреса получены от удаленных VTEP.

### 2. Проверка доступности сетевых узлов.
После настройки VxLAN EVPN проверим, что сервера "видят" друг друга. Для проверки доступности сетевых узлов будем использовать протокол ICMP, а именно команду ping, запускаемую на серверах ЦОД. Ниже представлен пример доступности всех серверов. Проверка проводилась с сервера 3 (Server-3).

```sh
Server-3#ping 10.123.100.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.123.100.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 30/64/168 ms

Server-3#ping 10.123.100.11
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.123.100.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 32/51/96 ms

Server-3#ping 10.123.254.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.123.254.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 13/31/95 ms

```

С полной версией конфигурации каждого сетевого оборудования можно ознакомиться [здесь](https://github.com/ilya0693/Design-DC-Networks/tree/main/Homework8/Configs).
