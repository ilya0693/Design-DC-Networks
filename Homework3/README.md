# Построение Underlay сети(IS-IS)

## Цель
Настроить IS-IS для Underlay сети

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите IS-IS в Underlay сети, для IP связанности между всеми устройствами NXOS
2. План работы, адресное пространство, схема сети, настройки - зафиксированы в документации.

## Общая информация

### Схема стенда

В данном задании стенд собран, согласно схеме, спроектированной в 1ом домашнем задании. Ниже предаставлена реализация спроектируемой сети в эмуляторе EVE-NG. С L2 и L3 схемой сети можно ознакомиться по [ссылке](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework1/README.md#%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F-%D1%81%D0%B5%D1%82%D0%B8-%D1%86%D0%BE%D0%B4-%D0%B8-%D0%B5%D0%B5-%D0%BE%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5).

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework3/Stand_ISIS.png "Схема стенда")

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
ip 10.123.100.12 255.255.255.0 10.123.100.1
save
```

Конфигурация VPCS **_"Server-4"_**

```sh
set pcname Server-4
ip 10.123.100.13 255.255.255.0 10.123.100.1
save
```
</details>

<details>

<summary> Базовая конфигурация коммутаторов NX-OS </summary>

Конфигурация коммутатора **_Leaf-1_**
  ```sh
hostname Leaf-1

feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers
  
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
  description VPCs
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
```

Конфигурация коммутатора **_Leaf-2_**
  ```sh
hostname Leaf-2

feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers
  
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
```

Конфигурация коммутатора **_Leaf-3_**
  ```sh
hostname Leaf-3

feature interface-vlan

no ip domain-lookup
ip domain-name dc.lab

vlan 100
  name Servers
  
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
  
interface ethernet 1/6-7
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
```

Конфигурация коммутатора **_Spine-1_**  
```sh
hostname Spine-1

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
```

Конфигурация коммутатора **_Spine-2_**
 ```sh
