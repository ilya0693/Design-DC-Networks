# Построение Underlay сети(eBGP)

## Цель
Настроить BGP для Underlay сети

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроите BGP в Underlay сети, для IP связанности между всеми устройствами NXOS
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
show bgp ipv4 unicast
show bgp ipv4 unicast neighbors
show bgp ipv4 unicast summary
show ip route bgp 
```

</details>

<details> 

<summary> Описание протокола </summary>

#### BGP (Border Gateway Protocol)

BGP — это основной протокол динамической маршрутизации, который используется в Интернете.

Маршрутизаторы, использующие протокол BGP, обмениваются информацией о доступности сетей. Вместе с информацией о сетях передаются различные атрибуты этих сетей, с помощью которых BGP выбирает лучший маршрут и настраиваются политики маршрутизации. Один из основных атрибутов, который передается с информацией о маршруте — это список автономных систем, через которые прошла эта информация. Эта информация позволяет BGP определять где находится сеть относительно автономных систем, исключать петли маршрутизации, а также может быть использована при настройке политик. Маршрутизация осуществляется пошагово от одной автономной системы к другой. Все политики BGP настраиваются, в основном, по отношению к внешним/соседним автономным системам. То есть, описываются правила взаимодействия с ними.
  
BGP это path-vector протокол со следующими общими характеристиками:
  * Использует TCP для передачи данных, это обеспечивает надежную доставку обновлений протокола (порт 179)
  * Отправляет обновления только после изменений в сети (нет периодических обновлений)
  * Периодически отправляет keepalive-сообщения для проверки TCP-соединения
  *Метрика протокола называется path vector или атрибуты (attributes)

Различают следующие виды BGP:
  * Внутренний BGP (Internal BGP, iBGP) — BGP работающий внутри автономной системы. iBGP-соседи не обязательно должны быть непосредственно соединены.
  * Внешний BGP (External BGP, eBGP) — BGP работающий между автономными системами. По умолчанию, eBGP-соседи должны быть непосредственно соединены.
  
Для того чтобы установить отношения соседства, в BGP надо настроить вручную каждого соседа.

Когда указывается сосед локального маршрутизатора, обязательно указывается автономная система соседа. По этой информации BGP определяет тип соседа:
 * Внутренний BGP сосед (iBGP-сосед) — сосед, который находится в той же автономной системе, что и локальный маршрутизатор. iBGP-соседи не обязательно должны быть непосредственно соединены.
 * Внешний BGP сосед (eBGP-сосед) — сосед, который находится в автономной системе отличной от локального маршрутизатора. По умолчанию, eBGP-соседи должны быть непосредственно соединены.

BGP выполняет следующие проверки, когда формирует отношения соседства:
* Маршрутизатор должен получить запрос на TCP-соединение с адресом отправителя, который маршрутизатор найдет указанным в списке соседей (команда neighbor).
* Номер автономной системы локального маршрутизатора должен совпадать с номером автономной системы, который указан на соседнем маршрутизаторе командой neighbor remote-as.
* Идентификаторы маршрутизаторов (Router ID) не должны совпадать.
* Если настроена аутентификация, то соседи должны пройти её.
</details>

## Выполнение домашнего задания

### 1. Конфигурация eBGP.

Выполнив базовую конфигурацию коммутаторов NX-OS далее необходимо обеспечить IP связность всех сетевых узлов. Для этого необходимо настроить маршрутизацию трафика. В рамках домашнего задания, в качестве протокола маршрутизации Underlay сети, будет использоваться протокол eBGP. 
Рекомендуется настраивать маршрутизацию eBGP следующим образом:
1. Использовать автономные системы (ASN), исключающие проблему path-hunting, то есть, коммутаторы уровня Leaf назначаем в разные ASN, Spine - в одну ASN;
2. Использовать multipath для балансировки ECMP;
3. Использовать одну BGP сессию для передачи информации для множественных address families;
4. Использовать таймеры более агрессивно;
5. Использовать BFD;
6. Использовать route-map при анонсе локальных префиксов;
7. Суммировать маршруты только на Leaf коммутаторах;
8. Сохранять конфигурацию минимально необходимой и простой.

_Примечание:_ BFD на стенде не поддерживается, потому, настраиваться данный протокол в рамках домашнего задания не будет.

Схема eBGP представлена на рисунке ниже. На рисунке видно, что, в качестве примера, используются 32 битные ASN32 из приватного диапазона 4 200 000 000 - 4 294 967 294.
В случае данного домашнего задания номера ASN назначались по предпочтениям автора, без какой-либо закономерности.

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework4/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-BGP%20Scheme.png "Схема eBGP ЦОД")

Рассмотрим подробно конфигурацию eBGP одного из коммутаторов ЦОДа.

##### 1.1. Пример конфигурации eBGP на коммутаторе _"Leaf-1"_

Включаем функционал BGP
```sh
feature bgp
```

Создаем route-map для анонсов connected маршрутов. В нашем случае анонсируем только сеть интерфейса Loopback 1.
```sh
route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback1
```

Создаем BGP Instance в AS 4200100011. В режиме конфигурации BGP настраиваем следующие параметры:
1. Router-ID;
2. Reconnect интервал для быстрой конвергенции;
3. Учитываем только длину пути (нужно для балансировки ECMP);
4. В адресном семействе IPv4 анонсируем directly connected маршруты (применяем созданный нами route-map с помощью редистрибьюции);
5. Включаем multipath для балансировки исходящего трафика по ECMP.
```sh
router bgp 4200100011
  router-id 10.123.0.11
  bestpath as-path multipath-relax
  reconnect-interval 12
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths 64
```

Далее в режиме конфигурации BGP настраиваем template, который будет применяться к BGP соседям. В данном template настраиваются:
1. Автономная система BGP соседа (remote-as);
2. Keepalive и Hold таймеры (делаем их более агрессивными для быстрой сходимости BGP);
3. Включаем адресное семейство IPv4 unicast.
```sh
  template peer Spines
    remote-as 4200100000
    timers 3 9
    address-family ipv4 unicast
