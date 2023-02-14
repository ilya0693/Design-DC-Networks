# VxLAN. EVPN L3

## Цель
Настроить маршрутизацию в рамках Overlay между клиентами

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите каждого клиента в своем VNI;
2. Настроите маршрутизацию между клиентами;
2. План работы, адресное пространство, схема сети, настройки - зафиксированы в документации.

## Общая информация

<details>

<summary> Схема стенда </summary>

В данном задании стенд собран, согласно схеме, спроектированной в 1ом домашнем задании. Ниже предаставлена реализация спроектируемой сети в эмуляторе EVE-NG. С L2 и L3 схемой сети можно ознакомиться по [ссылке](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework1/README.md#%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F-%D1%81%D0%B5%D1%82%D0%B8-%D1%86%D0%BE%D0%B4-%D0%B8-%D0%B5%D0%B5-%D0%BE%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5).

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework4/Stand.png "Схема стенда")

</details>

<details>

<summary> Базовая конфигурация VPCs </summary>

Конфигурация VPCS **_"Server-1"_**
```sh
set pcname Server-1
ip 10.123.100.10 255.255.255.0 10.123.100.1
save
```

Конфигурация VPCS **_"Server-2"_**
```sh
set pcname Server-2
ip 10.123.100.11 255.255.255.0 10.123.100.1
save
```
  
Конфигурация VPCS **_"Server-3"_**

```sh
set pcname Server-3
ip 10.123.200.10 255.255.255.0 10.123.200.1
save
```

Конфигурация VPCS **_"Server-4"_**

```sh
set pcname Server-4
ip 10.123.100.10 255.255.255.0 10.123.300.1
save
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

vlan 100
  name Servers
  
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback1
  
interface Vlan100
  description GW_for_Servers->VLAN100
  no shutdown
  no ip redirects
  ip address 10.123.100.1/24
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  no ip redirects
  ip address 10.123.1.1/31
  no shutdown

interface Ethernet1/2
  description to_Spine-2
  no switchport
  no ip redirects
  ip address 10.123.1.3/31
  no shutdown
  
  interface ethernet 1/7
  description Server-1
  switchport
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
  template peer Spines
    remote-as 4200100000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.0
    inherit peer Spines
    description Spine-1
  neighbor 10.123.1.2
    inherit peer Spines
    description Spine-2
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
  match interface loopback1
  
interface Vlan100
  description GW_for_Servers->VLAN100
  no shutdown
  no ip redirects
  ip address 10.123.100.1/24
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  no ip redirects
  ip address 10.123.1.5/31
  no shutdown

interface Ethernet1/2
  description to_Spine-2
  no switchport
  no ip redirects
  ip address 10.123.1.7/31
  no shutdown
  
  interface ethernet 1/7
  description Server-2
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
  template peer Spines
    remote-as 4200100000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.4
    inherit peer Spines
    description Spine-1
  neighbor 10.123.1.6
    inherit peer Spines
    description Spine-2
```

Конфигурация коммутатора **_Leaf-3_**
```sh
hostname Leaf-3

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers
  
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback1  

interface Vlan100
  description GW_for_Servers->VLAN100
  no shutdown
  no ip redirects
  ip address 10.123.100.1/24
  
interface Ethernet1/1
  description to_Spine-1
  no switchport
  no ip redirects
  ip address 10.123.1.9/31
  no shutdown

interface Ethernet1/2
  description to_Spine-2
  no switchport
  no ip redirects
  ip address 10.123.1.11/31
  no shutdown
  
interface ethernet 1/6
  description Server-3
  switchport
  switchport mode access
  switchport access vlan 100
  
interface ethernet 1/7
  description Server-4
  switchport
  switchport mode access
  switchport access vlan 100

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
  template peer Spines
    remote-as 4200100000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.8
    inherit peer Spines
    description Spine-1
  neighbor 10.123.1.10
    inherit peer Spines
    description Spine-2
```