hostname Spine-2

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
```
</details>


<details> 

<summary>Полезные команды </summary>

```
show isis
show isis interface
show ip route
show isis adjacency
```

</details>

<details> 

<summary> Описание протокола </summary>

#### IS-IS (Intermediate System to Intermediate System)

Протокол маршрутизации промежуточных систем (англ. IS-IS) — это протокол внутренних шлюзов (IGP), стандартизированный ISO и использующийся в основном в крупных сетях провайдеров услуг. IS-IS может также использоваться в корпоративных сетях особо крупного масштаба. IS-IS — это протокол маршрутизации на основе состояния каналов. Он обеспечивает быструю сходимость и отличную масштабируемость. Как и все протоколы на основе состояния каналов, IS-IS очень экономно использует пропускную способность сетей.

**IS-IS - работает поверх Ethernet v.1**, из-за этого сложно использовать его для построения VPN, так как VPN - это по сути инкапсуляция «IP внутрь IP», а тут IP нет. Зато нет лишних заголовков.

Скорость сходимости у IS-IS выше, поэтому в больших сетях может быть лучше.
Из коробки **может работать с 200-250 устройствами**, в то время когда OSPF для 50-70 устройств.
IS-IS более масштабируем - в него проще добавить например метку для MPLS.

IS-IS часто работает с BGP и можно дожидаться поднятия BGP.

**Уровни IS-IS**

- L1 - внутри одной зоны
- L2 - между несколькими зонами. L2 можно сравнить с магистралью.
Можно сделать только L2 взаимодействие, но это неоптимально.

</details>

## Выполнение домашнего задания

### 1. Конфигурация IS-IS
Выполнив базовую конфигурацию коммутаторов NX-OS далее необходимо обеспечить IP связность всех сетевых узлов. Для этого необходимо настроить маршрутизацию трафика. В рамках домашнего задания, в качестве протокола маршрутизации Underlay сети, будет использоваться протокол IS-IS. 
Рекомендуется настраивать маршрутизацию ISIS следующим образом:
1. Использовать p2p на интерфейсах;
2. Для каждого POD использовать свою зону (задается командой _net_);
3. Так как наша топология имеет 1 POD то, в рамках одного POD'а рекомендуется использовать L1 взаимодействие между Leaf-Spine коммутаторами. Такой вариант выбран с целью дальнейшего масштабирования (например, когда появится еще один POD и между POD'ами будет связность через Super Spine коммутаторы);
4. Использовать BFD;
5. Избегать использования redistribute;
6. Использовать динамическое распределение LSP (не поддерживается на виртуальных NX-OS);
7. Применять минимально необходимую конфигурацию IS-IS;
8. Настраивать аутентификацию.

_Примечание:_ BFD на стенде не поддерживается, потому, настраиваться данный протокол в рамках домашнего задания не будет.

Так как сеть компании ООО "Оринтекс" имеет только 1 POD, то интерфейсы IS-IS будут настраиваться в рамках одной Area - Area 1234. Схема IS-IS представлена на рисунке ниже.
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework3/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-ISIS%20Scheme.png "Схема IS-IS ЦОД")

Следует отметить, что, в случае масштабирования ЦОД (например, появился второй POD) рекомендуется использовать несколько Area для каждого POD. Помимо использования нескольких Area немаловажно настроить уровень взаимодействия между ISIS соседями. Ниже перечислены (на взгляд автора) основные рекомендации при проектировании ЦОД с использованием протокола ISIS:
1. Если ЦОД имеет только 1 POD, то все Spine/Leaf коммутаторы назначаем в одну зону и глобально включаем (в режиме конфигурации router isis) L1 взаимодействие между этими устройствами.
2. В случае наличия 2х и более POD назначаем коммутаторы уровня Spine/Leaf и Super Spine в разные Area, в зависимости от их расположения в рамках PODа. В данном сценарии уровень взаимодействия между ISIS соседями настраивается следующим образом:
2.1. Leaf коммутаторы взаимодействуют со Spine коммутаторами только на уровне L1, настраиваем данное взаимодействие глобально.
2.2. Spine коммутаторы взаимодействуют с Leaf коммутаторами на уровне L1, а с Super Spine - на уровне L2, то есть, Spine коммутатор должен "уметь" устанавливать как L1, так и L2 соседство.
2.3. Super Spine коммутаторы взаимодействуют с Leaf коммутаторами только на уровне L2, настраиваем данное взаимодействие глобально.
Ниже представлен пример схемы межсетевого взаимодействия между несколькими двумя POD в рамках одного ЦОДа
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework3/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-ISIS%20Multi-area.png "Схема ISIS с двумя POD")

Рассмотрим подробно конфигурацию ISIS одного из коммутаторов ЦОДа.

##### 1.1. Пример конфигурации IS-IS на коммутаторе _"Leaf-1"_

Включаем функционал ISIS
```sh
feature isis
```

Настраиваем цепочку ключей, которая будет использоваться для аутентификации ISIS сессий.
```sh
key chain ISIS
  key 0
    key-string 7 070c285f4d06
```

Настраиваем ISIS Instance и NET ID. Также глобально задаем уровень взаимодействия между коммутаторами NX-OS.
```sh
router isis UNDERLAY
  net 49.1234.0010.0123.0011.00
  is-type level-1
