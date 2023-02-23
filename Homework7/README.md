# VxLAN. VPC.

## Цель
Настроить отказоустойчивое подключение клиентов с использованием VPC

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Подключите клиентов 2-я линками к различным Leaf;
2. Настроите агрегированный канал со стороны клиента;
3. Настроите VPC для работы в Overlay сети;
4. План работы, адресное пространство, схема сети, настройки - зафиксированы в документации.

## Общая информация

<details>

<summary> Схема стенда </summary>
  
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework6/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D1%82%D0%B5%D0%BD%D0%B4%D0%B0%20EVE-NG.png "Схема стенда EVE-NG")

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
show vpc brief
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
  
**_vPC (Virtual Port-Channel)_** - Cisco проприетарная технология виртуализиции, которая позволяет объединить два коммутатора Nexus в единое логическое L2 устройство с точки зрения нижестоящих коммутаторов или устройств (серверов). Технология относится к семейству протоколов под названием MCEC - Multichassis EtherChannel (MLAG), в чем-то даже похожа на технологии объединения коммутаторов Catalyst - VSS и StackWise. Основное отличие от VSS состоит в том, что в vPC каждый коммутатор имеет независимый Control Plane и Management Plane, это дает гарантию того, что при отказе какого-либо компонента в ПО (OSPF процесс, например), сеть продолжит функционировать, в отличие от VSS, где сбой OSPF на Active коммутаторе не вызовет переключения Control Plane на Standby.

</details>

## Выполнение домашнего задания

### 1. Конфигурация Undarlay, VPC и VxLAN EVPN (L2 VNI).

В данной домашней работе нам необходимо объединить коммутаторы Leaf-1 и Leaf-2 в VPC пару и обеспечить L3 связность созданной VPC пары с коммутатором L3. После обеспечения L3 связности следующей задачей необходимо обеспечить L2 связность между серверами с использованием технологий VxLAN/EVPN. Для простоты изучения технологий, в данной домашней работе будет использоваться только L2VNI. Схема дизайна Underlay и Overlay сети представлена на рисунках ниже. 

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework7/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-VPC.drawio.png "Схема Underlay ЦОД")
Рисунок 1 - Схема Underlay ЦОД
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework7/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-VPC%20Overlay.drawio.png "Схема Overlay ЦОД")
Рисунок 2 - Схема Overlay ЦОД

Рассмотрим подробно конфигурацию сетевого оборудования. Следует акцентировать внимание, что, так как дизайн претерпел значительные изменения, ниже будет рассмотрена конфигурация каждого оборудования. Основной упор, при рассмотрении конфигурации, будет акцентирован на VPC, остальная конфигурация подробно рассматриваться не будет, так как была рассмотрена в предыдущих домашних работах.

##### 1.1. Базовая конфигурация и Underlay маршрутизация коммутатора Spine-1
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
  
router bgp 4200100000
  router-id 10.123.0.41
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 64
  neighbor 10.123.1.0/24 remote-as route-map LEAFS_AS
    timers 3 9
    address-family ipv4 unicast
        
boot nxos bootflash:nxos.9.3.10.bin
  
cli alias name wr copy running-config startup-config
```

##### 1.2. Базовая конфигурация и Underlay маршрутизация коммутаторов Leaf
Конфигурация коммутатора Leaf-1
```sh
hostname Leaf-1

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers

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

Конфигурация коммутатора Leaf-2
```sh
hostname Leaf-2

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers

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
  
router bgp 4200100011
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
Следует акцентировать внимание, что, BGP на коммутаторе Leaf-2 настраивается в той же автономной системе, что и коммутатор Leaf-1, так как между этими коммутаторами предполагается создание VPC пары. 
Коммутатор Leaf-3 настраивается также, как и коммутаторы Leaf-1 и Leaf-2. Настройки ниже.
<details>
<summary> Конфигурация коммутатора Leaf-3 </summary>

```sh
hostname Leaf-3

feature bgp
feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers
  
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
</details>