Конфигурация коммутатора **_Spine-1_**  
```sh
hostname Spine-1
  
feature bgp

no ip domain-lookup
ip domain-name dc.lab
  
interface Ethernet1/1
  description to_Leaf-1
  no switchport
  no ip redirects
  ip address 10.123.1.0/31
  no shutdown

interface Ethernet1/2
  description to_Leaf-2
  no switchport
  no ip redirects
  ip address 10.123.1.4/31
  no shutdown

interface Ethernet1/3
  description to_Leaf-3
  no switchport
  no ip redirects
  ip address 10.123.1.8/31
  no shutdown

interface loopback0
  description RID
  ip address 10.123.0.41/32

boot nxos bootflash:nxos.9.3.10.bin
  
cli alias name wr copy running-config startup-config
  
ip as-path access-list LEAFS_AS seq 10 permit "^42001000[1-9]{2}$"
  
route-map LEAFS_AS permit 10
  match as-path LEAFS_AS
  
router bgp 4200100000
  router-id 10.123.0.41
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    maximum-paths 64
  neighbor 10.123.1.0/24 remote-as route-map LEAFS_AS
    address-family ipv4 unicast
```

Конфигурация коммутатора **_Spine-2_**
```sh
hostname Spine-2
  
feature bgp

no ip domain-lookup
ip domain-name dc.lab
  
interface Ethernet1/1
  description to_Leaf-1
  no switchport
  no ip redirects
  ip address 10.123.1.2/31
  no shutdown

interface Ethernet1/2
  description to_Leaf-2
  no switchport
  no ip redirects
  ip address 10.123.1.6/31
  no shutdown

interface Ethernet1/3
  description to_Leaf-3
  no switchport
  no ip redirects
  ip address 10.123.1.10/31
  no shutdown

interface loopback0
  description RID
  ip address 10.123.0.51/32

boot nxos bootflash:nxos.9.3.10.bin  
  
cli alias name wr copy running-config startup-config
  
ip as-path access-list LEAFS_AS seq 10 permit "^42001000[1-9]{2}$"
  
route-map LEAFS_AS permit 10
  match as-path LEAFS_AS
  
router bgp 4200100000
  router-id 10.123.0.51
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    maximum-paths 64
  neighbor 10.123.1.0/24 remote-as route-map LEAFS_AS
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

### 1. Конфигурация VxLAN EVPN (L3 VNI).

Данная домашняя работа является прямым продолжением домашней работы [Homework5](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework4/README.md), но с некоторыми изменениями. В частности,  С начальной конфигурацией, в том числе и с конфигурацией Underlay сети, можно ознакомиться выше. 

Организовав связность Loopback интерфейсов между Leaf коммутаторами, далее необходимо построить Overlay сеть с использованием технологий VxLAN/EVPN. Схема Overlay сети представлена на рисунке ниже.
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework5/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-Overlay.drawio.png "Схема Overlay ЦОД")

Рассмотрим подробно конфигурацию VxLAN/EVPN одного из Leaf и Spine коммутатора ЦОД.

##### 1.1. Пример конфигурации EVPN на коммутаторе _"Spine-1"_

Включаем функционал EVPN. На Spine коммутаторах функционал VxLAN не включаем, так как Spine коммутаторы занимаются только распространением маршрутной информации (как IPv4, так и EVPN).
```sh
nv overlay evpn
```

Создаем route-map для анонсов connected маршрутов в адресном семействе ipv4. В нашем случае анонсируем только сеть интерфейса Loopback 0.
```sh
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback0
```

Создаем route-map, запрещающий менять next-hop при передаче маршрутной информации. Данный route-map необходим только для eBGP.
```sh
route-map NEXT-HOP-UNCH permit 10
  set ip next-hop unchanged
```

Так как EVPN соседство будет устанавливаться с использованием Loopback адресов, необходимо на Spine коммутаторах донастроить маршрутизацию eBGP в адресном семействе ipv4. Ранее была настроена маршрутизация только Loopback интерфейсов Leaf.
```sh
router bgp 4200100000
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
```

Настроим BGP в адресном семействе l2vpn evpn, тем самым создав Overlay сеть.
```sh
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

