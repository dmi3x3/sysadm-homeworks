# Домашнее задание к занятию "3.7. Компьютерные сети, лекция 2"

1. Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?

Ответ:
Linux "ip link show" = "ip l"
```bash
vagrant@vagrant:~$ ip l
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:73:60:cf brd ff:ff:ff:ff:ff:ff
```
Windows
```bash
C:\Documents and Settings\student>ipconfig /all

Настройка протокола IP для Windows

   Имя компьютера  . . . . . . . . . : shopsterminal
   Основной DNS-суффикс  . . . . . . : int.office.ru
   Тип узла. . . . . . . . . . . . . : неизвестный
   IP-маршрутизация включена . . . . : нет
   WINS-прокси включен . . . . . . . : нет
   Порядок просмотра суффиксов DNS . : int.office.ru

Подключение по локальной сети 2 - Ethernet адаптер:

   DNS-суффикс этого подключения . . : int.office.ru
   Описание  . . . . . . . . . . . . : Embedded Broadcom NetXtreme 5721 PCI-E Gi
gabit NIC #2
   Физический адрес. . . . . . . . . : 00-1A-4B-A4-09-22
   DHCP включен. . . . . . . . . . . : нет
   IP-адрес  . . . . . . . . . . . . : 192.168.2.17
   Маска подсети . . . . . . . . . . : 255.255.0.0
   IP-адрес  . . . . . . . . . . . . : 192.168.1.5
   Маска подсети . . . . . . . . . . : 255.255.0.0
   IP-адрес  . . . . . . . . . . . . : 192.168.1.17
   Маска подсети . . . . . . . . . . : 255.255.0.0
   Основной шлюз . . . . . . . . . . : 192.168.1.1
   DNS-серверы . . . . . . . . . . . : 192.168.50.1
                                       192.168.110.3

C:\Documents and Settings\student>netsh interface ip show config

Настройка интерфейса "Подключение по локальной сети 2"
    DHCP разрешен:                        Нет
    IP-адрес:                             192.168.1.17
    Маска подсети:                        255.255.0.0
    IP-адрес:                             192.168.1.5
    Маска подсети:                        255.255.0.0
    IP-адрес:                             192.168.2.17
    Маска подсети:                        255.255.0.0
    Основной шлюз:                        192.168.1.1
    Метрика шлюза:                        0
    Метрика интерфейса:                   0
    Статически настроенные DNS-серверы:    192.168.50.1
                                          192.168.110.3
    Статически настроенные WINS-серверы:   Отсутствует
    Зарегистрировать с суффиксом:           НЕТ

C:\Documents and Settings\student>netsh diag show adapter

Сетевые платы
     1. [00000008] Embedded Broadcom NetXtreme 5721 PCI-E Gigabit NIC
     2. [00000009] Устройство фильтрации для балансировки нагрузки сети


C:\Documents and Settings\student>netsh diag show ip

IP-адрес
     8. [00000008] Embedded Broadcom NetXtreme 5721 PCI-E Gigabit NIC
        IPAddress = 192.168.1.17
                    192.168.1.5
                    192.168.2.17
```
netsh в windows используется чаще для настройки сети, смотреть в нем информацию по интерфейсам не очень удобно.

2. Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?

Ответ: LLDP – протокол для обмена информацией между соседними устройствами, позволяет определить к какому порту коммутатора подключен сервер.
пакет для работы по протоколу LLDP в linux (Ubuntu 20.04) - lldpd.

