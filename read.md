# Эво конспект
Содержание:
* [Лаба 1](#лаба-1)
* [Лаба 2](#лаба-2)
* [Лаба 3](#лаба-3)
* [Лаба 4](#лаба-4)

## Лаба 1

* кол-во узлов LAN: n+2, 2 для broadcast и subnet
* самый младший адрес (не subnet) для шлюза по умолчанию, для ПК адреса берем с конца

### настройка маршрутизатора
* enable
* configure terminal
* hostname <имя> — для имени, что слева
* interface _fa0/0_
* ip address <адрес> <маска>
* no shutdown — чтобы интерфейс врубился

настройка статического адреса в настройках PC

## Лаба 2

* мак-адреса: show mac-address-table
* show spanning-tree — инфа о протоколе spanning-tree

### vtp (vlan transfer protocol)
#### настройка коммутаторов
* vlan 2 — новый vlan
* name abc — его название
* show vlan — смотрим, какие есть vlan
* access для интерфейсов, связанных с ПК, trunk для маршрутизаторов/коммутаторов
* switchport mode trunk — переводим интерфейс 
* show vtp status 
* vtp domain abc
* vtp password abc
* vtp mode client — лучше вводить все коммутаторы в качестве клиентов
* switchport access vlan 2 — настраиваем vlan на интерфейсах

#### настройка маршрутизатора
* int f0/0.2 — сабинтерфейс, который создается при первом обращении 
* encapsulation dot1Q 2 — цифра есть номер vlan
* ip address...
* и так для всех vlan

## Лаба 3 

### Статическая маршрутизация
* show ip interfaces brief — показывает интерфейсы маршрутизатора
* show ip route — информация о том, какие маршруты есть
* ip route <адрес назначения> <маска адреса назначения> <адрес next hop> — если кратко, то интерфейс/маска и "связущий" интерфейс
* суммарный адрес вычисляется по минималке (охватываем все типа) — это для internet


## Лаба 4

### RIP
#### Настраиваем маршрутизаторы
* router rip
* ver 2
* no auto-summary — отключаем автоматическое суммирование маршрутов
* network <сабнет нужной сети> — все сети, которые нужны будут 
* passive-interface f0/0 — чтобы не работал rip там, где он не нужен
* default-information originate — источник маршрута по умолчанию
* ip route 0.0.0.0 0.0.0.0 <сеть> — смотрящая на провайдера сети, в этой лабе чтобы было маршрутом по умолчанию

### OSPF
* router ospf 5 — номер означает айди процесса, любой
* passive-interface f0/0
* net <сеть> <инверсная маска> area 0 — добавляем сети, зона по умолчанию
* отключаем интерфейс там, где ПК 
* на интерфейсе включаем аутентификацию ip ospg authentication message-digest 
* ip ospf message-digest-key 10 md5 10 — пароль и хэш, настраивается подобное для обоих интерфейсов  

## Лаба 1 (весна)
* telnet <ip-адрес> — удаленное подключение через telnet
* show cdp neighbors (detail) — соседи по протоколу cisco discovery

#### для вычисления айпи (в том числе для PC) понадобится
* show arp — таблица arp (address resolution protocol) для роутера
* show mac-address-table — таблица мак-адресов для свича

### Настройка ssh
* username <имя> privilege 15 secret <пароль> — назначение нового пользователя 
* ip domain-name <имя> — доменное имя
* crypto key generate rsa — генерация ключа для ssh.
* line vty 0 4
* transport input ssh
* login local
* ssh -l <имя> <ip-адрес>


## Прочее
[Штука №1](https://ipcalc.co/)

[Суммарные маршруты](https://fixmypc.ru/services/route-summary/)

чтобы добавить адрес, надо в dns назначить имя и адрес, соответствующий адресу сервера

как идут по заданию адреса, так и назначай