```

Так как AS у всех Spine коммутаторов одинаковая, созданный template будет применяться ко всем BGP соседям коммутатора Leaf. Ниже представлена конфигурация BGP соседей (настраивается в режиме конфигурации BGP).
```sh
  neighbor 10.123.1.0
    inherit peer Spines
    description Spine-1
  neighbor 10.123.1.2
    inherit peer Spines
    description Spine-2
```

На остальных коммутаторах Nexus eBGP настраивается идентичным образом. С конфигурацией eBGP можно ознакомиться ниже.

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-2"</em> </summary>

```sh
feature bgp

route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback1

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
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-3"</em> </summary>

```sh
feature bgp

route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback1

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
</details> 

Следует отметить, что конфигурация BGP на Spine коммутаторах незначительно отличаются от конфигурации BGP на коммутаторах Leaf. Отличие заключается в том, что Spine не анонсирует никаких маршрутов (кроме тех, что принимает от Leaf) и не имеет template. Пример конфигурации ниже.

<details> 
<summary> Конфигурация коммутатора <em>"Spine-1"</em> </summary>

```sh
feature bgp

router bgp 4200100000
  router-id 10.123.0.41
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    maximum-paths 64
  neighbor 10.123.1.1
    remote-as 4200100011
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.5
    remote-as 4200100022
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.9
    remote-as 4200100033
    timers 3 9
    address-family ipv4 unicast
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Spine-2"</em> </summary>

```sh
router bgp 4200100000
  router-id 10.123.0.51
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    maximum-paths 64
  neighbor 10.123.1.3
    remote-as 4200100011
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.7
    remote-as 4200100022
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.123.1.11
    remote-as 4200100033
    timers 3 9
    address-family ipv4 unicast
```
</details> 

##### 1.2. Проверка работоспособности протокола BGP
После настройки маршрутизации по протоколу eBGP проверим, что у нас установилось соседство и коммутаторы обмениваются маршрутной информацией. В качестве примера, проверим работоспособность маршрутизации eBGP со стороны коммутатора **_Leaf-1_**

1. Проверка общей (суммарной) информации об установлении BGP соседства.
 ```sh
Leaf-1# show bgp ipv4 unicast summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.123.0.11, local AS number 4200100011
BGP table version is 8, IPv4 Unicast config peers 2, capable peers 2
3 network entries and 5 paths using 972 bytes of memory
BGP attribute entries [3/516], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.123.1.0      4 4200100000
                            813     821        8    0    0 00:40:23 2
10.123.1.2      4 4200100000
                            680     685        8    0    0 00:33:46 2
```
Как мы видим, BGP соседство установлено и они обмениваются 2мя префиксами/маршрутами.

