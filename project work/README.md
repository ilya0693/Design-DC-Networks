# _Проектная работа_

## _Проектирование сетевой фабрики на основне VxLAN EVPN_

_Цель проекта:_ Проектирование VxLAN EVPN фабрики между двумя географически распределенными ЦОД с использованием оборудования производителя Juniper.

_Результат:_ Документ, который можно будет использовать для построения VxLAN EVPN фабрики на базе оборудования Juniper.

_Задачи проекта:_
1.	Требования при проектировании ЦОД.
2.	Описание архитектуры ЦОД
*	Описание Underlay сети и шаблон конфигурации.
*	Описание Overlay сети и шаблон конфигурации.
*	VXLAN/EVPN фабрика в Junos (L2VNI и L3VNI) и шаблон конфигурации.
3.	Организация внешних подключений, стык с вышестоящим провайдером и их резервирование.
4. DCI или растягивание VLAN между несколькими ЦОД
* Обеспечение IP связности между Lo адресами оборудования разных ЦОД в Underlay
* Обеспечение связности между ЦОД в Overlay.
5. Кратко про взаимодействие с оборудованием безопасности
6. Общие сведения по сети ЦОД.

### _Введение_

В данном проекте отражены элементы Высокоуровневого (High Level Design) и Низкоуровневого дизайна (Low Level Design) сети передачи данных Центра Обработки Данных (ЦОД). Описывается рекомендуемая архитектура и шаблоны конфигурации для построения ЦОД на базе оборудования Juniper Networks. Данный проект является попыткой создать документ, который можно было бы использовать при проектировании сети ЦОД на базе оборудования Juniper Networks, а потому, некоторые моменты могут быть упущены или не учтены.

### _1. Требования при проектировании ЦОД_

Рассмотрим ситуацию, когда необходимо создать инфраструктуру ЦОД на двух площадках, находящихся в городе Москва и городе Ставрополь, а к проектированию ЦОД предъявляют следующие требования:

* Требования по использованию архитектуры фабрики коммутации с распределённой плоскостью управления;
* Требования по расположению L3-маршрутизации ЦОД на граничных маршрутизаторах;
* Соединение ЦОД через вышестоящего провайдера;
* Общие требования по отказоустойчивости решения на каждом уровне ЦОД;
* Требования к масштабированию ЦОД по количеству коммутаторов и пропускной способности;
* Требования по балансировке и оптимальному прохождению трафика в ЦОД.

### _2. Описание архитектуры ЦОД_
На рисунке ниже представлена архитектура ЦОД, которая разбита на на следующие структурные блоки:
* Блок фабрики коммутации;
* Блок граничных маршрутизаторов и взаимодействия с удалёнными площадками;
* Блок межсетевых экранов;
* Блок подключения серверов;
* Блок подключения устройств L4-L7 сервисов.

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/project%20work/%D0%90%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0.drawio.png "Архитектура ЦОД")

В данном проекте блоки межсетевых экранов и устройств L4-L7 сервисов подробно рассматриваться не будут, остальные блоки либо детально (блоки фабрики коммутации и граничных маршрутизаторов), либо частично (блок подключения серверов) будут рассмотрены.

В силу требований по расположению VXLAN L3 GW функционала на BR, BR маршрутизаторы участвуют сразу в блоке фабрики коммутации и блоке граничных маршрутизаторов.

На рисунке ниже представлена физическая топология ЦОДа в г. Москва. Предполагается, что в г. Ставрополь физическая топология ЦОДа будет проектироваться идентично, но, в рамках данного проекта физическая топология упрощена, так как предполагается, что данный ЦОД будет масштабироваться постепенно.

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/project%20work/%D0%A4%D0%B8%D0%B7%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%B0%D1%8F%20%D1%81%D1%85%D0%B5%D0%BC%D0%B0.drawio.png "Физическая схема сети ЦОД")

На данной схеме все SPINE и LEAF соединены агрерированными интерфейсами, каждый из которых состоит из двух портов. BR соединены со Spine обычными неагрегированными портами.

BR подключаются портами к вышестоящему провайдеру. Через вышестоящего провайдера подключаются клиентские VRF, а также через неё выполняется соединение Underlay для связности между несколькими ЦОД (Data Center Interconnect – DCI). BR дополнительно соединены друг с другом прямым интерфейсом. Этот интерфейс не является частью EVPN-фабрики, он используется для резервирования через MPLS L3VPN-стыков с вышестоящим провайдером.

К LEAF-коммутаторам подключаются сервера. К BLEAF подключаются сервера и блок безопасности, состоящий из межсетерых экранов и VPN-шлюзов. Фактически, с точки зрения
шаблонов конфигурации, BLEAF не отличаются от LEAF, поэтому далее они будут называться просто LEAF-коммутаторами.

Для построения VxLAN EVPN фабрики используются следующие технологии:

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

### _2.1. Описание Underlay сети и шаблон конфигурации_

Основным условием для сигнализации VXLAN-туннелей и обмена EVPN-маршрутами является IP-связность между всеми VTEP (точками терминации VXLAN), а именно между loopback-адресами коммутаторов ЦОД. В контексте ЦОД для организации IP-связности рекомендуется использовать архитектуру IP Fabric (RFC7938, Use of BGP for Routing in Large-Scale Data Centers), которая основывается на Clos-топологии (также известной как Spine&Leaf) и использовании протокола eBGP для сигнализации маршрутной информации. Также можно использовать протоколы семейства IGP (ISIS или OSPF) для распространения маршрутов loopback, но выбор в пользу eBGP для Underlay был сделан из-за гибких возможностей маркировки и фильтрации BGP-маршрутов. Для обеспечения балансировки в IP фабрике используется функция eBGP Multipath с включенной опцией multiple-as на основе длины атрибута AS_PATH. Таким образом, трафик между парой LEAF-коммутаторов будет всегда балансироваться через оба SPINE-коммутатора. На рисунках ниже представлена топология Underlay для обоих ЦОД.

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/project%20work/Underlay.drawio.png "Underlay топология")
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/project%20work/Underlay%20%D0%9C%D0%BE%D1%81%D0%BA%D0%B2%D0%B0.png "Underlay топология ЦОД г. Москва")
![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/project%20work/Underlay%20%D0%A1%D1%82%D0%B0%D0%B2%D1%80%D0%BE%D0%BF%D0%BE%D0%BB%D1%8C.png "Underlay топология ЦОД г. Ставрополь")