```bash
root@vagrant:~# systemctl enable lldpd && systemctl start lldpd
Synchronizing state of lldpd.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable lldpd
root@vagrant:~# systemctl status lldpd
● lldpd.service - LLDP daemon
     Loaded: loaded (/lib/systemd/system/lldpd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-01-07 00:00:30 UTC; 8min ago
       Docs: man:lldpd(8)
   Main PID: 13253 (lldpd)
      Tasks: 2 (limit: 1071)
     Memory: 2.7M
     CGroup: /system.slice/lldpd.service
             ├─13253 lldpd: monitor.
             └─13255 lldpd: no neighbor.

Jan 07 00:00:30 vagrant systemd[1]: Started LLDP daemon.
Jan 07 00:00:30 vagrant lldpd[13255]: /etc/localtime copied to chroot
Jan 07 00:00:30 vagrant lldpd[13255]: protocol LLDP enabled
Jan 07 00:00:30 vagrant lldpd[13255]: protocol CDPv1 disabled
Jan 07 00:00:30 vagrant lldpd[13255]: protocol CDPv2 disabled
Jan 07 00:00:30 vagrant lldpd[13255]: protocol SONMP disabled
Jan 07 00:00:30 vagrant lldpd[13255]: protocol EDP disabled
Jan 07 00:00:30 vagrant lldpd[13255]: protocol FDP disabled
Jan 07 00:00:30 vagrant lldpd[13255]: libevent 2.1.11-stable initialized with epoll method
Jan 07 00:00:30 vagrant lldpcli[13254]: lldpd should resume operations
root@vagrant:~# lldpcli sh neigh
-------------------------------------------------------------------------------
LLDP neighbors:
-------------------------------------------------------------------------------
Interface:    eth0, via: LLDP, RID: 1, Time: 0 day, 00:00:08
  Chassis:     
    ChassisID:    mac e0:d9:e3:a9:c9:80
  Port:        
    PortID:       ifname gi1/0/1
-------------------------------------------------------------------------------
Interface:    eth0, via: LLDP, RID: 2, Time: 0 day, 00:00:04
  Chassis:     
    ChassisID:    mac e0:d9:e3:bc:9b:40
  Port:        
    PortID:       ifname gi1/0/24
-------------------------------------------------------------------------------
root@vagrant:~# lldpcli sh stat sum
-------------------------------------------------------------------------------
LLDP Global statistics:
-------------------------------------------------------------------------------
Summary of stats:
  Transmitted:  13
  Received:     25
  Discarded:    0
  Unrecognized: 0
  Ageout:       0
  Inserted:     2
  Deleted:      0
-------------------------------------------------------------------------------
root@vagrant:~# lldpcli sh configuration
-------------------------------------------------------------------------------
Global configuration:
-------------------------------------------------------------------------------
Configuration:
  Transmit delay: 30
  Transmit hold: 4
  Receive mode: no
  Pattern for management addresses: (none)
  Interface pattern: (none)
  Interface pattern for chassis ID: (none)
  Override description with: (none)
  Override platform with: (none)
  Override system name with: (none)
  Advertise version: yes
  Update interface descriptions: no
  Promiscuous mode on managed interfaces: no
  Disable LLDP-MED inventory: yes
  LLDP-MED fast start mechanism: yes
  LLDP-MED fast start interval: 1
  Source MAC for LLDP frames on bond slaves: local
  Portid TLV Subtype for lldp frames: unknown
-------------------------------------------------------------------------------
```

  - lldpcli sh neigh или lldpctl - показывает соседей
  - ldpcli sh stat sum - статистика статистика по всем интерфейсам
  - lldpcli sh configuration - показывает конфигурацию программы

3. Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.

Ответ: VLAN – виртуальное разделение коммутатора.

Пакет для VLAN в Linux называется vlan.

Команды - vconfig (deprecated) и ip link

Добавление VLAN-интерфейса и просмотр его параметров
```bash
root@vagrant:~# ip link add link eth0 name eth0.1500 type vlan id 1500
root@vagrant:~# ip -d link show eth0.1500
3: eth0.1500@eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 08:00:27:73:60:cf brd ff:ff:ff:ff:ff:ff promiscuity 0 minmtu 0 maxmtu 65535 
    vlan protocol 802.1Q id 1500 <REORDER_HDR> addrgenmode eui64 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 
```

Для изменения параметров VLAN-интерфейса используется команда ip link set, с обязательным указанием type vlan.
```bash
 ip link set dev VLANNAME type vlan OPTION VALUE
````
Удаление VLAN-интерфейса
```bash
 ip link del VLANNAME
```

Настройки подынтерфейсов VLANов в Ubuntu точно так же, как и для сетевых интерфейсов, указываются в файле /etc/network/interfaces.

Для того чтобы информация о созданных VLAN'ах сохранилась после перезагрузки, необходимо добавить её в файл /etc/network/interfaces. Например:

```bash
auto vlan1400
iface vlan1400 inet static
        address 192.168.1.1
        netmask 255.255.255.0
        vlan_raw_device eth0
```
Или, если хочется использовать названия интерфейсов вида eth0.1400, а не vlan1400, то можно указывать прямо их:

```bash
auto eth0.1400
iface eth0.1400 inet static
        address 192.168.1.1
        netmask 255.255.255.0
        vlan_raw_device eth0
