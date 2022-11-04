# Построение Underlay сети(OSPF)

## Цель
Настроить OSPF для Underlay сети

## Описание/Пошаговая инструкция выполнения домашнего задания:
В этой самостоятельной работе мы ожидаем, что вы самостоятельно:

1. Настроить OSPF в Underlay сети, для IP связанности между всеми устройствами NXOS
2. План работы, адресное пространство, схема сети, настройки - зафиксированы в документации.

## Общая информация

### Схема стенда

В данном задании стенд собран, согласно схеме, спроектированной в 1ом домашнем задании. Ниже предаставлена реализация спроектируемой сети в эмуляторе EVE-NG. С L2 и L3 схемой сети можно ознакомиться по [ссылке](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework1/README.md#%D1%82%D0%BE%D0%BF%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D1%8F-%D1%81%D0%B5%D1%82%D0%B8-%D1%86%D0%BE%D0%B4-%D0%B8-%D0%B5%D0%B5-%D0%BE%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5).

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework2/Stand_OSPF.png "Схема стенда")

### Описание протокола

## Выполнение домашнего задания

### 1. Конфигурация VPCS

Первым шагом сконфигурируем устройства клиентов, которые, в дальнейшем, будут взаимодействовать между собой в рамках VxLAN фабрики.

Конфигурация VPCS **_"Client1"_**

```sh
set pcname Client1
ip 10.123.100.10 255.255.255.0 10.123.100.1
save
```

<details> 

<summary> Проверка конфигурации VPCS "Client1" </summary>

```sh
Client1> show ip

NAME        : Client1[1]
IP/MASK     : 10.123.100.10/24
GATEWAY     : 10.123.100.1
DNS         :
MAC         : 00:50:79:66:68:06
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

```
</details> 

Конфигурация VPCS **_"Client2"_**

```sh
set pcname Client2
ip 10.123.100.11 255.255.255.0 10.123.100.1
save
```

<details> 

<summary> Проверка конфигурации VPCS "Client2" </summary>

```sh
Client2> show ip

NAME        : Client2[1]
IP/MASK     : 10.123.100.11/24
GATEWAY     : 10.123.100.1
DNS         :
MAC         : 00:50:79:66:68:07
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

```
</details> 

Конфигурация VPCS **_"Client3"_**

```sh
set pcname Client3
ip 10.123.100.12 255.255.255.0 10.123.100.1
save
```

<details> 

<summary> Проверка конфигурации VPCS "Client3" </summary>

```sh
Client3> show ip

NAME        : Client3[1]
IP/MASK     : 10.123.100.12/24
GATEWAY     : 10.123.100.1
DNS         :
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

```
</details> 

Конфигурация VPCS **_"Client4"_**

```sh
set pcname Client4
ip 10.123.100.13 255.255.255.0 10.123.100.1
save
```

<details> 

<summary> Проверка конфигурации VPCS "Client4" </summary>

```sh
Client4> show ip

NAME        : Client4[1]
IP/MASK     : 10.123.100.13/24
GATEWAY     : 10.123.100.1
DNS         :
MAC         : 00:50:79:66:68:09
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

```
</details> 

### 2. Основная конфигурация Spine и Leaf коммутаторов

Перед тем, как приступисть к конфигурации маршрутизации в Underlay сети необходимо выполнить базовые настройки коммутаторов. В качестве примера рассмотрим базовую конфигурацию коммутатора **_"Leaf-1"_** более подробно. 

##### 2.1 Пример конфигурации коммутатора _"Leaf-1"_

Настройка имени хоста
```sh
hostname Leaf-1 
```
Включение функционала SVI для создания interface VLAN
```sh
feature interface-vlan
```

Выключаем разрешение доменных имен и задаем доменное имя _dc.lab_
```sh
no ip domain-lookup
ip domain-name dc.lab
```
Создаем VLAN 100 и даем ему имя
```sh
vlan 100
  name Clients
```
Создаем интерфейс VLAN100, который будет использоваться в качестве шлюза по умолчанию для клиентов в 100 VLAN'е
```sh
interface Vlan100
  description GW_for_Clients->VLAN100
  no shutdown
  no ip redirects
  ip address 10.123.100.1/24
```

