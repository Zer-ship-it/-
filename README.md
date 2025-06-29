Как работает настройка firewall и портов

hostnamectl hostname isp

exec bash

apt-get update

apt-get install mc

cd /etc/net/ifaces

ls

cp -r ens18 /etc/net/ifaces/ens19

cp -r ens18 /etc/net/ifaces/ens20

cd ens19

mcedit options

BOOTPROTO=static

F2 - Save F10 - exit

echo 172.16.4.1/28 > ipv4address

cd /etc/net/ifaces/ens20

mcedit options

BOOTPROTO=static

F2 - Save F10 - exit

echo 172.16.5.1/28 > ipv4address

cd

apt-get -y install firewalld

systemctl enable --now firewalld

firewall-cmd --permanent --zone=public --add-interface=ens18

firewall-cmd --permanent --zone=trusted --add-interface=ens19

firewall-cmd --permanent --zone=trusted --add-interface=ens20

firewall-cmd --permanent --zone=public --add-masquerade

firewall-cmd --complete-reload

systemctl restart network

ping ya.ru

ping ya.ru -I 172.16.4.1

ping ya.ru -I 172.16.5.1

Как настраивать порты на роутере 1

en

conf t

hostname hq-rtr.au-team.irpo

do sh port br

int isp

  ip address 172.16.4.14/28
  
  ex
  
int 100

  ip address 192.168.1.1/26
  
  ex
  
int 200

  ip address 192.168.1.65/28
  
  ex
  
int 999

  ip address 192.168.1.81/29
  
  ex
  
port te0

  service-instance isp
  
    encapsulation untagged
    
    connect ip interface isp
    
    ex
    
  ex
  
port te1

  service-instance 100
  
    encapsulation dot1q 100
    
    rewrite pop 1
    
    connect ip interface 100
    
    ex
    
  service-instance 200
  
    encapsulation dot1q 200
    
    rewrite pop 1
    
    connect ip interface 200
    
    ex
    
  service-instance 999
  
    encapsulation dot1q 999
    
    rewrite pop 1
    
    connect ip interface 999
    
    ex
    
  ex
  
ip route 0.0.0.0/0 172.16.4.1

ip name-server 77.88.8.8

ip nat pool INTERNET 192.168.1.1-192.168.1.87

ip nat source dynamic inside-to-outside pool INTERNET overload 172.16.4.14

int isp

  ip nat outside
  
  ex
  
int 100

  ip nat inside
  
  ex
  
int 200

  ip nat inside
  
  ex
  
int 999

  ip nat inside
  
  ex
  
ex

wr

Как настраивать порты на роутере 2

BR-RTR-Base

en

conf t

hostname br-rtr.au-team.irpo

do sh port br

int isp

  ip address 172.16.5.14/28
  
  ex
  
int lan

  ip address 192.168.2.1/27
  
  ex
  
port te0

  service-instance isp
  
    encapsulation untaged
    
    connect ip interface isp
    
    ex
    
  ex
  
port te1

    service-instance lan
    
    encapsulation untaged
    
    connect ip interface lan
    
    ex
    
  ex
  
ip route 0.0.0.0/0 172.16.5.1

ip name-server 77.88.8.8

ip nat pool INTERNET 192.168.2.1-192.168.2.30

ip nat source dynamic inside-to-outside pool INTERNET overload 172.16.5.14

int isp

  ip nat outside
  
  ex
  
int lan

  ip nat inside
  
  ex
  
ex

wr

Настройка тунеля роутер 1

en

conf t

int tunnel.1

  ip address 10.10.10.9/30
  
  ip tunnel 172.16.4.14 172.16.5.14 mode gre
  
  ex
  
ex

wr

Настройка тунеля роутер 2

en

conf t

int tunnel.1

  ip address 10.10.10.10/30
  
  ip tunnel 172.16.5.14 172.16.4.14 mode gre
  
  ex
  
ex

wr

Настройка ospf роутер 1

en

conf t

router ospf 1

  network 10.10.10.8/30 area 0
  
  network 192.168.1.0/26 area 0
  
  network 192.168.1.64/28 area 0
  
  network 192.168.1.80/29 area 0
  
  passive-interface isp
  
  passive-interface 100
  
  passive-interface 200
  
  passive-interface 999
  
  area 0 authentication message-digest

    ex
    