Шаблон конфигурации для Underlay топологии представлен ниже
<details>
<summary> Шаблон конфигурации d77-leaf-r11-sw01 </summary>

 ```sh
/* добавляем физические интерфейсы в LAG группы */
set interfaces xe-0/0/8 description d77-spine-r01-sw01
set interfaces xe-0/0/8 ether-options 802.3ad ae1
set interfaces xe-0/0/9 description d77-spine-r01-sw01
set interfaces xe-0/0/9 ether-options 802.3ad ae1
set interfaces xe-0/0/10 description d77-spine-r01-sw02
set interfaces xe-0/0/10 ether-options 802.3ad ae2
set interfaces xe-0/0/11 description d77-spine-r01-sw02
set interfaces xe-0/0/11 ether-options 802.3ad ae2
!
/* конфигурация агрегированных LAG интерфейсов в сторону SPINE */
set interfaces ae1 description d77-spine-r01-sw01
set interfaces ae1 mtu 9216
set interfaces ae1 aggregated-ether-options lacp active
set interfaces ae1 aggregated-ether-options lacp periodic fast
set interfaces ae1 unit 0 family inet address 10.77.1.1/31
set interfaces ae2 description d77-spine-r01-sw02
set interfaces ae2 mtu 9216
set interfaces ae2 aggregated-ether-options lacp active
set interfaces ae2 aggregated-ether-options lacp periodic fast
set interfaces ae2 unit 0 family inet address 10.77.1.129/31
!
set interfaces lo0 unit 0 family inet address 10.77.0.11/32
!
set protocols bgp log-updown
set protocols bgp group UNDERLAY-IPFABRIC type external
set protocols bgp group UNDERLAY-IPFABRIC mtu-discovery
set protocols bgp group UNDERLAY-IPFABRIC import POL-BGP-IPFABRIC-IMPORT /* политики маршрутизации BGP UNDERLAY */
set protocols bgp group UNDERLAY-IPFABRIC export POL-BGP-IPFABRIC-EXPORT /* политики маршрутизации BGP UNDERLAY */
set protocols bgp group UNDERLAY-IPFABRIC local-as 4207700011
set protocols bgp group UNDERLAY-IPFABRIC multipath multiple-as /* Балансировка BGP multipath */
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection minimum-interval 1000
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection multiplier 3
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection session-mode automatic
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.0 description d77-spine-r01-sw01_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.0 peer-as 4207700001
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.128 description d77-spine-r01-sw02_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.128 peer-as 4207700002
!
set routing-options router-id 10.77.0.11
set routing-options forwarding-table export POL-PFE-ECMP /* Применение политики инсталляции префиксов в FIB для балансировки трафика */
set routing-options forwarding-table ecmp-fast-reroute
!
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK from protocol direct
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK from route-filter 10.77.0.11/32 exact
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK then accept
!
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-LPBKS from route-filter 10.77.0.0/24 orlonger
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-LPBKS then accept
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-REJECT then reject
!
set policy-options policy-statement POL-PFE-ECMP then load-balance per-packet

Остальные Leaf коммутаторы настраиваются идентичным образом.
```
</details>

<details>
<summary> Шаблон конфигурации d77-spine-r01-sw01 </summary>

 ```sh
/* добавляем физические интерфейсы в LAG группы */
set interfaces xe-0/0/0 description d77-leaf-r11-sw01
set interfaces xe-0/0/0 ether-options 802.3ad ae1
set interfaces xe-0/0/1 description d77-spine-r11-sw01
set interfaces xe-0/0/1 ether-options 802.3ad ae1
set interfaces xe-0/0/2 description d77-leaf-r12-sw02
set interfaces xe-0/0/2 ether-options 802.3ad ae2
set interfaces xe-0/0/3 description d77-spine-r12-sw02
set interfaces xe-0/0/3 ether-options 802.3ad ae2
.....
!
/* конфигурация агрегированных LAG интерфейсов в сторону SPINE */
set interfaces ae1 description d77-leaf-r11-sw01
set interfaces ae1 mtu 9216
set interfaces ae1 aggregated-ether-options lacp active
set interfaces ae1 aggregated-ether-options lacp periodic fast
set interfaces ae1 unit 0 family inet address 10.77.1.0/31
set interfaces ae2 description d77-leaf-r12-sw02
set interfaces ae2 mtu 9216
set interfaces ae2 aggregated-ether-options lacp active
set interfaces ae2 aggregated-ether-options lacp periodic fast
set interfaces ae2 unit 0 family inet address 10.77.1.2/31
.....
!
/* конфигурация интерфейсов в сторону BR */ 
set interfaces xe-0/0/8 description d77-br-r02-br01
set interfaces xe-0/0/8 mtu 9216
set interfaces xe-0/0/8 unit 0 family inet address 10.77.1.121/31
set interfaces xe-0/0/9 description d77-br-r02-br02
set interfaces xe-0/0/9 mtu 9216
set interfaces xe-0/0/9 unit 0 family inet address 10.77.1.123/31
!
set interfaces lo0 unit 0 family inet address 10.77.0.1/32
!
set protocols bgp log-updown
set protocols bgp group UNDERLAY-IPFABRIC type external
set protocols bgp group UNDERLAY-IPFABRIC mtu-discovery
set protocols bgp group UNDERLAY-IPFABRIC import POL-BGP-IPFABRIC-IMPORT /* политики маршрутизации BGP UNDERLAY */
set protocols bgp group UNDERLAY-IPFABRIC export POL-BGP-IPFABRIC-EXPORT /* политики маршрутизации BGP UNDERLAY */
set protocols bgp group UNDERLAY-IPFABRIC local-as 4207700001
set protocols bgp group UNDERLAY-IPFABRIC multipath multiple-as /* Балансировка BGP multipath */
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection minimum-interval 1000
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection multiplier 3
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection session-mode automatic
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.1 description d77-leaf-r11-sw01_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.1 peer-as 4207700011
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.129 description d77-leaf-r12-sw02_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.129 peer-as 4207700012
.....
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.120 description d77-br-r02-br01_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.120 peer-as 4207700004
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.122 description d77-br-r02-br02_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.122 peer-as 4207700005
!
set routing-options router-id 10.77.0.1
set routing-options forwarding-table export POL-PFE-ECMP /* Применение политики инсталляции префиксов в FIB для балансировки трафика */
set routing-options forwarding-table ecmp-fast-reroute
!
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK from protocol direct
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK from route-filter 10.77.0.1/32 exact
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK then accept
!
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-LPBKS from route-filter 10.77.0.0/24 orlonger
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-LPBKS then accept
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-REJECT then reject
!
set policy-options policy-statement POL-PFE-ECMP then load-balance per-packet

Коммутатор d77-spine-r01-sw02 настраивается идентичным образом.
```
</details>