Несколько моментов, на которые стоит обратить внимание при конфигурации eBGP EVPN:
1. Команда _retain route-target all_ позволяет сохранить метки VNI на маршрутах EVPN. Без этой команды (так как на Spine не используется функционал VxLAN) Spine будет перезаписывать метки маршрутов, из-за чего может не построиться VxLAN туннель.
2. Для построения VxLAN туннеля необходимо сохранить достижимость next-hop. eBGP по умолчанию меняет next-hop, в связи с этим необходимо применить route-map политику в сторону Leaf коммутаторов, по направлению out.
3. Команда _ebgp-multihop 3_ необходима для построения eBGP соседства на Loopback адресах. По умолчанию TTL eBGP равен 1. 
4. Команда _send-community both_ нужна для передачи расширенных меток (route-target).

Коммутатор Spine-2 настраивается идентичным образом. С конфигурацией коммутатора Spine-2 можно ознакомиться ниже

<details>

<summary> Конфигурация EVPN на коммутаторе Spine-2 </summary>

```sh
nv overlay evpn
  
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback0
  
route-map NEXT-HOP-UNCH permit 10
  set ip next-hop unchanged
  
router bgp 4200100000
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
  
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
</details>

##### 1.2. Пример конфигурации VxLAN/EVPN на коммутаторе _"Leaf-1"_

Включаем функционал EVPN, VxLAN и возможность ассоциировать VLAN с VNI.
```sh
nv overlay evpn
feature vn-segment-vlan-based
feature nv overlay
```

Ассоциируем VLAN'ы с VNI. В данной домашней работе для VNI зарезервировано значение 8XXXX, где XXXX - номер VLAN'а.
```sh
vlan 100
  vn-segment 80100
```

Настроим BGP в адресном семействе l2vpn evpn, тем самым создав Overlay сеть.
```sh
router bgp 4200100011
  template peer Spines_Overlay
    remote-as 4200100000
    update-source loopback1
    ebgp-multihop 3
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.123.0.41
    inherit peer Spines_Overlay
    description Spine-1
  neighbor 10.123.0.51
    inherit peer Spines_Overlay
    description Spine-2
```

Сконфигурируем nve интерфейс для работы VxLAN.
```sh
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 80100
    ingress-replication protocol bgp
```

Сконфигурируем evpn, укажем, что у нас используется L2 VNI, настроим RD и RT для нашего VNI. Так как у нас используется eBGP, рекомендуется настраивать RD и RT вручную, чтобы они были одинаковыми в разных AS.
```sh
evpn
  vni 80100 l2
    rd 10.123.0.12:100
    route-target import 80100:100
    route-target export 80100:100
```

На остальных коммутаторах Leaf VxLAN EVPN настраивается идентичным образом. С конфигурацией коммутаторов можно ознакомиться ниже.

<details>

<summary> Конфигурация VxLAN EVPN на коммутаторе Leaf-2 </summary>

```sh
nv overlay evpn
feature vn-segment-vlan-based
feature nv overlay
 
vlan 100
  vn-segment 80100
  
router bgp 4200100022
  template peer Spines_Overlay
    remote-as 4200100000
    update-source loopback1
    ebgp-multihop 3
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.123.0.41
    inherit peer Spines_Overlay
    description Spine-1
  neighbor 10.123.0.51
    inherit peer Spines_Overlay
    description Spine-2
  
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 80100
    ingress-replication protocol bgp

evpn
  vni 80100 l2
    rd 10.123.0.22:100
    route-target import 80100:100
    route-target export 80100:100
```
</details>

<details>

<summary> Конфигурация VxLAN EVPN на коммутаторе Leaf-3 </summary>

```sh
nv overlay evpn
feature vn-segment-vlan-based
feature nv overlay
 
vlan 100
  vn-segment 80100
  
