# Построение Underlay сети(IS-IS)

## Цель
Настроить IS-IS для Underlay сети

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроить IS-IS в Underlay сети, для IP связанности между всеми устройствами NXOS
2. План работы, адресное пространство, схема сети, настройки - зафиксированы в документации.

## Общая информация

### Схема стенда

В данном задании стенд собран, согласно схеме, спроектированной в 1ом домашнем задании. Ниже предаставлена реализация спроектируемой сети в эмуляторе EVE-NG. С L2 и L3 схемой сети можно ознакомиться по [ссылке](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework1/README.md#%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F-%D1%81%D0%B5%D1%82%D0%B8-%D1%86%D0%BE%D0%B4-%D0%B8-%D0%B5%D0%B5-%D0%BE%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5).

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework2/Stand_OSPF.png "Схема стенда")

<details>

<summary> Базовая конфигурация VPCs </summary>

Конфигурация VPCS **_"Client1"_**
```sh
set pcname Client1
ip 10.123.100.10 255.255.255.0 10.123.100.1
save
```

Конфигурация VPCS **_"Client2"_**
```sh
set pcname Client2
ip 10.123.100.11 255.255.255.0 10.123.100.1
save
```

Конфигурация VPCS **_"Client3"_**

```sh
set pcname Client3
ip 10.123.100.12 255.255.255.0 10.123.100.1
save
```

Конфигурация VPCS **_"Client4"_**

```sh
set pcname Client4
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
  name Clients
  
interface Vlan100
  description GW_for_Clients->VLAN100
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
  name Clients
  
interface Vlan100
  description GW_for_Clients->VLAN100
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
  description VPCs
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
  name Clients
  
interface Vlan100
  description GW_for_Clients->VLAN100
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
  description VPCs
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
6. Использовать динамическое распределение LSP;
7. Применять минимально необходимую конфигурацию IS-IS;
8. Настраивать аутентификацию.

_Примечание:_ BFD на стенде не поддерживается, потому, настраиваться данный протокол в рамках домашнего задания не будет.

Так как сеть компании ООО "Оринтекс" имеет только 1 POD, то интерфейсы IS-IS будут настраиваться в рамках одной Area - Area 1234. Схема IS-IS представлена на рисунке ниже.
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework3/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-ISIS%20Scheme.png "Схема IS-IS ЦОД")

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

Настраиваем ISIS Instance и NET ID. Также глобально задаем уровень взаимодействия между коммутаторами NX-OS и настроем динамическое распределение LSP.
```sh
router isis UNDERLAY
  net 49.1234.0010.0123.0011.00
  is-type level-1
  dynamic-flooding
    algorithm algorithm-id 128 algorithm-name cisco-dual-spt-v1
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
  dynamic-flooding
    algorithm algorithm-id 128 algorithm-name cisco-dual-spt-v1

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
  dynamic-flooding
    algorithm algorithm-id 128 algorithm-name cisco-dual-spt-v1

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


##### 1.2. Пример конфигурации IS-IS на коммутаторе _"Spine-1"_

Акцентируем внимание на конфигурацию ISIS коммутаторов Spine, так как в настройках есть некоторые отличия при настройке динамического распределения LSP. В качестве примера рассмотрим конфигурацию коммутатора Spine-1.

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

Настраиваем ISIS Instance и NET ID. Также глобально задаем уровень взаимодействия между коммутаторами NX-OS и настроем динамическое распределение LSP. Так как Spine коммутаторы являются лидерами области, на них необходимо задать приоритеты. Чем выше приоритет, тем предпочтительней лидер области. В нашей топологии "основным лидером" будет Spine-1, резервным - Spine-2.
```sh
router isis UNDERLAY
  net 49.1234.0010.0123.0041.00
  is-type level-1
  dynamic-flooding
    algorithm algorithm-id 128 algorithm-name cisco-dual-spt-v1
    area-leader priority 20 algorithm-id 128
```

