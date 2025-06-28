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