2. Подробная информация об установлении BGP соседства
 ```sh
Leaf-1# show bgp ipv4 unicast neighbors
BGP neighbor is 10.123.1.0, remote AS 4200100000, ebgp link, Peer index 3
  Inherits peer configuration from peer-template Spines
  Description: Spine-1
  BGP version 4, remote router ID 10.123.0.41
  Neighbor previous state = OpenConfirm
  BGP state = Established, up for 00:41:57
  Neighbor vrf: default
  Peer is directly attached, interface Ethernet1/1
  Last read 0.740462, hold time = 9, keepalive interval is 3 seconds
  Last written 0.912985, keepalive timer expiry due 00:00:02
  Received 844 messages, 0 notifications, 0 bytes in queue
  Sent 852 messages, 0 notifications, 0(0) bytes in queue
  Enhanced error processing: On
    0 discarded attributes
  Connections established 1, dropped 0
  Last reset by us 00:42:14, due to session closed
  Last error length sent: 0
  Reset error value sent: 0
  Reset error sent major: 104 minor: 0
  Notification data sent:
  Last reset by peer never, due to No error
  Last error length received: 0
  Reset error value received 0
  Reset error received major: 0 minor: 0
  Notification data received:

  Neighbor capabilities:
  Dynamic capability: advertised (mp, refresh, gr) received (mp, refresh, gr)
  Dynamic capability (old): advertised received
  Route refresh capability (new): advertised received
  Route refresh capability (old): advertised received
  4-Byte AS capability: advertised received
  Address family IPv4 Unicast: advertised received
  Graceful Restart capability: advertised received

  Graceful Restart Parameters:
  Address families advertised to peer:
    IPv4 Unicast
  Address families received from peer:
    IPv4 Unicast
  Forwarding state preserved by peer for:
  Restart time advertised to peer: 120 seconds
  Stale time for routes advertised by peer: 300 seconds
  Restart time advertised by peer: 120 seconds
  Extended Next Hop Encoding Capability: advertised received
  Receive IPv6 next hop encoding Capability for AF:
    IPv4 Unicast  VPNv4 Unicast

  Message statistics:
                              Sent               Rcvd
  Opens:                        10                  1
  Notifications:                 0                  0
  Updates:                       2                  3
  Keepalives:                  839                838
  Route Refresh:                 0                  0
  Capability:                    2                  2
  Total:                       852                844
  Total bytes:               16070              16106
  Bytes in queue:                0                  0

  For address family: IPv4 Unicast
  BGP table version 8, neighbor version 8
  2 accepted prefixes (2 paths), consuming 480 bytes of memory
  0 received prefixes treated as withdrawn
  1 sent prefixes (1 paths)
  Last End-of-RIB received 00:00:01 after session start
  Last End-of-RIB sent 00:00:01 after session start
  First convergence 00:00:01 after session start with 1 routes sent

  Local host: 10.123.1.1, Local port: 26353
  Foreign host: 10.123.1.0, Foreign port: 179
  fd = 67

BGP neighbor is 10.123.1.2, remote AS 4200100000, ebgp link, Peer index 4
  Inherits peer configuration from peer-template Spines
  Description: Spine-2
  BGP version 4, remote router ID 10.123.0.51
  Neighbor previous state = OpenConfirm
  BGP state = Established, up for 00:35:20
  Neighbor vrf: default
  Peer is directly attached, interface Ethernet1/2
  Last read 00:00:02, hold time = 9, keepalive interval is 3 seconds
  Last written 0.879728, keepalive timer expiry due 00:00:02
  Received 711 messages, 0 notifications, 0 bytes in queue
  Sent 716 messages, 0 notifications, 0(0) bytes in queue
  Enhanced error processing: On
    0 discarded attributes
  Connections established 1, dropped 0
  Last reset by us 00:35:33, due to session closed
  Last error length sent: 0
  Reset error value sent: 0
  Reset error sent major: 104 minor: 0
  Notification data sent:
  Last reset by peer never, due to No error
  Last error length received: 0
  Reset error value received 0
  Reset error received major: 0 minor: 0
  Notification data received:

  Neighbor capabilities:
  Dynamic capability: advertised (mp, refresh, gr) received (mp, refresh, gr)
  Dynamic capability (old): advertised received
  Route refresh capability (new): advertised received
  Route refresh capability (old): advertised received
  4-Byte AS capability: advertised received
  Address family IPv4 Unicast: advertised received
  Graceful Restart capability: advertised received

  Graceful Restart Parameters:
  Address families advertised to peer:
    IPv4 Unicast
  Address families received from peer:
    IPv4 Unicast
  Forwarding state preserved by peer for:
  Restart time advertised to peer: 120 seconds
  Stale time for routes advertised by peer: 300 seconds
  Restart time advertised by peer: 120 seconds
  Extended Next Hop Encoding Capability: advertised received
  Receive IPv6 next hop encoding Capability for AF:
    IPv4 Unicast  VPNv4 Unicast

  Message statistics:
                              Sent               Rcvd
  Opens:                         6                  1
  Notifications:                 0                  0
  Updates:                       2                  3
  Keepalives:                  707                705
  Route Refresh:                 0                  0
  Capability:                    2                  2
  Total:                       716                711
  Total bytes:               13562              13579
  Bytes in queue:                0                  0

  For address family: IPv4 Unicast
  BGP table version 8, neighbor version 8
  2 accepted prefixes (2 paths), consuming 480 bytes of memory
  0 received prefixes treated as withdrawn
  1 sent prefixes (1 paths)
  Last End-of-RIB received 00:00:01 after session start
  Last End-of-RIB sent 00:00:01 after session start
  First convergence 00:00:01 after session start with 1 routes sent

  Local host: 10.123.1.3, Local port: 179
  Foreign host: 10.123.1.2, Foreign port: 33535
  fd = 68
```
3. Просмотр таблицы BGP адресного семейства IPv4
 ```sh
Leaf-1# show bgp ipv4 unicast
BGP routing table information for VRF default, address family IPv4 Unicast
BGP table version is 8, Local Router ID is 10.123.0.11
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
*>r10.123.0.12/32     0.0.0.0                  0        100      32768 ?
*|e10.123.0.22/32     10.123.1.2                                     0 420010000
0 4200100022 ?
*>e                   10.123.1.0                                     0 420010000
0 4200100022 ?
*|e10.123.0.32/32     10.123.1.2                                     0 420010000
0 4200100033 ?
*>e                   10.123.1.0                                     0 420010000
0 4200100033 ?

```