##### 1.3. Конфигурация VPC на коммутаторах Leaf-1 и Leaf-2
Первым шагом включим необходимый функционал для конфигурарования VPC. Также заранее включим функционал, необходимый для работы VxLAN/EVPN. Данные команды вводятся на обоих коммутаторах.
```sh
feature vpc
feature lacp
feature nv overlay
feature vn-segment-vlan-based
nv overlay evpn
```

Далее настраиваем IP-адресацию на mgmt интерфейсах, которые будут использоваться для обмена keepalive сообщениями. 
Конфигурация Leaf-1
```sh
interface mgmt 0
ip address 10.123.254.1/30
no shutdown
```
Конфигурация Leaf-2
```sh
interface mgmt 0
ip address 10.123.254.2/30
no shutdown
```

Следующим шагом настраиваем VPC домен. Рассмотрим конфигурацию Leaf-1.
```sh
vpc domain 10
  role priority 10
  peer-keepalive destination 10.123.254.2 source 10.123.254.1
  peer-switch
  
interface Ethernet1/2-3
  description vpc-peer-link
  switchport
  channel-group 1 mode active
  no shutdown

interface port-channel 1
  description vpc-peer-link (eth1/2-3)
  switchport mode trunk
  vpc peer-link
  no shutdown
```
Немного о конфигурации:
*	При создании VPC создается VPC домен. Он должен быть одинаковым на обоих Leaf коммутаторах в VPC паре;
*	Настраиваем приоритет. В случае Leaf-1 мы указываем, что в VPC паре он является Primary нодой;
*	командой "peer-switch" нужна для работы L2VNI. Если возникнет надобность в L3VNI, то потребуется добавить команду peer-gateway
*	Остальные команды используются для создания peer-link'а.

Аналогичным образом настраивается коммутатор Leaf-2.
```sh
vpc domain 10
  role priority 20
  peer-keepalive destination 10.123.254.1 source 10.123.254.2
  peer-switch
  
interface Ethernet1/2-3
  description vpc-peer-link
  switchport
  channel-group 1 mode active
  no shutdown

interface port-channel 1
  description vpc-peer-link (eth1/2-3)
  switchport mode trunk
  vpc peer-link
  no shutdown
```

Следующим шагом необходимо настроить VPC в сторону клиента. Конфигурация ниже применяется на обоих Leaf коммутаторах в VPC.
```sh
interface Ethernet1/7
  description to_Server-1
  switchport
  channel-group 100 mode active
  no shutdown

interface port-channel 100
  description to_Server-1
  switchport mode trunk
  vpc 10
  no shutdown
```
##### 1.4. Конфигурация Switch-1 и Server-1
Настроив VPC, далее настроим на коммутатора Switch-1 агрегацию каналов. 
```sh
vlan 100
  name Servers
  
interface range ethernet 0/0-1
  description Leafs
  duplex full
  switchport
  channel-group 1 mode active
  no shutdown

interface Port-channel1
  description Leafs
  switchport trunk encapsulation dot1q
  switchport mode trunk
  no shutdown
  
interface Ethernet0/2
 description Server-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

Также выполним базовую настройку Server-1.
```sh
hostname Server-1
  
interface Ethernet0/0
  no shutdown

interface Ethernet0/0.100
 encapsulation dot1Q 100
 ip address 10.123.100.10 255.255.255.0
```

На данном шаге настройку L2 связности между VPC парой и сервером можно считать завершенной. Далее настроим VxLAN/EVPN для обеспечения L2 связности между серверами, через Overlay сеть. 

##### 1.5. Конфигурация VxLAN/EVPN
Первым шагом сконфигурируем Overlay на уровне Spine коммутатора.
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
На данном этапе настройку Spine коммутатора можно считать завершенной.

Далее настроим Leaf коммутаторы, находящихся в VPC паре. Для корректной работы VxLAN/EVPN в VPC паре необходимо на интерфейсе Loopback1 задать secondary IP-адрес. Данный IP-адрес должен быть одинаковым на обоих Leaf коммутаторах.

```sh
interface loopback 1
  ip address 10.123.0.10/32 secondary