```

4. Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.

Ответ:

    LAG (Link Aggregation Group) – агрегация портов 
    LAG в Linux – бондинг,  имя интерфейса bond0, bond1.
    Типы LAG:
        ●статический (на Cisco mode on);
        ●динамический – LACP (Link Aggregation Control Protocol, описывается стандартом IEEE 802.3ad)
                        протокол (на Cisco mode active).

Статическое агрегирование:

Преимущества:
Не вносит дополнительную задержку при поднятии агрегированного канала или изменении его настроек
Вариант, который рекомендует использовать Cisco
Недостатки:
Нет согласования настроек с удаленной стороной. Ошибки в настройке могут привести к образованию петель
Агрегирование с помощью LACP:

Преимущества:
Согласование настроек с удаленной стороной позволяет избежать ошибок и петель в сети.
Поддержка standby-интерфейсов позволяет агрегировать до 16ти портов, 8 из которых будут активными, а остальные в режиме standby
Недостатки:
Вносит дополнительную задержку при поднятии агрегированного канала или изменении его настроек

В Linux агрегирование сетевых интерфейсов называется "бондингом". Управление агрегированием осуществляется с помощью утилиты "ifenslave", а работа, как таковая, с помощью модуля ядра "bonding".

Поддерживаются следующие режимы агрегирования (опции):
```text
  -  mode=0 (balance-rr)    Последовательно кидает пакеты, с первого по последний интерфейс.

  -  mode=1 (active-backup) Один из интерфейсов активен. Если активный интерфейс выходит из 
                            строя (link down и т.д.), другой интерфейс заменяет активный. Не 
                            требует дополнительной настройки коммутатора

  - mode=2 (balance-xor)    Передачи распределяются между интерфейсами на основе формулы 
                            ((MAC-адрес источника) XOR (MAC-адрес получателя)) % число интерфейсов. 
                            Один и тот же интерфейс работает с определённым получателем. Режим даёт 
                            балансировку нагрузки и отказоустойчивость.
  - mode=3 (broadcast)      Все пакеты на все интерфейсы
  
  - mode=4 (802.3ad)        Link Agregation — IEEE 802.3ad, требует от коммутатора настройки.
  
  - mode=5 (balance-tlb)    Входящие пакеты принимаются только активным сетевым интерфейсом, 
                            исходящий распределяется в зависимости от текущей загрузки каждого 
                            интерфейса. Не требует настройки коммутатора.

  - mode=6 (balance-alb)    Тоже самое что 5, только входящий трафик тоже распределяется между 
                            интерфейсами. Не требует настройки коммутатора, но интерфейсы должны
                            уметь изменять MAC.
```

Пример конфига

Объединение кабельного и беспроводного сетевых интерфейсов (RJ45/WLAN) вместе, чтобы определить единый виртуальный (т.е. связующий) сетевой интерфейс (например, bond0).

Пока сетевой кабель подключен, его интерфейс (например, eth0) используется для сетевого трафика. Если RJ45 будет отключен, ifenslave переключается на беспроводной интерфейс (например, wlan0) прозрачно, без потеря сетевых пакетов.

После повторного подключения сетевого кабеля ifenslave снова переключается на eth0. ("режим аварийного переключения").

С обратной стороны подключения (коммутатор, шлюз) не имеет значения, какой интерфейс активен. Связующее устройство представляет свой собственный программно-определяемый (т. е. виртуальный) MAC-адрес, отличный от аппаратно-определяемых MAC-адресов eth0 или wlan0.

DHCP-сервер будет использовать этот MAC-адрес для назначения IP-адреса устройству bond0. Таким образом, компьютер имеет один уникальный IP-адрес, по которому его можно идентифицировать. Без связывания каждый интерфейс будет иметь свой собственный IP-адрес.

```bash
vagrant@vagrant:~$ cat /etc/network/interfaces

# Define slaves   
auto eth0
iface eth0 inet manual
    bond-master bond0
    bond-primary eth0
    bond-mode active-backup
   
auto wlan0
iface wlan0 inet manual
    wpa-conf /etc/network/wpa.conf
    bond-master bond0
    bond-primary eth0
    bond-mode active-backup

# Define master
auto bond0
iface bond0 inet dhcp
    bond-slaves none
    bond-primary eth0
    bond-mode active-backup
    bond-miimon 100
```

5. Сколько IP адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.

Ответ: подсеть с маской /29 занимает 8 ip-адресов, емкость сети с маской /24 - 256 адресов, 256 / 8 = 32 (подсети с маской /29 можно получить из сети с маской /24)

Сделал небольшой однострочник, который генерирует параметры подсети с маской /29, вот часть его вывода:
```bash
vagrant@vagrant:~$ seq 0 8 255 | while read str; do ipcalc 10.10.10.$str/29; done
Address:   10.10.10.0           00001010.00001010.00001010.00000 000
Netmask:   255.255.255.248 = 29 11111111.11111111.11111111.11111 000
Wildcard:  0.0.0.7              00000000.00000000.00000000.00000 111
=>
Network:   10.10.10.0/29        00001010.00001010.00001010.00000 000
HostMin:   10.10.10.1           00001010.00001010.00001010.00000 001
HostMax:   10.10.10.6           00001010.00001010.00001010.00000 110
Broadcast: 10.10.10.7           00001010.00001010.00001010.00000 111
Hosts/Net: 6                     Class A, Private Internet

