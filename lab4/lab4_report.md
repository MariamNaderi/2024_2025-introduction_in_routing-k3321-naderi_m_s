University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2024/2025

Group: K3321

Author: Naderi Mariam Shakhovna

Lab: Lab4

Date of create: 17.12.2024

Date of finished: 

# Лабораторная работ №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

`Цель работы:`
Изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.

## Ход работы

Работа выполнялась на ubuntu в виртуальной машине. 

После установки всех необходимых компонентов, с помощью файла .yaml была задана топология корпоративной сети связи для компании "RogaIKopita Games":

<img src="./pic/topology.png">

Далее были настроены все роутеры с помощью следующих команд:
(ssh admin@[ip-адрес]; пароль -- admin)

## Часть 1

### RO1_NY:
```
/interface bridge
add name=loopback

/ip address
add address=3.3.1.1/30 interface=ether3 
add address=192.168.40.10/24 interface=ether4 
add address=192.168.10.1/32 interface=loopback 


/routing ospf instance
set [ find default=yes ] router-id=192.168.10.1
/routing ospf network
add area=backbone

/mpls ldp
set enabled=yes lsr-id=192.168.10.1 transport-address=192.168.10.1
/mpls ldp interface
add interface=ether3


/ip route vrf
add export-route-targets=65500:333 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65500:333 routing-mark=VRF_DEVOPS

/routing bgp instance
set default as=65500 router-id=192.168.10.1
/routing bgp instance vrf
add redistribute-connected=yes redistribute-ospf=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=vpnv4 name=peer_LND remote-address=192.168.10.2 remote-as=\
    65500 update-source=loopback
```

### RO1_LND:
```
/interface bridge
add name=loopback

/ip address
add address=3.3.1.2/30 interface=ether3 
add address=3.3.2.1/30 interface=ether4 
add address=3.3.3.1/30 interface=ether5 
add address=192.168.10.2 interface=loopback 

/routing ospf instance
set [ find default=yes ] router-id=192.168.10.2
/routing ospf network
add area=backbone

/mpls ldp
set enabled=yes lsr-id=192.168.10.2 transport-address=192.168.10.2
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5

/routing bgp instance
set default as=65500 router-id=192.168.10.2
/routing bgp peer
add address-families=vpnv4 name=peer_NY remote-address=192.168.10.1 remote-as=\
    65500 update-source=loopback
add address-families=vpnv4 name=peer_LBN remote-address=192.168.10.3 remote-as=\
    65500 update-source=loopback
add address-families=vpnv4 name=peer_HKI remote-address=192.168.10.5 remote-as=\
    65500 update-source=loopback
```

### RO1_LBN:
```
/interface bridge
add name=loopback

/ip address
add address=3.3.2.2/30 interface=ether3 
add address=3.3.5.2/30 interface=ether4 
add address=3.3.6.1/30 interface=ether5 
add address=192.168.10.3 interface=loopback 

/routing ospf instance
set [ find default=yes ] router-id=192.168.10.3
/routing ospf network
add area=backbone

/mpls ldp
set enabled=yes lsr-id=192.168.10.3 transport-address=192.168.10.3
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5

/routing bgp instance
set default as=65500 router-id=192.168.10.3
/routing bgp peer
add address-families=vpnv4 name=peer_SVL remote-address=192.168.10.4 remote-as=\
    65500 update-source=loopback
add address-families=vpnv4 name=peer_LND remote-address=192.168.10.2 remote-as=\
    65500 update-source=loopback
add address-families=vpnv4 name=peer_HKI remote-address=192.168.10.5 remote-as=\
    65500 update-source=loopback
```

### RO1_SVL:
```
/interface bridge
add name=loopback

/ip address
add address=3.3.6.2/30 interface=ether3 
add address=192.168.50.10/24 interface=ether4 
add address=192.168.10.4/32 interface=loopback 


/routing ospf instance
set [ find default=yes ] router-id=192.168.10.4
/routing ospf network
add area=backbone

/mpls ldp
set enabled=yes lsr-id=192.168.10.4 transport-address=192.168.10.4
/mpls ldp interface
add interface=ether3


/ip route vrf
add export-route-targets=65500:333 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65500:333 routing-mark=VRF_DEVOPS

/routing bgp instance
set default as=65500 router-id=192.168.10.4
/routing bgp instance vrf
add redistribute-connected=yes redistribute-ospf=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=vpnv4 name=peer_LBN remote-address=192.168.10.3 remote-as=\
    65500 update-source=loopback
```

### RO1_HKI:
```
/interface bridge
add name=loopback

/ip address
add address=3.3.3.2/30 interface=ether3 
add address=3.3.5.1/30 interface=ether4 
add address=3.3.4.1/30 interface=ether5 
add address=192.168.10.5 interface=loopback 

/routing ospf instance
set [ find default=yes ] router-id=192.168.10.5
/routing ospf network
add area=backbone

/mpls ldp
set enabled=yes lsr-id=192.168.10.5 transport-address=192.168.10.5
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5

/routing bgp instance
set default as=65500 router-id=192.168.10.5
/routing bgp peer
add address-families=vpnv4 name=peer_LND remote-address=192.168.10.2 remote-as=\
    65500 update-source=loopback
add address-families=vpnv4 name=peer_LBN remote-address=192.168.10.3 remote-as=\
    65500 update-source=loopback
add address-families=vpnv4 name=peer_SPB remote-address=192.168.10.6 remote-as=\
    65500 update-source=loopback
```

### RO1_SPB:
```
/interface bridge
add name=loopback

/ip address
add address=3.3.4.2/30 interface=ether3 
add address=192.168.30.10/24 interface=ether4 
add address=192.168.10.6/32 interface=loopback 


/routing ospf instance
set [ find default=yes ] router-id=192.168.10.6
/routing ospf network
add area=backbone

/mpls ldp
set enabled=yes lsr-id=192.168.10.6 transport-address=192.168.10.6
/mpls ldp interface
add interface=ether3


/ip route vrf
add export-route-targets=65500:333 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65500:333 routing-mark=VRF_DEVOPS

/routing bgp instance
set default as=65500 router-id=192.168.10.6
/routing bgp instance vrf
add redistribute-connected=yes redistribute-ospf=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=vpnv4 name=peer_HKI remote-address=192.168.10.5 remote-as=\
    65500 update-source=loopback
```

### PC1:
```
ip add add 100.1.10.10/24 dev eth0
```

### SGI_Prims:
```
ip add add 100.1.20.10/24 dev eth0
```


### Результаты пингов: 

### PC -> SGI Prims:

<img src="./pic/pic7.PNG" style="width:450px;">

### SGI Prims -> PC:

<img src="./pic/pic8.PNG" style="width:450px;">


### Проверки протоколов: 

### NY:

<img src="./pic/pic1.PNG" style="width:1450px;">

### LND:

<img src="./pic/pic2.PNG" style="width:1450px;">

### LBN:

<img src="./pic/pic3.PNG" style="width:1450px;">

### HKI:

<img src="./pic/pic4.PNG" style="width:1450px;">

### MSK:

<img src="./pic/pic5.PNG" style="width:1450px;">

### SPB:

<img src="./pic/pic6.PNG" style="width:1450px;">


### Трассировка:

<img src="./pic/trace.PNG" style="width:1450px;">