<details>
<summary> Шаблон конфигурации d77-br-r02-br01 </summary>

 ```sh
/* конфигурация интерфейсов в сторону SPINE */
set interfaces xe-0/0/0 description d77-spine-r01-sw01
set interfaces xe-0/0/0 mtu 9216
set interfaces xe-0/0/0 unit 0 family inet address 10.77.1.120/31
set interfaces xe-0/0/1 description d77-spine-r01-sw02
set interfaces xe-0/0/1 mtu 9216
set interfaces xe-0/0/1 unit 0 family inet address 10.77.1.248/31
.....
!
set interfaces lo0 unit 0 family inet address 10.77.0.4/32
!
set protocols bgp log-updown
set protocols bgp group UNDERLAY-IPFABRIC type external
set protocols bgp group UNDERLAY-IPFABRIC mtu-discovery
set protocols bgp group UNDERLAY-IPFABRIC import POL-BGP-IPFABRIC-IMPORT /* политики маршрутизации BGP UNDERLAY */
set protocols bgp group UNDERLAY-IPFABRIC export POL-BGP-IPFABRIC-EXPORT /* политики маршрутизации BGP UNDERLAY */
set protocols bgp group UNDERLAY-IPFABRIC local-as 4207700004
set protocols bgp group UNDERLAY-IPFABRIC multipath multiple-as /* Балансировка BGP multipath */
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection minimum-interval 1000
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection multiplier 3
set protocols bgp group UNDERLAY-IPFABRIC bfd-liveness-detection session-mode automatic
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.121 description d77-spine-r01-sw01_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.121 peer-as 4207700001
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.249 description d77-spine-r01-sw02_ebgp
set protocols bgp group UNDERLAY-IPFABRIC neighbor 10.77.1.249 peer-as 4207700002
!
set routing-options router-id 10.77.0.1
set routing-options forwarding-table export POL-PFE-ECMP /* Применение политики инсталляции префиксов в FIB для балансировки трафика */
set routing-options forwarding-table ecmp-fast-reroute
!
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK from protocol direct
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK from route-filter 10.77.0.1/32 exact
set policy-options policy-statement POL-BGP-IPFABRIC-EXPORT term T-LPBK then accept
!
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-LPBKS from route-filter 10.77.0.0/24 orlonger
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-LPBKS then accept
set policy-options policy-statement POL-BGP-IPFABRIC-IMPORT term T-REJECT then reject
!
set policy-options policy-statement POL-PFE-ECMP then load-balance per-packet

Маршрутизатор d77-br-r02-br02 настраивается идентичным образом.
```
</details>

### _2.2. Описание Overlay сети и шаблон конфигурации_

Overlay-топология представляет собой набор VXLAN-туннелей точка-точка между всеми коммутаторами ЦОД поверх Underlay-сети. При этом за распространение информации о MAC-
адресах отвечает протокол MP-BGP (RFC4760, Multiprotocol Extensions for BGP-4) с адресным семейством EVPN (AFI 25 / SAFI 70).

В силу требования о расположении L3GW серверных VLAN на BR-маршрутизаторах, LEAF выполняют роль L2GW, а SPINE роль осуществляют передачу IP-пакетов между LEAF-коммутаторами и маршрутизаторами BR, не участвуя в VXLAN.

Конфигурация EVPN/VXLAN в JUNOS соответствует VLAN-Aware Bundle Service из RFC7432 и draft-ietf-bess-evpn-overlay, когда для каждого из VNI, относящихся к MAC-VRF (EVI), внутри EVI создается отдельная Bridge-Domain таблица MAC-адресов.

Для полноценной работы IP Fabric с EVPN/VXLAN маршруты EVPN должны быть распространены между всеми VTEP:
* LEAF-коммутаторами, к которым подключаются сервера ЦОД;
* LEAF-коммутаторами, к которым подключается другое сетевое оборудование, участвующее в широковещательном домене VXLAN-сегмента. Например, коммутаторы, не поддерживающие EVPN/VXLAN, или межсетевой экран (Firewall);
* BR-маршрутизаторами, выступающими в роли VXLAN L3 GW.

Обмен EVPN-маршрутами между всеми точками VTEP осуществляется по схеме с использованием Internal BGP Route Reflectors (BGR RR, RFC4456) внутри каждого из ЦОД. Роли
Route Reflectors выполняют SPINE-коммутаторы как показано на схеме ниже:

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/project%20work/Overlay.drawio.png "Overlay топология ЦОД")

Выбор расположения функционала iBGP RR на SPINE обусловлен важностью данного функционала и тем, что на этих коммутаторах, в отличие от BR, редко производится
изменение конфигурации в процессе эксплуатации, поэтому вероятность человеческой ошибки на них минимальна.

Рекомендуется использовать полносвязный набор eBGP-сессий между RR/SPINE-коммутаторами нескольких ЦОД по следующим причинам:
* Использование BGP RR на уровне DCI превращает дизайн Overlay BGP в схему c Hierarchical Route Reflectors, что значительно усложняет дизайн ЦОД;
* По сравнению с добавлением нового ToR-коммутатора добавление нового ЦОД является крайне редким явлением.

Шаблон конфигурации для Overlay топологии представлен ниже
<details>
<summary> Шаблон конфигурации d77-leaf-r11-sw01 </summary>

 ```sh
set protocols bgp group OVERLAY-EVPN type internal
set protocols bgp group OVERLAY-EVPN local-address 10.77.0.11
set protocols bgp group OVERLAY-EVPN import POL-BGP-REJECT-RGW-IMPORT
set protocols bgp group OVERLAY-EVPN export POL-MARK-LEAF
set protocols bgp group OVERLAY-EVPN family evpn signaling
set protocols bgp group OVERLAY-EVPN vpn-apply-export
set protocols bgp group OVERLAY-EVPN multipath
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection minimum-interval 1000
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection multiplier 3
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection session-mode automatic
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.1 description d77-spine-r01-sw01_ibgp
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.2 description d77-spine-r01-sw02_ibgp
!
set routing-options router-id 10.77.0.11
set routing-options autonomous-system 65277
!
/* Запрет на импорт EVPN Type 1,2,3 маршрутов от удаленных BR */
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPORT term T-REJECT-REMOTE-GW from family evpn
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPORT term T-REJECT-REMOTE-GW from as-path ASP-LENGTH1MORE
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPORT term T-REJECT-REMOTE-GW from community CT-GW
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPORT term T-REJECT-REMOTE-GW from nlri-route-type 1
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPORT term T-REJECT-REMOTE-GW from nlri-route-type 2
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPORT term T-REJECT-REMOTE-GW from nlri-route-type 3
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPOR term T-REJECT-REMOTE-GW then reject
set policy-options policy-statement POL-BGP-REJECT-RGW-IMPOR term T-ACCEPT-ALL then accept
!
/* Маркируем анонсируемые EVPN маршруты community означающими принадлежность к определенному ЦОД и типу LEAF */
set policy-options policy-statement POL-MARK-LEAF term 10 then community add CT-LEAF
set policy-options policy-statement POL-MARK-LEAF term 10 then community add CT-65277
!
set policy-options community CT-65277 members target:65000:65277
set policy-options community CT-LEAF members target:65000:2
set policy-options community CT-GW members target:65000:1
set policy-options as-path ASP-LENGTH1MORE ".{1,}"

Остальные Leaf коммутаторы настраиваются идентичным образом.
```
</details>

<details>
<summary> Шаблон конфигурации d77-spine-r01-sw01 </summary>

 ```sh
set protocols bgp group OVERLAY-EVPN type internal
set protocols bgp group OVERLAY-EVPN local-address 10.77.0.1
set protocols bgp group OVERLAY-EVPN family evpn signaling
set protocols bgp group OVERLAY-EVPN multipath
set protocols bgp group OVERLAY-EVPN cluster 10.77.0.1
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection minimum-interval 1000
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection multiplier 3
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection session-mode automatic
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.3 description d77-br-r02-br01_ibgp
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.4 description d77-br-r02-br02_ibgp
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.11 description d77-leaf-r11-sw01_ibgp
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.12 description d77-leaf-r12-sw02_ibgp
.....
!
/* Прямая BGP сессия между RR (SPINE) внутри ЦОД, они не являются клиентами друг друга */
set protocols bgp group OVERLAY-EVPN-RR-RR type internal
set protocols bgp group OVERLAY-EVPN-RR-RR local-address 10.77.0.1
set protocols bgp group OVERLAY-EVPN-RR-RR family evpn signaling
set protocols bgp group OVERLAY-EVPN-RR-RR bfd-liveness-detection minimum-interval 1000
set protocols bgp group OVERLAY-EVPN-RR-RR bfd-liveness-detection multiplier 3
set protocols bgp group OVERLAY-EVPN-RR-RR bfd-liveness-detection session-mode automatic
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.2 description d77-spine-r01-sw02_rr
!
set routing-options router-id 10.77.0.1
set routing-options autonomous-system 65277

Коммутатор d77-spine-r01-sw02 настраивается идентичным образом.
```
</details>