```

Далее настроим VxLAN/EVPN на коммутаторах в VPC паре. Ниже представлена конфигурация коммутаторов Leaf-1 и Leaf-2.
```sh
vlan 100
  vn-segment 80100

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
    
interface nve1
  no shutdown
  host-reachability protocol bgp
  advertise virtual-rmac
  source-interface loopback1
  member vni 80100
    ingress-replication protocol bgp

evpn
  vni 80100 l2
    rd 10.123.0.12:100
    route-target import 80100:100
    route-target export 80100:100
```

Коммутатор Lea-3 настраивается идентичным образом. Конфигурация представлена ниже.
<details>

<summary> Конфигурация VxLAN/EVPN на коммутаторе Leaf-3 </summary>

```sh
nv overlay evpn
feature vn-segment-vlan-based
feature nv overlay
 
vlan 100
  vn-segment 80100
  
router bgp 4200100033
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

##### 1.6. Обеспечение L2 связности между коммутатором Leaf-3 и Server-2.
На последнем этапе обеспечим связность между коммутатором Leaf-3 и Server-2. Соответствующие настройки представлены ниже.
Конфигурация Leaf-3
```sh
interface Ethernet1/7
  description Server-2
  switchport mode trunk
  no shutdown
```

Конфигурация Server-2
```sh
hostname Server-1
  
interface Ethernet0/0
  no shutdown

interface Ethernet0/0.100
 encapsulation dot1Q 100
 ip address 10.123.100.12 255.255.255.0
```

##### 1.7. Проверка работоспособности VPC.
Перед тем, как проверять работу VxLAN/EVPN проверим, что у нас коммутаторы Leaf-1 и Leaf-2 собрались в VPC пару.
1. Проверка VPC на коммутаторе Leaf-1
```sh
Leaf-1# show vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 10
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,100


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po100         up     success     success               1,100




Please check "show vpc consistency-parameters vpc <vpc-num>" for the
consistency reason of down vpc and for type-2 consistency reasons for
any vpc.
```

2. Проверка VPC на коммутаторе Leaf-2
```sh
Leaf-2# show vpc
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 10
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : secondary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,100


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po100         up     success     success               1,100




Please check "show vpc consistency-parameters vpc <vpc-num>" for the
consistency reason of down vpc and for type-2 consistency reasons for
any vpc.
```

Как мы видим, коммутаторы успешно собраны в VPC пару.

##### 1.8. Проверка работоспособности VxLAN EVPN
После настройки L2 связности с использованием технологий VxLAN EVPN проверим, что у нас VxLAN туннель работает, а EVPN маршруты передаются. В качестве примера, проверим работоспособность маршрутизации VxLAN EVPN со стороны коммутатора Leaf-3, чтобы продемонстрировать, как Leaf-3 видит коммутаторы в VPC паре.

1. Проверка состояния NVE интерфейса (VxLAN туннеля).
 ```sh
Leaf-3# show nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      10.123.0.10                             Up    CP        00:26:40 n/a

```
Как мы видим, Leaf-3 видит коммутаторы в VPC паре через их общий (secondary) IP-адрес. 

2. Более подробная информация об nve интерфейсе.
 ```sh
Leaf-3# show nve vni
Codes: CP - Control Plane        DP - Data Plane
       UC - Unconfigured         SA - Suppress ARP
       SU - Suppress Unknown Unicast
       Xconn - Crossconnect
       MS-IR - Multisite Ingress Replication

Interface VNI      Multicast-group   State Mode Type [BD/VRF]      Flags
--------- -------- ----------------- ----- ---- ------------------ -----
nve1      80100    UnicastBGP        Up    CP   L2 [100]
```
Из вывода команды видим, что информацию об удаленных мак адресах Leaf-3 получает через Control Plane (через BGP) с использованием ingress репликации. Также из вывода команды видно, что nve интерфейс в состоянии UP и информацию о VNI.

3. Просмотр статуса VxLAN интерфейса
 ```sh
Leaf-3# show vxlan interface
Interface       Vlan    VPL Ifindex     LTL             HW VP
=========       ====    ===========     ===             =====
Eth1/7          100     0x530647fa      0x1801          2050
```

