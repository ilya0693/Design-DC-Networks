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

### Конфигурация VPCS

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