router bgp 4200100022
  template peer Spines_Overlay
    remote-as 4200100000
    update-source loopback1
    ebgp-multihop 3
    address-family l2vpn evpn
      send-community
      send-community extended
  neighbor 10.123.0.41
    inherit peer Spines_Overlay
    description Spine-1
  neighbor 10.123.0.51
    inherit peer Spines_Overlay
    description Spine-2
  
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 80100
    ingress-replication protocol bgp

evpn
  vni 80100 l2
    rd 10.123.0.32:100
    route-target import 80100:100
    route-target export 80100:100
```
</details>

##### 1.3. Проверка работоспособности VxLAN EVPN
После настройки L2 связности с использованием технологий VxLAN EVPN проверим, что у нас VxLAN туннель работает, а EVPN маршруты передаются. В качестве примера, проверим работоспособность маршрутизации VxLAN EVPN со стороны коммутатора **_Leaf-2_**

1. Проверка состояния NVE интерфейса (VxLAN туннеля).
 ```sh
Leaf-2# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.123.0.12                             Up    CP        00:27:38 n/a

nve1      10.123.0.32                             Up    CP        00:27:19 n/a

```
Как мы видим, интерфейс NVE в состоянии UP.

2. Более подробная информация об nve интерфейсе.
 ```sh
Leaf-2# show nve vni
Codes: CP - Control Plane        DP - Data Plane
       UC - Unconfigured         SA - Suppress ARP
       SU - Suppress Unknown Unicast
       Xconn - Crossconnect
       MS-IR - Multisite Ingress Replication

Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      80100    UnicastBGP        Up    CP   L2 [100]
```
Из вывода команды видим, что информацию об удаленных мак адресах Leaf-2 получает через Control Plane (через BGP) с использованием ingress репликации. Также из вывода команды видно, что nve интерфейс в состоянии UP и информацию о VNI.

3. Просмотр статуса VxLAN интерфейса
 ```sh
Leaf-2# show vxlan interface
Interface       Vlan    VPL Ifindex     LTL             HW VP
=========       ====    ===========     ===             =====
Eth1/7          100     0x530647fa      0x1801          2050
```

4. Проверка общей (суммарной) информации об установлении BGP соседства в адресном семействе L2VPN EVPN.
 ```sh
Leaf-2# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.123.0.21, local AS number 4200100022
BGP table version is 91, L2VPN EVPN config peers 2, capable peers 2
12 network entries and 17 paths using 2928 bytes of memory
BGP attribute entries [8/1376], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.123.0.41     4 4200100000
                            109      90       91    0    0 00:40:29 5
10.123.0.51     4 4200100000
                            103      88       91    0    0 00:41:18 5
```

Как мы видим, BGP соседство установлено и они обмениваются EVPN маршрутами.

5. Просмотр таблицы BGP адресного семейства L2VPN EVPN.
 ```sh
Leaf-2# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 91, Local Router ID is 10.123.0.21
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.123.0.12:100
* e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                      10.123.0.12                                    0 420010000
0 4200100011 i
*>e                   10.123.0.12                                    0 420010000
0 4200100011 i
* e[3]:[0]:[32]:[10.123.0.12]/88
                      10.123.0.12                                    0 420010000
0 4200100011 i
*>e                   10.123.0.12                                    0 420010000
0 4200100011 i

Route Distinguisher: 10.123.0.22:100    (L2VNI 80100)
*>e[2]:[0]:[0]:[48]:[0050.7966.6806]:[0]:[0.0.0.0]/216
                      10.123.0.12                                    0 420010000
0 4200100011 i
*>l[2]:[0]:[0]:[48]:[0050.7966.6807]:[0]:[0.0.0.0]/216
                      10.123.0.22                       100      32768 i
*>e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      10.123.0.32                                    0 420010000
0 4200100033 i
*>e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                      10.123.0.32                                    0 420010000
0 4200100033 i
*>e[3]:[0]:[32]:[10.123.0.12]/88
                      10.123.0.12                                    0 420010000
0 4200100011 i
*>l[3]:[0]:[32]:[10.123.0.22]/88
                      10.123.0.22                       100      32768 i
*>e[3]:[0]:[32]:[10.123.0.32]/88
                      10.123.0.32                                    0 420010000