Конфигурация p2p соединений между Leaf-1 и Spine коммутаторами. В режиме конфигурации физического интерфейса настраивается:
1. описание интерфейса;
2. переключение интерфейса в режим L3;
3. отключение переадресации протокола ICMP
4. IP-адрес и включение интерфейса.

```sh
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
```

Конфигурация физического интерфейса, к которому подключен клиент. В режиме конфигурации интерфейса переводим интерфейс в режим L2 и режим "Access", далее назначаем интерфейс в VLAN 100.
```sh
interface ethernet 1/7
  description VPCs
  switchport
  switchport mode access
  switchport access vlan 100
```

Конфигурация Loopback интерфейсов.
```sh
interface loopback0
  description RID
  ip address 10.123.0.11/32

interface loopback1
  description VTEP
  ip address 10.123.0.12/32
```

Конфигурация cli alias для упрощенного способа сохранения конфигурации.
```sh
cli alias name wr copy running-config startup-config
```

Базовая конфигурация остальных коммутаторов настраивается идентичным образом. С базовой конфигурацией можно ознакомиться ниже.

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-2"</em> </summary>

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

cli alias name wr copy running-config startup-config
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-3"</em> </summary>

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

cli alias name wr copy running-config startup-config
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Spine-1"</em> </summary>

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

cli alias name wr copy running-config startup-config
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Spine-2"</em> </summary>

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

cli alias name wr copy running-config startup-config
```
</details> 

##### 2.2 Проверка доступности коммутаторов напрямую.
После настройки основной конфигурации коммутаторов проверим, что коммутаторы "видят" друг друга напрямую, то есть, проверим p2p соединения между Spine и Leaf коммутаторами. Для проверки доступности узлов будем использовать протокол ICMP, а именно команду ping со Spine узлов. Так как к Spine подклчюаются все Leaf коммутаторы, целесообразно проводить проверку доступности именно с этих узлов. Ниже представлен пример доступности Leaf-3 коммутатора со Spine-2.

```sh
Spine-2# ping 10.123.1.11
PING 10.123.1.11 (10.123.1.11): 56 data bytes
64 bytes from 10.123.1.11: icmp_seq=0 ttl=254 time=5.815 ms
64 bytes from 10.123.1.11: icmp_seq=1 ttl=254 time=2.965 ms
64 bytes from 10.123.1.11: icmp_seq=2 ttl=254 time=2.711 ms
64 bytes from 10.123.1.11: icmp_seq=3 ttl=254 time=1.957 ms
64 bytes from 10.123.1.11: icmp_seq=4 ttl=254 time=2.319 ms