4. Проверка общей (суммарной) информации об установлении BGP соседства в адресном семействе L2VPN EVPN.
 ```sh
Leaf-3# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.123.0.31, local AS number 4200100033
BGP table version is 20, L2VPN EVPN config peers 1, capable peers 1
6 network entries and 6 paths using 1224 bytes of memory
BGP attribute entries [5/860], BGP AS path entries [1/10]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.123.0.41     4 4200100000
                             44      39       20    0    0 00:31:31 2

```

Как мы видим, BGP соседство установлено и они обмениваются EVPN маршрутами.

5. Просмотр таблицы BGP адресного семейства L2VPN EVPN.
 ```sh
Leaf-3# show bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 20, Local Router ID is 10.123.0.31
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.123.0.12:100
*>e[2]:[0]:[0]:[48]:[aabb.cc00.5000]:[0]:[0.0.0.0]/216
                      10.123.0.10                                    0 420010000
0 4200100011 i
*>e[3]:[0]:[32]:[10.123.0.10]/88
                      10.123.0.10                                    0 420010000
0 4200100011 i

Route Distinguisher: 10.123.0.32:100    (L2VNI 80100)
*>e[2]:[0]:[0]:[48]:[aabb.cc00.5000]:[0]:[0.0.0.0]/216
                      10.123.0.10                                    0 420010000
0 4200100011 i
*>l[2]:[0]:[0]:[48]:[aabb.cc00.6000]:[0]:[0.0.0.0]/216
                      10.123.0.32                       100      32768 i
*>e[3]:[0]:[32]:[10.123.0.10]/88
                      10.123.0.10                                    0 420010000
0 4200100011 i
*>l[3]:[0]:[32]:[10.123.0.32]/88
                      10.123.0.32                       100      32768 i

```
Из вывода команды видно, чте в таблице BGP есть EVPN маршруты до удаленных серверов (маршруты 2 типа) и маршруты до удаленных VTEP (маршруты 3 типа). Также из вывода команды видно, что мак-адрес удаленного сервера доступен за общим IP-адресом VPC пары.

6. Просмотр L2 маршрутов EVPN.
 ```sh
Leaf-3# show l2route evpn mac all

Flags -(Rmac):Router MAC (Stt):Static (L):Local (R):Remote (V):vPC link
(Dup):Duplicate (Spl):Split (Rcv):Recv (AD):Auto-Delete (D):Del Pending
(S):Stale (C):Clear, (Ps):Peer Sync (O):Re-Originated (Nho):NH-Override
(Pf):Permanently-Frozen, (Orp): Orphan

Topology    Mac Address    Prod   Flags         Seq No     Next-Hops

----------- -------------- ------ ------------- ---------- ---------------------------------------
100         aabb.cc00.5000 BGP    Rcv           0          10.123.0.10 (Label: 80100)
100         aabb.cc00.6000 Local  L,            0          Eth1/7
```
Из вывода команды видно, что мак адрес Server-2 выучен локально, а удаленный мак адрес получен от удаленных VTEP, собранных в VPC пару.

### 2. Проверка доступности серверов.
После настройки VxLAN EVPN проверим, что сервера "видят" друг друга. Для проверки доступности сетевых узлов будем использовать протокол ICMP, а именно команду ping, запускаемую на серверах ЦОД. Ниже представлен пример доступности всех серверов. Проверка проводилась с сервера 2 (Server-2).

```sh
Server-2#ping 10.123.100.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.123.100.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 13/22/59 ms
```

Из результатов видно, что удаленный сервер, находящийся за VPC парой, доступен. Протестируем отказоустойчивость, сымитировав поломку коммутатора Leaf-1 (см. рисунок ниже).
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework7/Leaf-1%20fail.png "Поломка Leaf-1")



С полной версией конфигурации каждого сетевого оборудования можно ознакомиться [здесь](https://github.com/ilya0693/Design-DC-Networks/tree/main/Homework5/Configs).
