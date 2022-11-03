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

