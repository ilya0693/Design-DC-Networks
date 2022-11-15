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

Конфигурация коммутатора Leaf-1
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

Конфигурация коммутатора Leaf-2
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
  
</details>