0 4200100033 i

Route Distinguisher: 10.123.0.32:100
* e[2]:[0]:[0]:[48]:[0050.7966.6808]:[0]:[0.0.0.0]/216
                      10.123.0.32                                    0 420010000
0 4200100033 i
*>e                   10.123.0.32                                    0 420010000
0 4200100033 i
* e[2]:[0]:[0]:[48]:[0050.7966.6809]:[0]:[0.0.0.0]/216
                      10.123.0.32                                    0 420010000
0 4200100033 i
*>e                   10.123.0.32                                    0 420010000
0 4200100033 i
* e[3]:[0]:[32]:[10.123.0.32]/88
                      10.123.0.32                                    0 420010000
0 4200100033 i
*>e                   10.123.0.32                                    0 420010000
0 4200100033 i
```
Из вывода команды видно, чте в таблице BGP есть EVPN маршруты до удаленных серверов (маршруты 2 типа) и маршруты до удаленных VTEP (маршруты 3 типа).

6. Просмотр L2 маршрутов EVPN.
 ```sh
Leaf-2# show l2route evpn mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Pf):Permanently-Frozen, (Orp): Orphan

Topology    Mac Address    Prod   Flags         Seq No     Next-Hops

----------- -------------- ------ ------------- ---------- ---------------------
------------------
100         0050.7966.6806 BGP    Rcv           0          10.123.0.12 (Label: 80100)
100         0050.7966.6807 Local  L,            0          Eth1/7
100         0050.7966.6808 BGP    Rcv           0          10.123.0.32 (Label: 80100)
100         0050.7966.6809 BGP    Rcv           0          10.123.0.32 (Label: 80100)
```
Из вывода команды видно, что мак адрес Server-2 выучен локально, остальные получены от удаленных VTEP.

### 2. Проверка доступности сетевых узлов.
После настройки VxLAN EVPN проверим, что сервера "видят" друг друга. Для проверки доступности сетевых узлов будем использовать протокол ICMP, а именно команду ping, запускаемую на серверах ЦОД. Ниже представлен пример доступности всех серверов. Проверка проводилась с сервера 1 (Server-1).

```sh
Server-1> ping 10.123.100.11

84 bytes from 10.123.100.11 icmp_seq=1 ttl=64 time=18.914 ms
84 bytes from 10.123.100.11 icmp_seq=2 ttl=64 time=12.561 ms
84 bytes from 10.123.100.11 icmp_seq=3 ttl=64 time=14.446 ms
84 bytes from 10.123.100.11 icmp_seq=4 ttl=64 time=10.016 ms
84 bytes from 10.123.100.11 icmp_seq=5 ttl=64 time=20.848 ms

Server-1> ping 10.123.100.12

84 bytes from 10.123.100.12 icmp_seq=1 ttl=64 time=20.319 ms
84 bytes from 10.123.100.12 icmp_seq=2 ttl=64 time=12.372 ms
84 bytes from 10.123.100.12 icmp_seq=3 ttl=64 time=16.958 ms
84 bytes from 10.123.100.12 icmp_seq=4 ttl=64 time=19.306 ms
84 bytes from 10.123.100.12 icmp_seq=5 ttl=64 time=10.845 ms

Server-1> ping 10.123.100.13

84 bytes from 10.123.100.13 icmp_seq=1 ttl=64 time=14.940 ms
84 bytes from 10.123.100.13 icmp_seq=2 ttl=64 time=11.760 ms
84 bytes from 10.123.100.13 icmp_seq=3 ttl=64 time=15.948 ms
84 bytes from 10.123.100.13 icmp_seq=4 ttl=64 time=11.794 ms
84 bytes from 10.123.100.13 icmp_seq=5 ttl=64 time=12.585 ms
```

С остальных серверов результаты положительные, у каждого сервера есть L2 связность.

С полной версией конфигурации каждого сетевого оборудования можно ознакомиться [здесь](https://github.com/ilya0693/Design-DC-Networks/tree/main/Homework5/Configs).