--- 10.123.1.11 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 1.957/3.153/5.815 ms
```

Результаты тестирования приведены в таблице 1.

Таблица 1 - Результаты тестирования
|Коммутатор источник |Коммутатор назначения  |Результат тестирования|
|--------------------|-----------------------|----------------------|
|Spine-1             |Leaf-1                 |Успешно               |
|Spine-1             |Leaf-2                 |Успешно               |
|Spine-1             |Leaf-3                 |Успешно               |
|Spine-2             |Leaf-1                 |Успешно               |
|Spine-2             |Leaf-2                 |Успешно               |
|Spine-2             |Leaf-3                 |Успешно               |

### 3. Конфигурация OSPF.
Настроив базовую конфигурацию сетевых устройств далее необходимо обеспечить IP связность всех сетевых узлов на уровне IP. Для этого необходимо настроить маршрутизацию трафика. В рамках домашнего задания, в качестве протокола маршрутизации Underlay сети, будет использоваться протокол OSPF. 

Рекомендуется настраивать маршрутизацию OSPF следующим образом:
1. Использовать p2p на интерфейсах
2. Указывать RID вручную
3. Использовать passive-interface default. Те интерфейсы, которые задействованы в маршрутизации, выводим из passive-interface.
4. Для каждого POD использовать свою зону (Area). 
5. Для активации OSPF на интерфейсах рекомендуется использовать `ip ospf area` вместо `network`.
6. Использовать BFD
7. Избегать использования redistribute.
8. Применять минимально необходимую конфигурацию OSPF.
9. Настраивать OSPF adjacency в GTR.
10. Настраивать аутентификацию.

_Примечание:_ BFD на стенде не поддерживается, потому, настраиваться данный протокол в рамках домашнего задания не будет.

Так как сеть компании ООО "Оринтекс" имеет только 1 POD, то интерфейсы OSPF будут настраиваться в рамках одной Area - Area 0. Схема OSPF представлена на рисунке ниже.
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/Homework2/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D1%81%D0%B5%D1%82%D0%B8%20%D0%A6%D0%9E%D0%94%20(%D0%94%D0%971)%20v1.0-OSPF%20Scheme.png "Схема OSPF ЦОД")

Рассмотрим подробно конфигурацию OSPF одного из коммутаторов ЦОДа.

##### 3.1 Пример конфигурации OSPF на коммутаторе _"Leaf-1"_

Включаем функционал OSPF
```sh
feature ospf
```

Настраиваем цепочку ключей, которая будет использоваться для аутентификации OSPF сессий.
```sh
key chain OSPF
  key 0
    key-string 7 070c285f4d06
```

Настраиваем OSPF Instance и Router ID. Также переводим все интерфейсы в пассивный режим.
```sh
router ospf UNDERLAY
  router-id 10.123.0.11
  passive-interface default
```

В режиме конфигурации физических интерфейсов:
1. Ассоциируем сети с OSPF Instance и Area. 
2. Настраиваем аутентификацию.
3. Выводим интерфейсы, который участвуют в обмене пакетами LSDB, из пассивного режима
4. Переводим интерфейс в режим point-to-point, чтобы исключить процесс выбора DR/BRD.

```sh
interface Ethernet1/1-2
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0
```

В режиме конфигурации Loopback интерфейсов:
1. Ассоциируем сети с OSPF Instance и Area. 
2. Переводим интерфейс в режим point-to-point, чтобы исключить процесс выбора DR/BRD.

```sh
interface loopback0
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback1
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
```

На остальных коммутаторах OSPF настраивается идентичным образом. С конфигурацией OSPF можно ознакомиться ниже.

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-2"</em> </summary>

  ```sh
feature ospf

key chain OSPF
  key 0
    key-string 7 070c285f4d06

router ospf UNDERLAY
  router-id 10.123.0.21
  passive-interface default
  
interface Ethernet1/1-2
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback0
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback1
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0

```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Leaf-3"</em> </summary>

  ```sh
feature ospf

key chain OSPF
  key 0
    key-string 7 070c285f4d06

router ospf UNDERLAY
  router-id 10.123.0.31
  passive-interface default
  
interface Ethernet1/1-2
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback0
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback1
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0

```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Spine-1"</em> </summary>

  ```sh
feature ospf

key chain OSPF
  key 0
    key-string 7 070c285f4d06

router ospf UNDERLAY
  router-id 10.123.0.41
  passive-interface default
  
interface Ethernet1/1-3
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback0
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
```
</details> 

<details> 
<summary> Конфигурация коммутатора <em>"Spine-2"</em> </summary>

  ```sh
feature ospf

key chain OSPF
  key 0
    key-string 7 070c285f4d06

router ospf UNDERLAY
  router-id 10.123.0.51
  passive-interface default
  
interface Ethernet1/1-3
  ip ospf authentication message-digest
  ip ospf authentication key-chain OSPF
  ip ospf network point-to-point
  no ip ospf passive-interface
  ip router ospf UNDERLAY area 0.0.0.0