Далее в режиме конфигурации физических интерфейсов настраиваем:
1. Ассоциацию интерфейсов с ISIS Instance. 
2. Аутентификацию.
3. Интерфейс в режим point-to-point, для "упрощенного" согласования соседства и исключения выбора DIS.

```sh
interface Ethernet1/1-3
  isis authentication-type md5 level-1
  isis authentication key-chain ISIS
  medium p2p
  isis authentication-check level-1
  ip router isis UNDERLAY
```

В режиме конфигурации Loopback ассоциируем интерфейс с ISIS Instance.
```sh
interface loopback0
  ip router isis UNDERLAY
```

На Spine-2 коммутаторе ISIS настраивается идентичным образом. С конфигурацией ISIS можно ознакомиться ниже.
<details> 
<summary> Конфигурация коммутатора <em>"Spine-2"</em> </summary>

```sh
feature isis

key chain ISIS
  key 0
    key-string 7 070c285f4d06

router isis UNDERLAY
  net 49.1234.0010.0123.0051.00
  is-type level-1
  dynamic-flooding
    algorithm algorithm-id 128 algorithm-name cisco-dual-spt-v1
    area-leader priority 10 algorithm-id 128

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

##### 1.3. Проверка работоспособности протокола ISIS
Информация будет добавлена позже




### 2. Проверка доступности сетевых узлов.
После настройки маршрутизации ISIS проверим, что коммутаторы "видят" друг друга. Для проверки доступности сетевых узлов будем использовать протокол ICMP, а именно команду ping и traceroute, запускаемые на коммутаторах ЦОД. Для упрощения тестирования целесообразно проверять доступность Loopback интерфейсов, так как доступность Loopback интерфейсов также подтверждает доступность физических интерфейсов. Ниже представлен пример доступности интерфейса Loopback1, который настроен на коммутаторе Leaf-3. Проверка проводилась с коммутатора Leaf-1.

```sh
Leaf-1# ping 10.123.0.32 source 10.123.0.12
PING 10.123.0.32 (10.123.0.32) from 10.123.0.12: 56 data bytes
64 bytes from 10.123.0.32: icmp_seq=0 ttl=253 time=9.549 ms
64 bytes from 10.123.0.32: icmp_seq=1 ttl=253 time=6.283 ms
64 bytes from 10.123.0.32: icmp_seq=2 ttl=253 time=5.196 ms
64 bytes from 10.123.0.32: icmp_seq=3 ttl=253 time=8.832 ms
64 bytes from 10.123.0.32: icmp_seq=4 ttl=253 time=4.853 ms

--- 10.123.0.32 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 4.853/6.942/9.549 ms

Leaf-1# traceroute 10.123.0.32 source 10.123.0.12
traceroute to 10.123.0.32 (10.123.0.32) from 10.123.0.12 (10.123.0.12), 30 hops max, 40 byte packets
 1  10.123.1.2 (10.123.1.2)  4.996 ms  5.358 ms  3.425 ms
 2  10.123.0.32 (10.123.0.32)  10.606 ms  11.046 ms  10.766 ms

```

Результаты тестирования приведены в таблице 2.

Таблица 2 - Результаты тестирования
|Коммутатор источник |IP-адрес источника     |Коммутатор назначения |IP-адрес назначения|Результат тестирования|
|--------------------|-----------------------|----------------------|-------------------|----------------------|
|Leaf-1              |10.123.0.12            |Leaf-2                |10.123.0.22        |Успешно               |
|Leaf-1              |10.123.0.12            |Leaf-3                |10.123.0.32        |Успешно               |
|Leaf-1              |10.123.0.12            |Spine-1               |10.123.0.41        |Успешно               |
|Leaf-1              |10.123.0.12            |Spine-2               |10.123.0.51        |Успешно               |

_Примечание_: В таблице приведены результаты тестирования только с коммутатора Leaf-1. С остальных коммутаторов результаты также положительные.