Address:   10.10.10.8           00001010.00001010.00001010.00001 000
Netmask:   255.255.255.248 = 29 11111111.11111111.11111111.11111 000
Wildcard:  0.0.0.7              00000000.00000000.00000000.00000 111
=>
Network:   10.10.10.8/29        00001010.00001010.00001010.00001 000
HostMin:   10.10.10.9           00001010.00001010.00001010.00001 001
HostMax:   10.10.10.14          00001010.00001010.00001010.00001 110
Broadcast: 10.10.10.15          00001010.00001010.00001010.00001 111
Hosts/Net: 6                     Class A, Private Internet

Address:   10.10.10.16          00001010.00001010.00001010.00010 000
Netmask:   255.255.255.248 = 29 11111111.11111111.11111111.11111 000
Wildcard:  0.0.0.7              00000000.00000000.00000000.00000 111
=>
Network:   10.10.10.16/29       00001010.00001010.00001010.00010 000
HostMin:   10.10.10.17          00001010.00001010.00001010.00010 001
HostMax:   10.10.10.22          00001010.00001010.00001010.00010 110
Broadcast: 10.10.10.23          00001010.00001010.00001010.00010 111
Hosts/Net: 6                     Class A, Private Internet

Address:   10.10.10.24          00001010.00001010.00001010.00011 000
Netmask:   255.255.255.248 = 29 11111111.11111111.11111111.11111 000
Wildcard:  0.0.0.7              00000000.00000000.00000000.00000 111
=>
Network:   10.10.10.24/29       00001010.00001010.00001010.00011 000
HostMin:   10.10.10.25          00001010.00001010.00001010.00011 001
HostMax:   10.10.10.30          00001010.00001010.00001010.00011 110
Broadcast: 10.10.10.31          00001010.00001010.00001010.00011 111
Hosts/Net: 6                     Class A, Private Internet

Address:   10.10.10.32          00001010.00001010.00001010.00100 000
Netmask:   255.255.255.248 = 29 11111111.11111111.11111111.11111 000
Wildcard:  0.0.0.7              00000000.00000000.00000000.00000 111
=>
Network:   10.10.10.32/29       00001010.00001010.00001010.00100 000
HostMin:   10.10.10.33          00001010.00001010.00001010.00100 001
HostMax:   10.10.10.38          00001010.00001010.00001010.00100 110
Broadcast: 10.10.10.39          00001010.00001010.00001010.00100 111
Hosts/Net: 6                     Class A, Private Internet


```

6. Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.

Ответ:

```text
Из диапазона 100.64.0.0 — 100.127.255.255 (маска подсети: 255.192.0.0 или /10) Carrier-Grade NAT
можно взять сеть 100.64.0.0/26 маска 255.255.255.192 позволит использовать внутри сети до 62 хостов.
```

7. Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?

Проверить ARP таблицу:
```text
Linux: ip neigh
Windows: arp -a
```
Очистить ARP кеш полностью:
```text
Linux: ip neigh flush
Windows: arp -d *
```
Удалить ARP таблицы только один нужный IP:
```text
Linux: ip neigh delete <IP> dev <INTERFACE>
Windows: arp -d <IP>
```

 ---
## Задание для самостоятельной отработки (необязательно к выполнению)

 8*. Установите эмулятор EVE-ng.
 
 Инструкция по установке - https://github.com/svmyasnikov/eve-ng

 Выполните задания на lldp, vlan, bonding в эмуляторе EVE-ng. 
 
 ---

## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.

Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: "1.1. Введение в DevOps — Сусанна Алиева".

Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка), иначе преподаватель не сможет проверить работу. Чтобы это проверить, откройте ссылку в браузере в режиме инкогнито.

[Как предоставить доступ к файлам и папкам на Google Диске](https://support.google.com/docs/answer/2494822?hl=ru&co=GENIE.Platform%3DDesktop)

[Как запустить chrome в режиме инкогнито ](https://support.google.com/chrome/answer/95464?co=GENIE.Platform%3DDesktop&hl=ru)

[Как запустить  Safari в режиме инкогнито ](https://support.apple.com/ru-ru/guide/safari/ibrw1069/mac)

Любые вопросы по решению задач задавайте в чате учебной группы.

---