<details>
<summary> Шаблон конфигурации d77-br-r02-br01 </summary>

 ```sh
set protocols bgp group OVERLAY-EVPN type internal
set protocols bgp group OVERLAY-EVPN local-address 10.77.0.4
set protocols bgp group OVERLAY-EVPN family evpn signaling
set protocols bgp group OVERLAY-EVPN export POL-MARK-GW
set protocols bgp group OVERLAY-EVPN vpn-apply-export
set protocols bgp group OVERLAY-EVPN multipath
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection minimum-interval 1000
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection multiplier 3
set protocols bgp group OVERLAY-EVPN bfd-liveness-detection session-mode automatic
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.1 description d77-spine-r01-sw01_ibgp
set protocols bgp group OVERLAY-EVPN neighbor 10.77.0.2 description d77-spine-r01-sw02_ibgp
!
/* Маркируем анонсируемые EVPN маршруты community означающими принадлежность к определенному ЦОД и типу GW */
set policy-options policy-statement POL-MARK-GW term 10 then community add CT-GW
set policy-options policy-statement POL-MARK-GW term 10 then community add CT-65277
!
set policy-options community CT-65277 members target:65000:65277
set policy-options community CT-GW members target:65000:1
!
set routing-options router-id 10.77.0.4
set routing-options autonomous-system 65277

Маршрутизатор d77-br-r02-br02 настраивается идентичным образом.
```
</details>

### _2.3. VXLAN/EVPN фабрики в Junos (L2VNI и L3VNI) и шаблон конфигурации_

В данном разделе описана конфигурация EVPN сервиса на LEAF- и BR-устройствах в рамках функционала L2GW и L3GW. Рассмотрим сначала описание и реализацию функционала L2GW.

L2GW предполагает инкапсуляцию и декапсуляцию VXLAN-трафика, работает с L2- фреймами, получаемыми от подключённого к EVPN-фабрике оборудования. На L2GW устройствах настраивается соответствие между значениями VLAN ID и VNI. Генерация значений RT VNI и добавление их в export/import EVPN-политиках выполняется автоматически с помощью функции Auto RT. Значение генерируются на основе Overlay ASN и VNI. 

Отдельное значение, общее для всех коммутаторов, используется для ES RT. Данное extended community используется при анонсе и импорте Autodiscovery route per EVPN instance Type 1-маршрута, который фактически описывает порты коммутаторов. В рамках данного проекта предлагается использовать значение ES RT target:65000:9999 для
конфигурации EVPN/VXLAN.

Необходимо отметить, что данные значения RT передаются только вместе с EVPN-маршрутами ЦОД по явно настроенным BGP-сессиям и не имеют возможность пересечься с
BGP-маршрутами других address family на маршрутизаторах BR.

Шаблон конфигурации для EVP/VXLAN (L2GW) представлен ниже
<details>
<summary> Шаблон конфигурации d77-leaf-r11-sw01 </summary>

 ```sh
set protocols evpn encapsulation vxlan
set protocols evpn extended-vni-list all /* Все vni/vlan относим к единтвенному EVPN instance */
!
set switch-options vtep-source-interface lo0.0 /* Явно указывается источник VXLAN туннелей */
set switch-options vrf-target auto /* Включение автоматической генерации RT для VNI */
set switch-options vrf-target target:65000:9999 /* Добавление единого RT ES на экспорт EVPN Type 1 маршрутов. Данный RT ES одинаковый во всех ЦОД в целях корректной работы DCI для ESI-LAG */
set switch-options route-distinguisher 10.77.0.11:1 /* Уникальный RD на каждом EVPN/VXLAN PE */
!
set vlans v100 description PROD
set vlans v100 vlan-id 100
set vlans v100 vxlan vni 770100
set vlans v200 description DEV
set vlans v200 vlan-id 200
set vlans v200 vxlan vni 770200

Остальные Leaf коммутаторы настраиваются идентичным образом.
```
</details>

Рассмотрим описание и реализацию функционала L3GW.

На L3GW выполняется маршрутизация трафика между разными VLAN/VXLAN. Модель, в которой маршрутизация выполняется централизованно на BR, называется CRB Centrally-
Routed Bridging. Эта модель и используется в рамках нашего проекта. Альтернативным вариантом является маршрутизация сразу на LEAF-устройствах, такая модель называется ERB Edge-routed Bridging.
В данном дизайне резервирование шлюза осуществляется средствами EVPN. Существуют два способа резервирования шлюза:
* Резервирование шлюза методом VGA
* Резервирование шлюза по умолчанию методом IP Anycast

В данном проекте для резервирования шлюза выбран метод VGA, так как данный метод обладает возможностью балансировки трафика per-flow. 

Шаблон конфигурации для EVP/VXLAN на маршрутизаторах BR представлен ниже. В данном шаблоне рассматривается конфигурация, как L2GW, так и L3GW, а также резервирование методом VGA.