int tunnel.1

  ip ospf authentication-key P@ssw0rd
  
  ip ospf authentication message-digest
  
  ex
  
ex

wr

do sh ip route

Настройка ospf роутер 2

en

conf t

router ospf 1

  network 10.10.10.8/30 area 0
  
  network 192.168.2.0/27 area 0
  
  area 0 authentication message-digest
  
  passive-interface isp
  
  passive-interface lan
  
  ex
  
int tunnel.1

  ip ospf authentication-key P@ssw0rd
  
  ip ospf authentication message-digest
  
  ex
  
ex

wr

do sh ip route

Как настроить сервер ?

hostnamectl hostname hq-srv.au-team.irpo

exec bash

cd /etc/net/ifaces/ens18

echo 192.168.1.2/26 > ipv4address

echo default via 192.168.1.1 > ipv4route

echo nameserver 77.88.8.8 > resolv.conf

systemctl restart network

apt-get update

apt-get install bind bind-utils -y

cd /etc/bind

mcedit named.conf

<<!--Коментируем строку

#include "/etc/bind/rndc.conf";

перед строкой должен стоять знак #--!>>

F2 - Save F10 - exit

mcedit options.conf

<<!--Приводим к следующему виду--!>>

listen-on { 192.168.1.2; };

//listen-on-v6 { ::1; };

forwaders { 77.88.8.8; };

allow-query { any; };

allow-query-cache { any; };

allow-recursion { any; };

F2 - Save F10 - exit

mcedit local.conf

zone "au-team.irpo" {

  type master;
  
  file "/etc/bind/zone/db.au";
  
};

zone "1.168.192.in-addr.arpa" {

  type master;
  
  file "/etc/bind/zone/db.revers";
  
};

F2 - Save F10 - exit

cd zone

cp localhost db.au

mcedit db.au

<<!--Выполнить замену localhost на au-team.irpo

и 127.0.0.1 на 192.168.1.2--!>>

F2 - Save F10 - exit

cp db.au db.revers

chownd root:named db.*

mcedit db.au

<<!--Дописываем ниже--!>>

hq-srv  IN  A  192.168.1.2

hq-rtr  IN  A  192.168.1.1

br-srv  IN  A  192.168.2.2

bt-rtr  IN  A  192.168.2.1

hq-cli  IN  A  192.168.1.66

moodle  IN  CNAME  hq-rtr

wiki  IN  CNAME  hq-rtr

F2 - Save F10 - exit

mcedit db.revers

<<!--Дописываем ниже--!>>

1  IN  PTR  hq-rtr.au-team.irpo.

2  IN  PTR  hq-srv.au-team.irpo.

66  IN  PTR hq-cli.au-team.irpo.

F2 - Save F10 - exit

named-checkzone au-team.irpo db.au

named-checkzone 1.168.192.in-addr.arpa db.revers

systemctl restart bind

cd /etc/net/iface/ens19

mcedit resolv.conf

nameserver 192.168.1.2

search au-team.irpo

F2 - Save F10 - exit

systemctl restart network

systemctl restart bind

host hq-srv


host 192.168.1.2
host wiki
host ya.ru

Как запустить днс на роутере 1

en

conf t

ip name-server 192.168.1.2

ip domain-name au-team.irpo

ip domain-lookup

no ip name-server 77.88.8.8

do wr

do ping hq-srv

Как запустить днс на роутере 2

en

conf t

ip name-server 192.168.1.2

ip domain-name au-team.irpo

ip domain-lookup

no ip name-server 77.88.8.8

do wr

do ping hq-srv

Как запустить dhcp на роутере 1

en

conf t

ip pool hq 192.168.1.67-192.168.1.78

dhcp-server 1

  static ip 192.168.1.66
  
    client-id mac XX:XX:XX:XX:XX:XX (MAC адрес HQ-Cli)
    
    mask 255.255.255.240
    
    gateway 192.168.1.65
    
    dns 192.168.1.2
    
    domain_search au-team.irpo
    
    ex
    
  pool hq 1
  
  mask 255.255.255.240
  
  gateway 192.168.165
  
  dns 192.168.1.2

  domain_search au-team.irpo
  
do wr

int lan

int 200

  dhcp-server 1
  
  do wr
