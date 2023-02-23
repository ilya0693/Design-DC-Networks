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