```

NET ID обозначает следующее:
1. AFI (Authority and Format Identifier) - 49.  AFI является частью номера области и указывает тип адреса (у нас приватный, поэтому он равен 49).
2. Area ID - 1234. Это номер области (Area ID), которой принадлежит коммутатор.
3. System ID - 0010.0123.0011. Это идентификатор, в нашем случае, коммутатора Leaf-1. У каждого коммутатора в топологии он должен быть уникальным, поскольку именно по System ID коммутатора "узнают" друг друга при осознании топологии.
4. Selector - 00. Значение Selector, равное 00, обозначало в NET адресах, что адрес принадлежит самому маршрутизатору. В большинстве случаев он всегда равен нулю.

Далее в режиме конфигурации физических интерфейсов настраиваем:
1. Ассоциацию интерфейсов с ISIS Instance. 
2. Аутентификацию.
3. Интерфейс в режим point-to-point, для "упрощенного" согласования соседства и исключения выбора DIS.

```sh
interface Ethernet1/1-2
  isis authentication-type md5 level-1
  isis authentication key-chain ISIS
  medium p2p
  isis authentication-check level-1
  ip router isis UNDERLAY
```

В режиме конфигурации Loopback интерфейсов ассоциируем интерфейсы с ISIS Instance.
```sh
interface loopback0
  ip router isis UNDERLAY

interface loopback1
  ip router isis UNDERLAY
```

На остальных Leaf коммутаторах ISIS настраивается идентичным образом. С конфигурацией ISIS можно ознакомиться ниже.

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-2"</em> </summary>

```sh
feature isis

key chain ISIS
  key 0
    key-string 7 070c285f4d06

router isis UNDERLAY
  net 49.1234.0010.0123.0021.00
  is-type level-1

interface Ethernet1/1-2
  isis authentication-type md5 level-1
  isis authentication key-chain ISIS
  medium p2p
  isis authentication-check level-1
  ip router isis UNDERLAY

interface loopback0
  ip router isis UNDERLAY

interface loopback1
  ip router isis UNDERLAY
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-3"</em> </summary>

```sh
feature isis

key chain ISIS
  key 0
    key-string 7 070c285f4d06

router isis UNDERLAY
  net 49.1234.0010.0123.0031.00
  is-type level-1

interface Ethernet1/1-2
  isis authentication-type md5 level-1
  isis authentication key-chain ISIS
  medium p2p
  isis authentication-check level-1
  ip router isis UNDERLAY

interface loopback0
  ip router isis UNDERLAY

interface loopback1
  ip router isis UNDERLAY
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Spine-1"</em> </summary>

```sh
feature isis

key chain ISIS
  key 0
    key-string 7 070c285f4d06

router isis UNDERLAY
  net 49.1234.0010.0123.0041.00
  is-type level-1-2

interface Ethernet1/1-3
  isis authentication-type md5 level-1
  isis authentication key-chain ISIS
  medium p2p
  isis authentication-check level-1
  ip router isis UNDERLAY

interface loopback0
  ip router isis UNDERLAY
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Spine-2"</em> </summary>

```sh
feature isis

key chain ISIS
  key 0
    key-string 7 070c285f4d06

router isis UNDERLAY
  net 49.1234.0010.0123.0051.00
  is-type level-1-2

interface Ethernet1/1-3
  isis authentication-type md5 level-1
  isis authentication key-chain ISIS
  medium p2p
  isis authentication-check level-1
  ip router isis UNDERLAY

interface loopback0
  ip router isis UNDERLAY
```
</details> 

##### 1.2. Проверка работоспособности протокола ISIS
После настройки маршрутизации по протоколу ISIS проверим, что у нас установилось соседство и коммутаторы обмениваются маршрутной информацией. В качестве примера, проверим работоспособность маршрутизации ISIS со стороны коммутатора **_Leaf-1_**

1. Проверка установления соседства
 ```sh
Leaf-1# show isis adjacency
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
Spine-1         N/A             1      UP     00:00:30   Ethernet1/1
Spine-2         N/A             1      UP     00:00:30   Ethernet1/2
```
Как мы видим, соседство установлено успешно.