4. Просмотр маршрутов BGP
 ```sh
Leaf-1# show ip route bgp
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.123.0.22/32, ubest/mbest: 2/0
    *via 10.123.1.0, [20/0], 00:45:34, bgp-4200100011, external, tag 4200100000
    *via 10.123.1.2, [20/0], 00:38:12, bgp-4200100011, external, tag 4200100000
10.123.0.32/32, ubest/mbest: 2/0
    *via 10.123.1.0, [20/0], 00:42:59, bgp-4200100011, external, tag 4200100000
    *via 10.123.1.2, [20/0], 00:37:36, bgp-4200100011, external, tag 4200100000
```

Из таблицы BGP и таблицы маршрутизации видно, что Loopback адреса удаленных Leaf коммутаторов доступны по двум равнозначным путям, а значит трафик будет балансироваться.

### 2. Проверка доступности сетевых узлов.
После настройки маршрутизации eBGP проверим, что коммутаторы "видят" друг друга. Для проверки доступности сетевых узлов будем использовать протокол ICMP, а именно команду ping и traceroute, запускаемые на коммутаторах ЦОД. Для упрощения тестирования целесообразно проверять доступность Loopback интерфейсов, так как доступность Loopback интерфейсов также подтверждает доступность физических интерфейсов. Ниже представлен пример доступности интерфейса Loopback1, который настроен на коммутаторе Leaf-3. Проверка проводилась с коммутатора Leaf-1.

```sh
Leaf-1# ping 10.123.0.32 source 10.123.0.12
PING 10.123.0.32 (10.123.0.32) from 10.123.0.12: 56 data bytes
64 bytes from 10.123.0.32: icmp_seq=0 ttl=253 time=150.657 ms
64 bytes from 10.123.0.32: icmp_seq=1 ttl=253 time=20.501 ms
64 bytes from 10.123.0.32: icmp_seq=2 ttl=253 time=15.829 ms
64 bytes from 10.123.0.32: icmp_seq=3 ttl=253 time=15.874 ms
64 bytes from 10.123.0.32: icmp_seq=4 ttl=253 time=21.052 ms

--- 10.123.0.32 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 15.829/44.782/150.657 ms

Leaf-1# traceroute 10.123.0.32 source 10.123.0.12
traceroute to 10.123.0.32 (10.123.0.32) from 10.123.0.12 (10.123.0.12), 30 hops max, 40 byte packets
 1  10.123.1.2 (10.123.1.2)  17.668 ms  13.11 ms  8.408 ms
 2  10.123.0.32 (10.123.0.32) (AS 4200100033)  32.229 ms  19.336 ms  18.743 ms
```

Результаты тестирования приведены в таблице 1.

Таблица 1 - Результаты тестирования
|Коммутатор источник |IP-адрес источника     |Коммутатор назначения |IP-адрес назначения|Результат тестирования|
|--------------------|-----------------------|----------------------|-------------------|----------------------|
|Leaf-1              |10.123.0.12            |Leaf-2                |10.123.0.22        |Успешно               |
|Leaf-1              |10.123.0.12            |Leaf-3                |10.123.0.32        |Успешно               |

_Примечание_: В таблице приведены результаты тестирования только с коммутатора Leaf-1. С остальных Leaf коммутаторов результаты также положительные.

С полной версией конфигурации каждого сетевого оборудования можно ознакомиться [здесь](https://github.com/ilya0693/Design-DC-Networks/tree/main/Homework3/Configs).
