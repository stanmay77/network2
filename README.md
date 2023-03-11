# Домашнее задание к занятию «Компьютерные сети. Лекция 2»

1. Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?

В linux - ip link
В windows - ipconfig /all
В MacOS - ifconfig

На примере виртуальной машины:

```
vagrant@vagrant:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:59:cb:31 brd ff:ff:ff:ff:ff:ff
```

2. Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?

Протокол LLDP. Для установки:

```
sudo apt install lldpd
```

3. Какая технология используется для разделения L2-коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.

Используется технология VLAN (virtual local area network)

Для использования vlan нужно проверить загружен ли модуль 8021q ядра linux:

```
lsmod | grep 8021q
```

загружаем:
```
sudo modprobe 8021q
```
и проверияем:
```
$ lsmod | grep 8021q
8021q                  32768  0
garp                   16384  1 8021q
mrp                    20480  1 8021q
```

Теперь поставим пакет vlan:
```
sudo apt install vlan
```

и сконфигурируем интерфейс eth0.777:

```
sudo vconfig add eth0 777
sudo ip link set eth0.777 up
sudo ip a add 192.168.77.7/255.255.255.0 dev eth0.777
```

Как результат:

```
vagrant@vagrant:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:59:cb:31 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 83395sec preferred_lft 83395sec
    inet6 fe80::a00:27ff:fe59:cb31/64 scope link 
       valid_lft forever preferred_lft forever

3: eth0.777@eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:59:cb:31 brd ff:ff:ff:ff:ff:ff
    inet 192.168.77.7/24 scope global eth0.777
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe59:cb31/64 scope link 
       valid_lft forever preferred_lft forever
```

4. Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.


Типы аггрегации:
* balance-rr
* active-backup
* balance-xor
* broadcast
* 802.3ad
* balance-tlb
* balance-alb

Для начала проверяем загружен ли модуль bonding:

```
lsmod | grep bonding
```
загружаем:

```
sudo modprobe bonding
vagrant@vagrant:~$ lsmod | grep bonding
bonding               167936  0
```

Ставим утилиту ifenslave:

```
sudo apt install ifenslave
```

Пример конфига аггрегации интерфейсов в /etc/network/interfaces

```
iface bond0 inet static
    address 192.168.77.77
    netmask 255.255.255.0
    network 192.168.77.0
    gateway 74.201.77.10
    bond-slaves enp3s0 enp4s0
    bond-mode active-backup
    bond-miimon 100
    bond-downdelay 200
    bond-updelay 200
```

5. Сколько IP-адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.

В сети с маской /29 доступно 6 адресов для хостов.

В сети /24 можно организовать 32 подсети /29

Примеры:
10.10.10.24/29
10.10.10.32/29
10.10.10.40/29

6. Задача: вас попросили организовать стык между двумя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP-адреса? Маску выберите из расчёта — максимум 40–50 хостов внутри подсети.

100.64.0.0/26 на 62 хоста

7. Как проверить ARP-таблицу в Linux, Windows? Как очистить ARP-кеш полностью? Как из ARP-таблицы удалить только один нужный IP?

В windows: 
* Просмотр arp -a
* Удаление кэша arp -d
* Удаление IP адреса: arp -d xxx.xxx.xxx.xxx

В Linux:
* Просмотр arp
* Удаление кэша ip neigh flush all
* Удаление IP адреса: arp -d xxx.xxx.xxx.xxx