interface loopback0
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
```
</details> 

##### 3.2 Проверка работоспособности протокола OSPF
После настройки маршрутизации по протоколу OSPF проверим, что у нас установилось соседство и коммутаторы обмениваются маршрутной информацией. В качестве примера, проверим работоспособность маршрутизации OSPF со стороны коммутатора **_Leaf-1_**

1. Проверка установления соседства
 ```sh
Leaf-1# show ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.123.0.41       1 FULL/ -          00:22:50 10.123.1.0      Eth1/1
 10.123.0.51       1 FULL/ -          00:20:33 10.123.1.2      Eth1/2
```
Как мы видим, соседство установлено успешно.

2. Проверка интерфейсов, задействованных в OSPF instance и их состояние.
 ```sh
Leaf-1# show ip ospf interface
 Ethernet1/1 is up, line protocol is up
    IP address 10.123.1.1/31
    Process ID UNDERLAY VRF default, area 0.0.0.0
    Enabled by interface configuration
    State P2P, Network type P2P, cost 40
    Index 1, Transmit delay 1 sec
    1 Neighbors, flooding to 1, adjacent with 1
    Timer intervals: Hello 10, Dead 40, Wait 40, Retransmit 5
      Hello timer due in 00:00:04
    Message-digest authentication, using keychain OSPF (ready)
      Sending SA: Default key id 0, Default algorithm MD5
    Number of opaque link LSAs: 0, checksum sum 0
    Interface ospf state change count: 6
 Ethernet1/2 is up, line protocol is up
    IP address 10.123.1.3/31
    Process ID UNDERLAY VRF default, area 0.0.0.0
    Enabled by interface configuration
    State P2P, Network type P2P, cost 40
    Index 2, Transmit delay 1 sec
    1 Neighbors, flooding to 1, adjacent with 1
    Timer intervals: Hello 10, Dead 40, Wait 40, Retransmit 5
      Hello timer due in 00:00:00
    Message-digest authentication, using keychain OSPF (ready)
      Sending SA: Default key id 0, Default algorithm MD5
    Number of opaque link LSAs: 0, checksum sum 0
    Interface ospf state change count: 6
 loopback0 is up, line protocol is up
    IP address 10.123.0.11/32
    Process ID UNDERLAY VRF default, area 0.0.0.0
    Enabled by interface configuration
    State P2P, Network type P2P, cost 1
    Index 3, Transmit delay 1 sec
    0 Neighbors, flooding to 0, adjacent with 0
    Timer intervals: Hello 10, Dead 40, Wait 40, Retransmit 5
      Hello timer due in 00:00:04
    No authentication
    Number of opaque link LSAs: 0, checksum sum 0
    Interface ospf state change count: 3
 loopback1 is up, line protocol is up
    IP address 10.123.0.12/32
    Process ID UNDERLAY VRF default, area 0.0.0.0
    Enabled by interface configuration
    State P2P, Network type P2P, cost 1
    Index 4, Transmit delay 1 sec
    0 Neighbors, flooding to 0, adjacent with 0
    Timer intervals: Hello 10, Dead 40, Wait 40, Retransmit 5
      Hello timer due in 00:00:02
    No authentication
    Number of opaque link LSAs: 0, checksum sum 0
    Interface ospf state change count: 3
```
Из результатов вывода команды видно, что интерфейсы в активном состоянии и на них используется аутентификация.

3. Просмотр базы данных OSPF.
 ```sh
Leaf-1# show ip ospf database
        OSPF Router with ID (10.123.0.11) (Process ID UNDERLAY VRF default)

                Router Link States (Area 0.0.0.0)

Link ID         ADV Router      Age        Seq#       Checksum Link Count
10.123.0.11     10.123.0.11     1608       0x80000008 0xb7dc   6
10.123.0.21     10.123.0.21     1607       0x80000009 0xfb5f   6
10.123.0.31     10.123.0.31     1603       0x80000008 0x44df   6
10.123.0.41     10.123.0.41     1714       0x80000009 0xde08   7
10.123.0.51     10.123.0.51     1598       0x80000007 0x734b   7
```

4. Просмотр маршрутов OSPF.
 ```sh