<details>
<summary> Шаблон конфигурации d77-br-r02-br01 </summary>

 ```sh
set routing-instances RI-VS-d77VSWITCH instance-type virtual-switch
set routing-instances RI-VS-d77VSWITCH vtep-source-interface lo0.0
set routing-instances RI-VS-d77VSWITCH route-distinguisher 10.77.0.4:1
set routing-instances RI-VS-d77VSWITCH vrf-target auto /* Включение автоматической генерации RT для VNI */
set routing-instances RI-VS-d77VSWITCH vrf-target target:65000:9999 /* Добавление единого RT ES на экспорт EVPN Type 1 маршрутов. */
set routing-instances RI-VS-d77VSWITCH protocols evpn encapsulation vxlan
set routing-instances RI-VS-d77VSWITCH protocols evpn extended-vni-list all /* Все vni/vlan относим к единтвенному EVPN instance */
set routing-instances RI-VS-d77VSWITCH protocols evpn default-gateway no-gateway-community /* Так как используются VGA или Anycast IP, то gateway-community не нужно */
!
/* Конфигурация BD для каждого VNI */
set routing-instances RI-VS-d77VSWITCH bridge-domains v100 description PROD
set routing-instances RI-VS-d77VSWITCH bridge-domains v100 vlan-id 100
set routing-instances RI-VS-d77VSWITCH bridge-domains v100 routing-interface irb.100 /* Связываем VLAN/VXLAN c L3 интерфейсом IRB, L3GW */
set routing-instances RI-VS-d77VSWITCH bridge-domains v100 vxlan vni 770100
set routing-instances RI-VS-d77VSWITCH bridge-domains v200 description DEV
set routing-instances RI-VS-d77VSWITCH bridge-domains v200 vlan-id 200
set routing-instances RI-VS-d77VSWITCH bridge-domains v200 routing-interface irb.200 /* Связываем VLAN/VXLAN c L3 интерфейсом IRB, L3GW */
set routing-instances RI-VS-d77VSWITCH bridge-domains v200 vxlan vni 770200
!
/* Конфигурация L3VPN */
set routing-instances RI-VRF-10 instance-type vrf
set routing-instances RI-VRF-10 interface irb.100
set routing-instances RI-VRF-10 interface irb.200
set routing-instances RI-VRF-10 route-distinguisher 10.77.0.4:10
set routing-instances RI-VRF-10 vrf-target 65277:10
set routing-instances RI-VRF-10 vrf-table-label
!
/* L3 конфигурация IRB, L3GW */
set interfaces irb mtu 9216
set interfaces irb unit 100 apply-groups GR-IRB-INTF
set interfaces irb unit 100 virtual-gateway-accept-data
set interfaces irb unit 100 family inet address 10.77.100.2 virtual-gateway-address 10.77.100.1 /* Уникальный адрес BR + Виртуальный VGA адрес BR, шлюз */
set interfaces irb unit 200 apply-groups GR-IRB-INTF
set interfaces irb unit 200 virtual-gateway-accept-data
set interfaces irb unit 200 family inet address 10.77.200.2 virtual-gateway-address 10.77.200.1
!
set groups GR-IRB-INTF interfaces <irb*> unit <*> family inet mtu 9152
set groups GR-IRB-INTF interfaces <irb*> unit <*> virtual-gateway-v4-mac 00:00:5e:00:aa:bb
set groups GR-IRB-INTF interfaces <irb*> unit <*> mac 00:00:5e:00:aa:bb
 
Маршрутизатор d77-br-r02-br02 настраивается идентичным образом.
```
</details>

### _3. Организация внешних подключений, стык с вышестоящим провайдером и их резервирование_
На BR выполняется подключение сервисов ЦОД к сети вышестоящего оператора. Физически каждый BR подключается одним портом к PE-маршрутизаторам, BR1 к одному PE, BR2 к другому. Сервисы могут подключаться как на уровне L2, так и на уровне L3. Основным вариантом является L3-подключение.

Для подключения клиентского EVPN/VXLAN на уровне L2 к L2VPN/VPLS-сервису на сети ISP используется следующая конфигурация:
<details>
<summary> Шаблон конфигурации d77-br-r02-br01 </summary>

 ```sh
set routing-instances RI-VS-d77VSWITCH interface xe-0/0/3.1011 /* Добавление логического интерфейса L2 стыка с PE в EVI Tenant */
!
/* Конфигурация VLAN/VXLAN стыкуемого сервиса */
set routing-instances RI-VS-d77VSWITCH bridge-domains v1011 description CLIENT
set routing-instances RI-VS-d77VSWITCH bridge-domains v1011 vlan-id 1011
set routing-instances RI-VS-d77VSWITCH bridge-domains v1011 vxlan vni 771011
!
/* Конфигурация стыковочного интерфейса */
set interfaces xe-0/0/3 unit 1011 family bridge interface-mode trunk
set interfaces xe-0/0/3 unit 1011 family bridge vlan-id-list 1011
 
```
</details>
 
Т.е. на BR конфигурируется связка клиентского VLAN/VXLAN из ЦОД с логическим интерфейсом стыка c PE, предназначенным для L2-сервисов. Этот логический интефрейс
настраивается как trunk с соответствующим списком VLAN. Как видно из конфигурации, один логический интефрейс стыка используется для множества сервисов.

Сервисы на уровне L3-маршрутизации рекомендуется стыковать с использованием протокола eBGP на PE-CE, где в роли CE выступает BR. На BR в клиентском VRF создается eBGP-сессия с вышестоящим PE, по которой анонсируются префиксы IRB-интерфейсов данного VRF и принимаются маршруты соответствующего сервиса из сети вышестоящего провайдера, что соответствует Inter-AS MPLS VPN Option-A-стыку L3VPN. 

Для подключения клиентского EVPN/VXLAN на уровне L3 используется следующий шаблон конфигурации:
<details>
<summary> Шаблон конфигурации d77-br-r02-br01 </summary>

 ```sh
set routing-instances RI-VRF-100 instance-type vrf
set routing-instances RI-VRF-100 interface irb.500
set routing-instances RI-VRF-100 interface xe-0/0/3.1050
set routing-instances RI-VRF-100 route-distinguisher 10.77.0.4:100
set routing-instances RI-VRF-100 vrf-target 65277:100
set routing-instances RI-VRF-100 vrf-table-label
!
/* Конфигурация eBGP сессии OPT-A стыка PE-CE */
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP hold-time 10
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP export POL-VRF-100-EBGP-EXPORT
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP peer-as 12800
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP local-as 65277
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP bfd-liveness-detection minimum-interval 250
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP bfd-liveness-detection multiplier 3
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP bfd-liveness-detection session-mode automatic
set routing-instances RI-VRF-100 protocols bgp group GRP-PECE-ISP neighbor 198.211.120.1
!
/* Конфигурация стыковочного интерфейса */
set interfaces xe-0/0/3 unit 1050 vlan-id 1050
set interfaces xe-0/0/3 unit 1050 family inet address 198.211.120.2/30
!
set policy-options policy-statement POL-VRF-100-EBGP-EXPORT term 1 from route-filter 200.0.0.0/24 orlonger
set policy-options policy-statement POL-VRF-100-EBGP-EXPORT term 1 then accept
set policy-options policy-statement POL-VRF-100-EBGP-EXPORT then reject
 
```
</details>

Для резервирования L3 сервисов используется BGP VPNv4/6-сессии и MPLS между BR для передачи префиксов, полученных от PE. Дело в том, что серверам и LEAF-коммутаторам не доступна информация о наличии связности между BR (он же CE) и PE. Поэтому, в случае отказа стыка CE - PE, возникнет black hole и трафик, поступающий на CE с отказавшим стыком и назначенный к адресам доступным через PE, будет отбрасываться. На рисунке ниже представлен пример отказа стыка BR2 и PE2

