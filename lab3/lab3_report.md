University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2024/2025

Group: K3321

Author: Naderi Mariam Shakhovna

Lab: Lab3

Date of create: 11.11.2024

Date of finished: 13.11.2024

# Лабораторная работ №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

`Цель работы:`
Изучить протоколы OSPF и MPLS, механизмы организации EoMPLS.

## Ход работы

Работа выполнялась на ubuntu в виртуальной машине. 

После установки всех необходимых компонентов, с помощью файла .yaml была задана топология корпоративной сети связи:

<img src="./pic/topology.png">

Далее были настроены все роутеры с помощью следующих команд:
(ssh admin@[ip-адрес]; пароль -- admin)


### RO1_NY:
```
/interface bridge
add name=loopback
add name=EoMPLS_br

/ip address
add address=1.1.1.1/30 interface=ether3
add address=1.1.2.1/30 interface=ether4
add address=1.1.8.1/30 interface=ether5
add address=192.168.10.1/32 interface=loopback
add address=100.1.3.1/24 interface=EoMPLS_br 

/mpls ldp
set enabled=yes lsr-id=192.168.10.1 transport-address=192.168.10.1
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5

/routing ospf instance
set 0 router-id=192.168.10.1

/routing ospf network
add area=backbone

/interface vpls
add name=EoMPLS vpls-id=300:300 remote-peer=192.168.10.6 disabled=no

/interface bridge port
add bridge=EoMPLS_br interface=ether5
add bridge=EoMPLS_br interface=EoMPLS
```

### RO1_LND:
```
/interface bridge
add name=loopback

/ip address
add address=1.1.1.2/30 interface=ether3
add address=1.1.3.1/30 interface=ether4
add address=192.168.10.2/32 interface=loopback

/mpls ldp
set enabled=yes lsr-id=192.168.10.2 transport-address=192.168.10.2
/mpls ldp interface
add interface=ether3
add interface=ether4

/routing ospf instance
set 0 router-id=192.168.10.2
/routing ospf network
add area=backbone
```

### RO1_LBN:
```
/interface bridge
add name=loopback

/ip address
add address=1.1.5.1/30 interface=ether3
add address=1.1.2.2/30 interface=ether4
add address=1.1.4.1/30 interface=ether5
add address=192.168.10.3/32 interface=loopback

/mpls ldp
set enabled=yes lsr-id=192.168.10.3 transport-address=192.168.10.3
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5

/routing ospf instance
set 0 router-id=192.168.10.3
/routing ospf network
add area=backbone
```

### RO1_HKI:
```
/interface bridge
add name=loopback

/ip address
add address=1.1.6.1/30 interface=ether3
add address=1.1.3.2/30 interface=ether4
add address=1.1.4.2/30 interface=ether5
add address=192.168.10.4/32 interface=loopback

/mpls ldp
set enabled=yes lsr-id=192.168.10.4 transport-address=192.168.10.4
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5

/routing ospf instance
set 0 router-id=192.168.10.4
/routing ospf network
add area=backbone
```

### RO1_MSK:
```
/interface bridge
add name=loopback

/ip address
add address=1.1.5.2/30 interface=ether3
add address=1.1.7.1/30 interface=ether4
add address=192.168.10.5/32 interface=loopback

/mpls ldp
set enabled=yes lsr-id=192.168.10.5 transport-address=192.168.10.5
/mpls ldp interface
add interface=ether3
add interface=ether4

/routing ospf instance
set 0 router-id=192.168.10.5
/routing ospf network
add area=backbone
```

### RO1_SPB:
```
/interface bridge
add name=loopback
add name=EoMPLS_br

/ip address
add address=1.1.6.2/30 interface=ether3
add address=1.1.7.2/30 interface=ether4
add address=1.1.9.1/30 interface=ether5
add address=192.168.10.6/32 interface=loopback
add address=100.1.4.1/24 interface=EoMPLS_br

/mpls ldp
set enabled=yes lsr-id=192.168.10.5 transport-address=192.168.10.6
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5

/routing ospf instance
set 0 router-id=192.168.10.6
/routing ospf network
add area=backbone

/interface vpls
add name=EoMPLS vpls-id=300:300 remote-peer=192.168.10.1 disabled=no
/interface bridge port
add bridge=EoMPLS_br interface=ether5
add bridge=EoMPLS_br interface=EoMPLS
```

### PC1:
```
ip add add 100.1.1.2/24 dev eth0
```

### SGI_Prims:
```
ip add add 100.1.2.2/24 dev eth0
```


### Результаты пингов: 

На скриншотах видно, что PC имеют друг к другу доступ благодаря статической маршрутизации.

### PC1:

<img src="./pic/PC1_ping.jpg" style="width:350px;">

### PC2:

<img src="./pic/PC2_ping.jpg" style="width:350px;">

### PC3:

<img src="./pic/PC3_ping.jpg" style="width:350px;">