Leaf-1# show ip ospf route
 OSPF Process ID UNDERLAY VRF default, Routing Table
  (D) denotes route is directly attached      (R) denotes route is in RIB
  (L) denotes route label is in ULIB          (NHR) denotes next-hop is in RIB
10.123.0.11/32 (intra)(D) area 0.0.0.0
     via 10.123.0.11/Lo0*  , cost 1 distance 110 (NHR)
10.123.0.12/32 (intra)(D) area 0.0.0.0
     via 10.123.0.12/Lo1*  , cost 1 distance 110 (NHR)
10.123.0.21/32 (intra)(R) area 0.0.0.0
     via 10.123.1.0/Eth1/1  , cost 81 distance 110 (NHR)
     via 10.123.1.2/Eth1/2  , cost 81 distance 110 (NHR)
10.123.0.22/32 (intra)(R) area 0.0.0.0
     via 10.123.1.0/Eth1/1  , cost 81 distance 110 (NHR)
     via 10.123.1.2/Eth1/2  , cost 81 distance 110 (NHR)
10.123.0.31/32 (intra)(R) area 0.0.0.0
     via 10.123.1.0/Eth1/1  , cost 81 distance 110 (NHR)
     via 10.123.1.2/Eth1/2  , cost 81 distance 110 (NHR)
10.123.0.32/32 (intra)(R) area 0.0.0.0
     via 10.123.1.0/Eth1/1  , cost 81 distance 110 (NHR)
     via 10.123.1.2/Eth1/2  , cost 81 distance 110 (NHR)
10.123.0.41/32 (intra)(R) area 0.0.0.0
     via 10.123.1.0/Eth1/1  , cost 41 distance 110 (NHR)
10.123.0.51/32 (intra)(R) area 0.0.0.0
     via 10.123.1.2/Eth1/2  , cost 41 distance 110 (NHR)
10.123.1.0/31 (intra)(D) area 0.0.0.0
     via 10.123.1.0/Eth1/1*  , cost 40 distance 110 (NHR)
10.123.1.2/31 (intra)(D) area 0.0.0.0
     via 10.123.1.2/Eth1/2*  , cost 40 distance 110 (NHR)
10.123.1.4/31 (intra)(R) area 0.0.0.0
     via 10.123.1.0/Eth1/1  , cost 80 distance 110 (NHR)
10.123.1.6/31 (intra)(R) area 0.0.0.0
     via 10.123.1.2/Eth1/2  , cost 80 distance 110 (NHR)
10.123.1.8/31 (intra)(R) area 0.0.0.0
     via 10.123.1.0/Eth1/1  , cost 80 distance 110 (NHR)
10.123.1.10/31 (intra)(R) area 0.0.0.0
     via 10.123.1.2/Eth1/2  , cost 80 distance 110 (NHR)
```

Из маршрутной информации видно, что для коммутатора **_Leaf-1_** коммутаторы **_Leaf-2_** и **_Leaf-3_** достижимы через 2 коммутатора _Spine_ одновременно, то есть, у нас используется балансировка трафика с равной стоимостью из-за наличия одинакового пути.

##### 3.3 Проверка доступности сетевых узлов.
После настройки маршрутизации OSPF проверим, что коммутаторы "видят" друг друга. Для проверки доступности сетевых узлов будем использовать протокол ICMP, а именно команду ping и traceroute, запускаемые на коммутаторах ЦОД. Для упрощения тестирования целесообразно проверять доступность Loopback интерфейсов, так как доступность Loopback интерфейсов также подтверждает доступность физических интерфейсов. Ниже представлен пример доступности интерфейса Loopback1, который настроен на коммутаторе Leaf-3. Проверка проводилась с коммутатора Leaf-1.

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
|Leaf-1              |10.123.0.12            |Spine-2               |10.123.0.41        |Успешно               |

_Примечание_: В таблице приведены результаты тестирования только с коммутатора Leaf-1. С остальных коммутаторов результаты также положительные.

С полной версией конфигурации каждого сетевого оборудования можно ознакомиться здесь.
