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
show bgp all
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
  Внутренний BGP сосед (iBGP-сосед) — сосед, который находится в той же автономной системе, что и локальный маршрутизатор. iBGP-соседи не обязательно должны быть непосредственно соединены.
  Внешний BGP сосед (eBGP-сосед) — сосед, который находится в автономной системе отличной от локального маршрутизатора. По умолчанию, eBGP-соседи должны быть непосредственно соединены.

BGP выполняет следующие проверки, когда формирует отношения соседства:
* Маршрутизатор должен получить запрос на TCP-соединение с адресом отправителя, который маршрутизатор найдет указанным в списке соседей (команда neighbor).
* Номер автономной системы локального маршрутизатора должен совпадать с номером автономной системы, который указан на соседнем маршрутизаторе командой neighbor remote-as.
* Идентификаторы маршрутизаторов (Router ID) не должны совпадать.
* Если настроена аутентификация, то соседи должны пройти её.
</details>