2. Проверка интерфейсов, задействованных в ISIS instance и их состояние.
 ```sh
Leaf-1# show isis interface
IS-IS process: UNDERLAY VRF: default
loopback0, Interface status: protocol-up/link-up/admin-up
  IP address: 10.123.0.11, IP subnet: 10.123.0.11/32
  IPv6 routing is disabled
  Level1
    No auth type and keychain
    Auth check set
  Level2
    No auth type and keychain
    Auth check set
  Index: 0x0003, Local Circuit ID: 0x01, Circuit Type: L1
  BFD IPv4 is locally disabled for Interface loopback0
  BFD does not support AF IPv4
  BFD IPv6 is locally disabled for Interface loopback0
  BFD does not support AF IPv6
  MTR is disabled
  Passive level: level-2
  Level      Metric
  1               1
  2               1
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
    1  0        1       no   UP    UP       yes      DN       no
    2  0        1       no   DN    DN       no       DN       no

loopback1, Interface status: protocol-up/link-up/admin-up
  IP address: 10.123.0.12, IP subnet: 10.123.0.12/32
  IPv6 routing is disabled
  Level1
    No auth type and keychain
    Auth check set
  Level2
    No auth type and keychain
    Auth check set
  Index: 0x0004, Local Circuit ID: 0x01, Circuit Type: L1
  BFD IPv4 is locally disabled for Interface loopback1
  BFD does not support AF IPv4
  BFD IPv6 is locally disabled for Interface loopback1
  BFD does not support AF IPv6
  MTR is disabled
  Passive level: level-2
  Level      Metric
  1               1
  2               1
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
    1  0        1       no   UP    UP       yes      DN       no
    2  0        1       no   DN    DN       no       DN       no

Ethernet1/1, Interface status: protocol-up/link-up/admin-up
  IP address: 10.123.1.1, IP subnet: 10.123.1.0/31
  IPv6 routing is disabled
     Auth type:MD5
    Auth keychain: ISIS
    Auth check set
  Index: 0x0001, Local Circuit ID: 0x01, Circuit Type: L1
  BFD IPv4 is locally disabled for Interface Ethernet1/1
  BFD IPv6 is locally disabled for Interface Ethernet1/1
  MTR is disabled
  Extended Local Circuit ID: 0x1A000000, P2P Circuit ID: 0000.0000.0000.00
  Retx interval: 5, Retx throttle interval: 66 ms
  LSP interval: 33 ms, MTU: 1500
  MTU check OFF on P2P interface
  P2P Adjs: 1, AdjsUp: 1, Priority 64
  Hello Interval: 10, Multi: 3, Next IIH: 00:00:05
  MT    Adjs   AdjsUp  Metric   CSNP  Next CSNP  Last LSP ID
  1          1        1      40     60  00:00:54   ffff.ffff.ffff.ff-ff
  2          0        0      40     60  Inactive   ffff.ffff.ffff.ff-ff
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
    1  0        40      no   UP    UP       yes      DN       no
    2  0        40      no   DN    DN       no       DN       no

Ethernet1/2, Interface status: protocol-up/link-up/admin-up
  IP address: 10.123.1.3, IP subnet: 10.123.1.2/31
  IPv6 routing is disabled
     Auth type:MD5
    Auth keychain: ISIS
    Auth check set
  Index: 0x0002, Local Circuit ID: 0x01, Circuit Type: L1
  BFD IPv4 is locally disabled for Interface Ethernet1/2
  BFD IPv6 is locally disabled for Interface Ethernet1/2
  MTR is disabled
  Extended Local Circuit ID: 0x1A000200, P2P Circuit ID: 0000.0000.0000.00
  Retx interval: 5, Retx throttle interval: 66 ms
  LSP interval: 33 ms, MTU: 1500
  MTU check OFF on P2P interface
  P2P Adjs: 1, AdjsUp: 1, Priority 64
  Hello Interval: 10, Multi: 3, Next IIH: 00:00:05
  MT    Adjs   AdjsUp  Metric   CSNP  Next CSNP  Last LSP ID
  1          1        1      40     60  00:00:07   ffff.ffff.ffff.ff-ff
  2          0        0      40     60  Inactive   ffff.ffff.ffff.ff-ff
  Topologies enabled:
    L  MT  Metric  MetricCfg  Fwdng IPV4-MT  IPV4Cfg  IPV6-MT  IPV6Cfg
    1  0        40      no   UP    UP       yes      DN       no
    2  0        40      no   DN    DN       no       DN       no
```
Из результатов вывода команды видно, что интерфейсы в активном состоянии и на них используется аутентификация.