![alt-текст](https://github.com/ilya0693/Design-DC-Networks/blob/main/project%20work/Reservation%20of%20channels.drawio.png "Пример отказа стыка BR2 и PE2")

Для резервирования L3 сервисов используется следующий шаблон конфигурации:
<details>
<summary> Шаблон конфигурации d77-br-r02-br01 </summary>

 ```sh
/* BGP группа для VPNv4 iBGP сессии */
set protocols bgp group BR-MPLSVPN type internal
set protocols bgp group BR-MPLSVPN family inet-vpn unicast
set protocols bgp group BR-MPLSVPN neighbor 10.77.2.1 local-address 10.77.2.2
!
/* Через rib-group интерфейс стыка между BR помещается в inet.3 */
set routing-options interface-routes rib-group inet RIBGRP-INTRF-TO-INET3
set routing-options rib-groups RIBGRP-INTRF-TO-INET3 import-rib inet.0
set routing-options rib-groups RIBGRP-INTRF-TO-INET3 import-rib inet.3
set routing-options rib-groups RIBGRP-INTRF-TO-INET3 import-policy POL-RIBGRP-INTRF-TO-INET3
!
/* Конфигурация стыковочного интерфейса между BR */
set interfaces xe-0/0/2 description d77-br-r02-br02
set interfaces xe-0/0/2 unit 0 family inet address 10.77.2.2/30
set interfaces xe-0/0/2 unit 0 family mpls /* Включение обработки MPLS инкапсуляции на интерфейсе стыка между BR */
!
set policy-options policy-statement POL-RIBGRP-INTRF-TO-INET3 term 10 from interface xe-0/0/2
set policy-options policy-statement POL-RIBGRP-INTRF-TO-INET3 term 10 then accept
set policy-options policy-statement POL-RIBGRP-INTRF-TO-INET3 term 20 from interface lo0.0
set policy-options policy-statement POL-RIBGRP-INTRF-TO-INET3 term 20 then accept
set policy-options policy-statement POL-RIBGRP-INTRF-TO-INET3 term 100 to rib inet.0
set policy-options policy-statement POL-RIBGRP-INTRF-TO-INET3 term 100 then accept
set policy-options policy-statement POL-RIBGRP-INTRF-TO-INET3 term 1000 then reject
 
```
</details>

### _4. DCI или растягивание VLAN между несколькими ЦОД_
Для подключения серверов разных ЦОД в одну и ту же L2-среду и выполнения миграции виртуальных машин между ЦОД реализуется DC Interconnect. Реализацию DCI можно условно
разделить на следующие части:
* Создание IP-связности между Lo0-адресами оборудования разных ЦОД в Underlay. Это достигается конфигурацией BR, на которых создаются eBGP IPv4-сессии с сетью вышестоящего провайдера. По этим сессиям анонсируются адреса Lo0-интерфейсов локального ЦОД и принимаются адреса Lo0-интерфейсов удалённых ЦОД.
* Сигнализация EVPN – обмен BGP EVPN-маршрутами между ЦОД в Overlay. Это достигается конфигурацией full mesh BGP EVPN-сессий между Route Reflectors (SPINE) разных ЦОД.
* Настройка параметров EVPN для растянутых VXLAN. Достигается конфигурацией LEAF.

#### _4.1. Обеспечение IP связности между Lo адресами оборудования разных ЦОД в Underlay_

Для создания DCI первым шагом обеспечивается IP-связность между Lo0-адресами оборудования разных ЦОД в Underlay. Устанавливаются eBGP IPv4-сессии между GRT ЦОД BR и транспортным VRF для DCI, созданным в сети провайдера. Шаблон конфигурации ниже:
<details>
<summary> Шаблон конфигурации d77-br-r02-br01 </summary>

 ```sh
/* BGP группа для DCI Underlay */
set protocols bgp group DCI type external
set protocols bgp group DCI family inet unicast
set protocols bgp group DCI export POL-DCI-EXPORT /* Политика анонса префиксов Lo0 интерфейсов локального ЦОД */
set protocols bgp group DCI neighbor 201.199.195.1
set protocols bgp group DCI peer-as 12800
set protocols bgp group DCI local-as 65277
set protocols bgp group DCI bfd-liveness-detection minimum-interval 250
set protocols bgp group DCI bfd-liveness-detection multiplier 3
set protocols bgp group DCI bfd-liveness-detection session-mode automatic
!
/* Конфигурация стыковочного интерфейса BR и PE для DCI Underlay */
set interfaces xe-0/0/3 flexible-vlan-tagging
set interfaces xe-0/0/3 encapsulation flexible-ethernet-services
set interfaces xe-0/0/3 unit 2000 vlan-id 2000
set interfaces xe-0/0/3 unit 2000 family inet address 201.199.195.2/30
!
/* Политика анонса префиксов Lo0 интерфейсов локального ЦОД */
set policy-options policy-statement POL-DCI-EXPORT term 10 from protocol bgp
set policy-options policy-statement POL-DCI-EXPORT term 10 then accept
set policy-options policy-statement POL-DCI-EXPORT term 20 from protocol direct
set policy-options policy-statement POL-DCI-EXPORT term 20 from interface lo0.0
set policy-options policy-statement POL-DCI-EXPORT term 20 then accept
 
```
</details>

#### _4.2. Обеспечение связности между ЦОД в Overlay_
Для обмена BGP EVPN-маршрутами создаются eBGP EVPN multihop-сессий между Route Reflectors (RR) разных ЦОД. Далее RR распространяют полученные префиксы внутри своих
ЦОД. Чтобы уменьшить объём сигнализации, в политике экспорта на DCI eBGP EVPN-сессиях разрешается передача Type 2- и Type 3-маршрутов только с target community, зарезервированными под растянутые VLAN/VXLAN – target:65000:1000-1999. При этом распространение Type 1 маршутов не ограничивается, так как они нужны для корректной работы EVPN в DCI, например, для сигнализации информации о ESI-LAG. Таким образом, префиксы Type 2 и Type 3, относящиеся к нерастянутым VLAN, не будут распространятся между ЦОД.

VNI-идентификаторы VXLAN должны быть одинаковыми в растянутых по разным ЦОД VXLAN. В рамках данного проекта для них зарезервирован диапазон значений 65XXXX. Автоматическое назначение target community не подходит для растянутых VXLAN, потому что при автоматическом назначении используется номер автоновной системы Overlay. В разных ЦОД номера Overlay AS разные, соответственно, автоматически будут назначены разные target community для одинаковых номеров VNI, а target community должны быть одинаковы для корректной работы сервиса. Поэтому target community для растянутых VXLAN конфигурируется вручную статически. 

Сами номера VLAN могут быть различными в разных ЦОД, но рекомендуется использовать одинаковые номера для простоты эксплуатации. Для таких растянутых VLAN рекомендуется зарезервировать отдельный диапазон, например, 3500-4000.

RT ES, используемая для Type 1 маршрутов, должна быть одинакова во всех ЦОД для корректной работы ESI-LAG. Если она будет разная, то один ЦОД не будет видеть ESI-LAG
другого ЦОД, а соответственно, и сервера, за ними расположенные. Технически можно применять разные RT ES в разных ЦОД и явным образом конфигурировать политики импорта/экспорта EVPN, чтобы добиться корректного обмена маршрутов, но вариант с одинаковой RT ES проще в эксплуатации.
 
<details>
<summary> Шаблон конфигурации d77-spine-r01-sw01 </summary>

 ```sh
/* BGP группа для DCI Overlay */
set protocols bgp group OVERLAY-DCI-EVPN type external
set protocols bgp group OVERLAY-DCI-EVPN multihop ttl 200
set protocols bgp group OVERLAY-DCI-EVPN multihop no-nexthop-change /* Multi-hop eBGP сессия без изменения аттрибута next-hop передаваемых маршрутов, по умолчанию
eBGP сессии являются single-hop и автоматически выполняют next-hop-self */
set protocols bgp group OVERLAY-DCI-EVPN local-address 10.77.0.1
set protocols bgp group OVERLAY-DCI-EVPN family evpn signaling
set protocols bgp group OVERLAY-DCI-EVPN export POL-OVERLAY-DCI-EVPN-EXPORT /* Политика фильтрации EVPN маршрутов DCI */
set protocols bgp group OVERLAY-DCI-EVPN multipath
set protocols bgp group OVERLAY-DCI-EVPN bfd-liveness-detection minimum-interval 250
set protocols bgp group OVERLAY-DCI-EVPN bfd-liveness-detection multiplier 3
set protocols bgp group OVERLAY-DCI-EVPN bfd-liveness-detection session-mode automatic
/* Сессии со всеми RR других ЦОД, RR full-mesh */
set protocols bgp group OVERLAY-DCI-EVPN neighbor 10.26.0.1 description d26-spine-r01-sw01
set protocols bgp group OVERLAY-DCI-EVPN neighbor 10.26.0.1 peer-as 65226
set protocols bgp group OVERLAY-DCI-EVPN neighbor 10.26.0.2 description d26-spine-r01-sw02
set protocols bgp group OVERLAY-DCI-EVPN neighbor 10.26.0.2 peer-as 65226
!
/* Политика фильтрации EVPN маршрутов DCI */
/* Разрешается только передача Type2 и Type3 маршрутов EVPN c community из выделенного диапазона */
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 10 from family evpn
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 10 from community CT-DCI-RAGNE
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 10 from nlri-route-type 2
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 10 from nlri-route-type 3
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 10 then accept
/* Разрешается передача всех Type1 и Type4 маршрутов */
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 20 from family evpn
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 20 from nlri-route-type 1
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 20 from nlri-route-type 4
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 20 then accept
set policy-options policy-statement POL-OVERLAY-DCI-EVPN-EXPORT term 1000 then reject
!
set policy-options community CT-DCI-RAGNE members "target:65000:1[0-9][0-9][0-9]";
 
```
</details>
 
<details>
<summary> Шаблон конфигурации d77-leaf-r11-sw01 </summary>

 ```sh
set switch-options vtep-source-interface lo0.0
set switch-options vrf-target auto /* Автоматическое назначение vrf-target для нерастянутых VXLAN */
set switch-options vrf-target target:65000:9999 /* RT ES одинаковая во всех ЦОД в целях корректной работы DCI для ESI-LAG */
set switch-options route-distinguisher 10.77.0.11:1
!
set protocols evpn vni-options vni 653550 vrf-target target:65000:99 /* Статическое назначение vrf-target для растянутых VXLAN */
set protocols evpn encapsulation vxlan
set protocols evpn encapsulation vxlan
!
set vlans v3550 description DC1-DC2
set vlans v3550 vlan-id 3550
set vlans v3550 vxlan vni 773550 /* VNI из зарезервированого для DCI диапазона, одинаков во всех ЦОД для данного VXLAN */
 
```
</details>

### _5. Кратко про взаимодействие с оборудованием безопасности_

Блок безопасности состоит из кластера межсетевых экранов и кластера VPN-концентраторов. Каждый из кластеров представляет собой пару устройств, работающих в режиме Active-Backup. Протокольная сигнализация для передачи информации о текущем состоянии кластеров блока безопасности с оборудованием ЦОД не используется. Решение о
том, какое из устройств кластера является активным на конкретный момент, принимается парой устройств кластера по их внутренним протоколам сигнализации. При переключении кластером безопасности Active-Backup режима между устройствами, с точки зрения фабрики ЦОД происходит простое переучивание MAC адресов. Основные направления трафика внутри ЦОД следующие:

* Между двух серверов внутри одного VLAN/VXLAN – простая L2-коммутация;
* От сервера до BR L3GW (любой трафик требующий маршрутизацию, например, трафик между VXLAN или из VXLAN через L3GW шлюз в сеть вышестоящего провайдера);
* Между серверами и межсетевым экраном;
* Между межсетевым экраном и BR L3GW;
* Между VPN-концентратором и межсетевым экраном;
* Между VPN-концентратором и BR L3GW.

### _6. Общие правила и рекомендации по эксплуатации сети_
В данном разделе описаны правила именование площадок и устройств. Также в данном разделе описано, как назначать ASN, Lo интерфейсы, MTU, VNI и т.д. Раздел создан с целью упростить понимание архитектуры и процессы эксплуатации оборудования, а также облегчить процесс поиска неполадок. 

#### _6.1. Правила обозначения площадок_
Для того, чтобы упростить понимание, на какой площадке находится ЦОД ему назначают уникальный идентификатор **_dxx_**, где **_d_** расшифровывается как Data Center, а xxx - значение от 1 до 999. В нашем случае каждой площадке присваиваются следующие номера:

* d77 - г. Москва;
* d26 - г. Ставрополь.

В данном примере значения xxx выбраны исходя из их номера региона. Используя номера регионов для обозначения площадок ЦОД можно будет, практически, не "подглядывая" в документацию, определить, в каком городе или регионе расположен ЦОД. Если же в рамках одного региона будут располагаться 2 или 3 ЦОДа, то к значению региона прибавляется +100. Например, если в Москве будут расположены 2 или 3 ЦОДа, то второму ЦОДу будет присвоен идентификатор d177, а третьему - d277.

#### _6.2. Правила именования оборудования_
Для назначения наименования устройства (hostname) предлагается использовать следующее обозначение:
{site_name}(3)-{role}(2-5)-{rack}(3)-{device_id}(2), 

где

{site_name} - дентификатор площадки, на которой установлено устройство, длина – 3 символа, используемые символы: a-z, 0-9, нижний регистр. В рамках данного проекта используются площадки с кодами d77 и d26.

{role} - идентификатор роли устройства в ЦОД, длина – 2-5 символов, используемые символы: 0-9, a-z, нижний регистр. В рамках проектной работы используются следующие
идентификаторы устройств leaf, spine, br.

{rack} - идентификатор стойки, длина – 3 символа, используемые символы: 0-9, r, нижний регистр. Например, если устройство расположено в 5 стойке, то его номер будет r05.

{device_id} - уникальный идентификатор устройства данной роли в ЦОД.

Например, d77-leaf-r01-sw01 расшифровывается как Leaf-коммутатор с id=1 на площадке d77 (Москва) в стойке 1.

Использование данного правила значительно успростит понимание, к какому оборудованию вы подключены и в каком ЦОДе оно расположено.

#### _6.3. План адресации и нумерации ASN_

При распределении всей адресной информации между оборудованием ЦОД предлагается ввести таблицу с числовым ID каждого из устройств. Тогда вычисление необходимых значений адресных переменных будет осуществляться следующим образом: { parameter_space } + { device_id }. Это удобно при использовании средств автоматизации при генерации шаблонов конфигурации и выполнении проверочных тестов.

Таблица 1 - Таблица соответствия hostname и Device ID для ЦОД d77
|Hostname            |Device ID |
|--------------------|----------|
|d77-spine-r01-sw01  |1         |
|d77-spine-r01-sw02  |2         |
|d77-leaf-r11-sw01   |11        |
|d77-leaf-r12-sw02   |12        |
|d77-leaf-r13-sw03   |13        |
|d77-leaf-r14-sw04   |14        |
|d77-br-r02-br01     |4         |
|d77-br-r02-br02     |5         |

Таблица 2 - Таблица соответствия hostname и Device ID для ЦОД d26
|Hostname            |Device ID |
|--------------------|----------|
|d26-spine-r01-sw01  |1         |
|d26-spine-r01-sw02  |2         |
|d26-leaf-r11-sw01   |11        |
|d26-leaf-r12-sw02   |12        |
|d26-leaf-r13-sw03   |13        |
|d26-leaf-r14-sw04   |14        |
|d26-br-r02-br01     |4         |
|d26-br-r02-br02     |5         |

При добавлении нового оборудования в ЦОД необходимо назначить следующий свободный ID из соответствующего диапазона.

В рамках данного проекта разработано следующее распределение IP подсетей:
* 10.x.0.0/24 – диапазон для выделения адресов loopback коммутаторов ЦОД;
* 10.x.1.0/24 – диапазон для выделения адресных префиксов Point-to-Point линковых подсетей (длина префиксов /31);
* 10.x.253.0/25 - диапазон для выделения под mgmt IP-адреса коммутаторов.

Выделять 2 октет для сетей/подсетей рекомендуется исходя из номера площадки ЦОД. Например, если ЦОДу в г. Москва присвоен номер d77, то общая сеть у нее будет 10.77.0.0/16, которая, далее, разбивается на подсети. В случае назначения 3 октета закономерность отсутствует. Если возникнет ситуация, что в рамках одного региона у организации 2 ЦОДа, то второму ЦОДу 2 октет назначается по правилу № региона + 100. Например, в случае Москвы общая сеть 2 ЦОДа будет 10.177.0.0/16. Данное правило выделения 2 октета для сети не распространяется на регионы со значением выше 55. Например, если в Москве расположено 3 ЦОДа, то третьему ЦОДу уже не получится назначить во 2 октет значение 277. Следовательно, для 3 ЦОДа в качестве 2 октета выбираем любое свободное число, главное, чтобы оно не пересекалось с адресацией в других регионах. Как правило, такие исключения бывают редко, так как не часто встретишь ситуацию, когдва в рамках одного региона строят более 2х ЦОДов.

Что касается выделения последнего октета для Loopback и Mgmt интерфейса, то вычисление происходит путем прибавления ID к последнему октету адреса подсети, а MGMT-адреса путем прибавления ID+10 к последнему октету адреса подсети. Например, для d77-leaf-r11-sw01 с id=11 loopback IP равен 10.77.0.11. А MGMT-адрес равен 10.77.0.21. Исключением из предлагаемых правил генерации IP-адресов являются адреса BR устройств, так как данные устройства имеют как внешнюю, так и частную адресацию.

Для удобства нумерации используется 32-битные приватные ASN32 вида 42077000 + { device_id }(2) для d77 и 42026000 + { device_id }(2) для d26. Для device_id, состоящих из одного символа, слева добавляется ноль для сохранения формата записи. Для Overlay используются приватные ASN16:
* d77 – AS65277;
* d26 – AS65226.

Закономерности в выборе приватной ASN16 нет.

Как итог, в таблице 3, 4 представлен план адресации и нумерации ASN для Underlay

Таблица 3 - План адресации и нумерации ASN для d77
|Hostname            |mgmt IP                |Loopback               |Underlay AS  |Underlay link Spine-1 (subnet)|Underlay link Spine-2 (subnet)|
|--------------------|-----------------------|-----------------------|-------------|------------------------------|------------------------------|
|d77-spine-r01-sw01  |10.77.253.11/25        |10.77.0.1/32           |4207700001   |10.77.1.0/25                  |                              |
|d77-spine-r01-sw02  |10.77.253.12/25        |10.77.0.2/32           |4207700002   |                              |10.77.1.128/25                |
|d77-leaf-r11-sw01   |10.77.253.21/25        |10.77.0.11/32          |4207700011   |10.77.1.0/31                  |10.77.1.128/31                |
|d77-leaf-r12-sw02   |10.77.253.22/25        |10.77.0.12/32          |4207700012   |10.77.1.2/31                  |10.77.1.130/31                |
|d77-leaf-r13-sw03   |10.77.253.23/25        |10.77.0.13/32          |4207700013   |10.77.1.4/31                  |10.77.1.132/31                |
|d77-leaf-r14-sw04   |10.77.253.24/25        |10.77.0.14/32          |4207700014   |10.77.1.6/31                  |10.77.1.134/31                |
|d77-br-r02-br01     |10.77.253.4/25         |10.77.0.4/32           |4207700004   |10.77.1.120/31                |10.77.1.248/31                |
|d77-br-r02-br02     |10.77.253.7/25         |10.77.0.5/32           |4207700005   |10.77.1.122/31                |10.77.1.250/31                |

Таблица 4 - План адресации и нумерации ASN для d26
|Hostname            |mgmt IP                |Loopback               |Underlay AS  |Underlay link Spine-1 (subnet)|Underlay link Spine-2 (subnet)|
|--------------------|-----------------------|-----------------------|-------------|------------------------------|------------------------------|
|d26-spine-r01-sw01  |10.26.253.11/25        |10.26.0.1/32           |4202600001   |10.26.1.0/25                  |                              |
|d26-spine-r01-sw02  |10.26.253.12/25        |10.26.0.2/32           |4202600002   |                              |10.26.1.128/25                |
|d26-leaf-r11-sw01   |10.26.253.21/25        |10.26.0.11/32          |4202600011   |10.26.1.0/31                  |10.26.1.128/31                |
|d26-leaf-r12-sw02   |10.26.253.22/25        |10.26.0.12/32          |4202600012   |10.26.1.2/31                  |10.26.1.130/31                |
|d26-leaf-r13-sw03   |10.26.253.23/25        |10.26.0.13/32          |4202600013   |10.26.1.4/31                  |10.26.1.132/31                |
|d26-leaf-r14-sw04   |10.26.253.24/25        |10.26.0.14/32          |4202600014   |10.26.1.6/31                  |10.26.1.134/31                |
|d26-br-r02-br01     |10.26.253.4/25         |10.26.0.4/32           |4202600004   |10.26.1.120/31                |10.26.1.248/31                |
|d26-br-r02-br02     |10.26.253.7/25         |10.26.0.5/32           |4202600005   |10.26.1.122/31                |10.26.1.250/31                |

#### _6.4. Таблица значений VNI, используемых на площадках d77 и d26_
Значения VNI в обоих ЦОД должны быть уникальными. Ниже представлены рекомендуемые значения VNI для каждой из площадок, а также значения для «растянутых» VLAN:
* 77xxxx – локальные VLAN для d77, где xxxx – значение VLAN. В случае, если значение VLAN – это трёх- или менее значное число, то к этому значению слева дописывается
необходимое количество нулей для сохранения формата записи номеров VNI. Например, 773011 (VLAN3011) и 770018 (VLAN18);
* 26yyyy – локальные VLAN для d26. Например, 260025 (VLAN25) и 263500 (VLAN3500);
* 65zzzz – «растянутые» VLAN между d77 и d26. Например, 650048 (VLAN48) или 650005 (VLAN 5).
