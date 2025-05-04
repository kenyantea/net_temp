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
* passive-interface f0/0 — отключаем интерфейс там, где ПК 
* net <сеть> <инверсная маска> area 0 — добавляем сети, зона по умолчанию
* на интерфейсе включаем аутентификацию ip ospf authentication message-digest 
* ip ospf message-digest-key 10 md5 10 — пароль и хэш, настраивается подобное для обоих интерфейсов  

## Лаба 1 (весна)
* telnet <ip-адрес> — удаленное подключение через telnet
* show cdp neighbors (detail) — соседи по протоколу cisco discovery

#### для вычисления айпи (в том числе для PC) понадобится
* show arp — таблица arp (address resolution protocol) для роутера
* show mac-address-table — таблица мак-адресов для свича

### Настройка ssh
* conf t 
* username <имя> privilege 15 secret <пароль> — назначение нового пользователя 
* ip domain-name <имя> — доменное имя
* crypto key generate rsa — генерация ключа для ssh, после этой команды ввести, например, 1024, 2048...
* line vty 0 4
* transport input ssh
* (опционально) transport input all — чтобы можно было подключаться не только по ssh, но и по telnet
* login local
* ssh -l <имя> <ip-адрес>

## Лаба 2 (весна)

### Сбрасываем у роутера пароль
* физически выключаем и включаем его, бежим в консоль
* confreg 2142
* reset — пойдет перезагрузка
* нажимаем no, когда будет вопрос "continue with configuration dialog?"
* входим в enable
* copy start run — копируем конфиг
* conf t
* no enable secret/password — отключаем пароли
* config-register 2102
* exit
* write
* (выходим) reload

## Лаба 3 (весна)
настройка свича (STP)
* hostname <имя>
* no ip domain-lookup
* [настройка ssh](#настройка-ssh)
* vlan <нужный номер>
* int range _f0/10-13_ — настройки сразу для диапазона интерфейсов
* switch mode trunk — после изменения свойств интерфейсов нас выкинет
* spanning-tree mode rapid-pvst
* int _f0/12_
* spanning-tree link-type point-to-point либо shared — тип соединения между коммутаторами указан в задании
* int _f0/3_
* switchport mode access -> switchport port-security — сначала мы явно должны перевести порт в режим access
* **статический port-security (shutdown) edge-port**: switchport mode access -> switch access vlan 30 -> sw port-security -> spanning-tree portfast (если надо) -> sw port-security maximum 1 (ограничение в 1 устройство) -> sw po mac-address <мак-адрес>
* **bpduguard**: sw mo ac -> sw ac vlan 30 -> spanning-tree bpduguard enable (интерфейс при "вредоносном" пакете переходит в error-disabled)
* **статический port-security restrict/protect**: так же, как и для shutdown, но с добавлением команды sw po violation restrict/protect
* sw po mac-address sticky позволяет записать мак-адрес автоматически, то есть не придется узнавать мак-адрес самому. ну а вообще, это вроде как **динамический port-security**

### Настройка агрегирования 

* переходим на нужные интерфейсы
* shutdown — надо вырубить интерфейс(ы)
* sw mo tr
* channel-group 1 mode <...>
  * desirable/auto (для одного свича первое, для второго второе, хотя вроде можно для обоих первое) — для PaGP
  * active — для LACP
  * on — для статического
* no sh
* так настраиваем для обоих свичей
* show etherchannel summary — показывает агрегированные каналы (как мы их соединили)

### Настройка DHCP
* ip dhcp pool *vlan40*
* network <ip> <маска>
* default-router <ip>
* dns-server 8.8.8.8
* ip dhcp excluded-address <ip default-router и прочие>

## Лаба №4

### Dual Stack.

Настройка IP-адресов интерфейсов:
* cont f
* interface se0/0/0
* ip address 10.2.2.1 255.255.255.252
* ipv6 address 2001:2:2:2::1/64
* ipv6 enable
* no shutdown
* (то же самое для другого роутера)
* Для включения передачи сообщений по IPv6 необходимо на каждом маршрутизаторе ввести команду: ipv6 unicast-routing

### IPv6 tunneling
* ipv6 unicast-routing
* interface tunnel 0
* ipv6 address 2003::1/64
* tunnel mode ipv6ip
* tunnel source fa0/0
* tunnel destination 10.3.3.2
* ipv6 route ::/0 2003::2 (если нужен маршрут)

### Совместная работа IPv6 и IPv4. Настройка NAT-PT
* ipv6 enable
* ipv6 nat
* ipv6 nat prefix 2002:20::/96
* ipv6 nat v4v6 source 10.0.0.2 2002:20::2
* ipv6 nat v6v4 source 2001:15::2 15.16.17.2
* show ipv6 nat translations — посмотреть преобразования NAT 

### DHCPv6 (у Сосенушкина)
* ipv6 unicast-routing
* ipv6 dhcp pool _v6pool_ — у него перебрасывает в режим config-dhcpv6, у меня просто в config-dhcp
* address prefix <адрес/префикс>
* interface _fa0/0_
* ipv6 dhcp server _v6pool_
* ipv6 np managed-config-flag
* no shutdown
* ipv6 address _тот же префикс_

### DHCPv6 (у меня как получилось)
* ipv6 dhcp pool MY_POOL
* dns-server 2001:4860:4860::8888 — публичный
* domain-name example.com — необязательно
* interface GigabitEthernet0/0
* ipv6 address 2001:9::1/64
* ipv6 nd other-config-flag
* ipv6 dhcp server marina

### Прочие замечания
* ipv6 enable — база
* смотреть на ip route и ipv6 route обязательно

## Лаба 5 (весна)

### настройка туннелирования
* ip route <ip-адрес> <маска> _s0/1/0_ — предварительная настройка с двух сторон
* interface tun0 — Tunnel0, можно любой id указать
* tunnel mode gre ip
* tunnel source _s0/1/0_
* tunnel destination <ip-адрес>
* эти команды выполняем с двух сторон (с двух роутеров)

### настройка EIGRP 
* router eigrp _100_
* network <ip> <маска> — повторить для всех нужных сетей
* no auto-summary
* passive-interface _eth0/0/0_ — все те, которые соединены с PC, то есть не между роутерами
* повторяем для каждого роутера, который планируется включить в eigrp

## DR (Designated Router) и BDR (Backup DR)
* interface lo0 (Loopback)
* ip address 1.0.0.0 255.255.255.255 — наименьший адрес, маска 32, чтобы этот адрес стал идентификатором роутера
* переходим на роутер, выбранный BDR
* ip address 2.0.0.0 255.255.255.255 — второй наименьший адрес

## Прочее
[Штука №1](https://ipcalc.co/)

[Суммарные маршруты](https://fixmypc.ru/services/route-summary/)

чтобы добавить адрес, надо в dns назначить имя и адрес, соответствующий адресу сервера

как идут по заданию адреса, так и назначай

**tip**: если потеряла доступ к роутеру или не можешь войти в кластер, то PC + синий проводок, подключенный к console, вероятно, выручит