3. Проверка ISIS процесса
 ```sh
ISIS process : UNDERLAY
 Instance number :  1
 UUID: 1090519320
 Process ID 23857
VRF: default
  System ID : 0010.0123.0011  IS-Type : L1
  SAP : 412  Queue Handle : 11
  Maximum LSP MTU: 1492
  Stateful HA enabled
  Graceful Restart enabled. State: Inactive
  Last graceful restart status : none
  Start-Mode Complete
  BFD IPv4 is globally disabled for ISIS process: UNDERLAY
  BFD IPv6 is globally disabled for ISIS process: UNDERLAY
  Topology-mode is base
  Metric-style : advertise(wide), accept(narrow, wide)
  Area address(es) :
    49.1234
  Process is up and running
  VRF ID: 1
  Stale routes during non-graceful controlled restart
  Enable resolution of L3->L2 address for ISIS adjacency
  SRTE: Not registered
  OAM: Not registered
  SR IPv4 is not configured and disabled for ISIS process: UNDERLAY
  SR IPv6 is not configured and disabled for ISIS process: UNDERLAY
 SRv6 is disabled for ISIS process: UNDERLAY
  Interfaces supported by IS-IS :
    loopback0
    loopback1
    Ethernet1/1
    Ethernet1/2
  Topology : 0
  Address family IPv4 unicast :
    Number of interface : 4
    Distance : 115
    Default-information not originated
  Address family IPv6 unicast :
    Number of interface : 0
    Distance : 115
    Default-information not originated
  Topology : 2
  Address family IPv4 unicast :
    Number of interface : 0
    Distance : 115
    Default-information not originated
  Address family IPv6 unicast :
    Number of interface : 0
    Distance : 115
    Default-information not originated
  Level1
  No auth type and keychain
  Auth check set
  Level2
  No auth type and keychain
  Auth check set
  L1 Next SPF: Inactive
  L2 Next SPF: Inactive
  Attached bits
   MT-0 L-1: Att 0 Spf-att 0 Cfg 1 Adv-att 0
   MT-0 L-2: Att 0 Spf-att 0 Cfg 1 Adv-att 0
```

4. Просмотр маршрутов ISIS.
 ```sh
Leaf-1# show ip route isis
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.123.0.21/32, ubest/mbest: 2/0
    *via 10.123.1.0, Eth1/1, [115/81], 00:11:00, isis-UNDERLAY, L1
    *via 10.123.1.2, Eth1/2, [115/81], 00:07:40, isis-UNDERLAY, L1
10.123.0.22/32, ubest/mbest: 2/0
    *via 10.123.1.0, Eth1/1, [115/81], 00:11:00, isis-UNDERLAY, L1
    *via 10.123.1.2, Eth1/2, [115/81], 00:07:40, isis-UNDERLAY, L1
10.123.0.31/32, ubest/mbest: 2/0
    *via 10.123.1.0, Eth1/1, [115/81], 00:11:01, isis-UNDERLAY, L1
    *via 10.123.1.2, Eth1/2, [115/81], 00:07:40, isis-UNDERLAY, L1
10.123.0.32/32, ubest/mbest: 2/0
    *via 10.123.1.0, Eth1/1, [115/81], 00:11:01, isis-UNDERLAY, L1
    *via 10.123.1.2, Eth1/2, [115/81], 00:07:40, isis-UNDERLAY, L1
10.123.0.41/32, ubest/mbest: 1/0
    *via 10.123.1.0, Eth1/1, [115/41], 00:11:05, isis-UNDERLAY, L1
10.123.0.51/32, ubest/mbest: 1/0
    *via 10.123.1.2, Eth1/2, [115/41], 00:07:26, isis-UNDERLAY, L1
10.123.1.4/31, ubest/mbest: 1/0
    *via 10.123.1.0, Eth1/1, [115/80], 00:11:05, isis-UNDERLAY, L1
10.123.1.6/31, ubest/mbest: 1/0
    *via 10.123.1.2, Eth1/2, [115/80], 00:07:40, isis-UNDERLAY, L1
10.123.1.8/31, ubest/mbest: 1/0
    *via 10.123.1.0, Eth1/1, [115/80], 00:11:05, isis-UNDERLAY, L1
10.123.1.10/31, ubest/mbest: 1/0
    *via 10.123.1.2, Eth1/2, [115/80], 00:07:40, isis-UNDERLAY, L1
```

Из маршрутной информации видно, что для коммутатора **_Leaf-1_** коммутаторы **_Leaf-2_** и **_Leaf-3_** достижимы через 2 коммутатора _Spine_ одновременно, то есть, у нас используется ECMP (балансировка трафика с равной стоимостью) из-за наличия "одинакового" пути.


### 2. Проверка доступности сетевых узлов.
После настройки маршрутизации ISIS проверим, что коммутаторы "видят" друг друга. Для проверки доступности сетевых узлов будем использовать протокол ICMP, а именно команду ping и traceroute, запускаемые на коммутаторах ЦОД. Для упрощения тестирования целесообразно проверять доступность Loopback интерфейсов, так как доступность Loopback интерфейсов также подтверждает доступность физических интерфейсов. Ниже представлен пример доступности интерфейса Loopback1, который настроен на коммутаторе Leaf-3. Проверка проводилась с коммутатора Leaf-1.

```sh
Leaf-1# ping 10.123.0.32 source 10.123.0.12
PING 10.123.0.32 (10.123.0.32) from 10.123.0.12: 56 data bytes
64 bytes from 10.123.0.32: icmp_seq=0 ttl=253 time=42.16 ms
64 bytes from 10.123.0.32: icmp_seq=1 ttl=253 time=10.162 ms
64 bytes from 10.123.0.32: icmp_seq=2 ttl=253 time=11.085 ms
64 bytes from 10.123.0.32: icmp_seq=3 ttl=253 time=10.654 ms
64 bytes from 10.123.0.32: icmp_seq=4 ttl=253 time=12.557 ms

--- 10.123.0.32 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 10.162/17.323/42.16 ms

Leaf-1# traceroute 10.123.0.32 source 10.123.0.12
traceroute to 10.123.0.32 (10.123.0.32) from 10.123.0.12 (10.123.0.12), 30 hops max, 40 byte packets
 1  10.123.1.2 (10.123.1.2)  6.021 ms  6.125 ms  4.167 ms
 2  10.123.0.32 (10.123.0.32)  12.639 ms  14.213 ms  10.91 ms

```

Результаты тестирования приведены в таблице 1.

Таблица 1 - Результаты тестирования
|Коммутатор источник |IP-адрес источника     |Коммутатор назначения |IP-адрес назначения|Результат тестирования|
|--------------------|-----------------------|----------------------|-------------------|----------------------|
|Leaf-1              |10.123.0.12            |Leaf-2                |10.123.0.22        |Успешно               |
|Leaf-1              |10.123.0.12            |Leaf-3                |10.123.0.32        |Успешно               |
|Leaf-1              |10.123.0.12            |Spine-1               |10.123.0.41        |Успешно               |
|Leaf-1              |10.123.0.12            |Spine-2               |10.123.0.51        |Успешно               |

_Примечание_: В таблице приведены результаты тестирования только с коммутатора Leaf-1. С остальных коммутаторов результаты также положительные.

С полной версией конфигурации каждого сетевого оборудования можно ознакомиться [здесь](https://github.com/ilya0693/Design-DC-Networks/tree/main/Homework3/Configs).
