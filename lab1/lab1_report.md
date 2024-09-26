University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2024/2025

Group: K3321

Author: Naderi Mariam Shakhovna

Lab: Lab1

Date of create: 25.09.2024

Date of finished: 28.09.2023

# Лабораторная работ №1 "Установка ContainerLab и развертывание тестовой сети связи"

`Цель работы:`
Ознакомиться с инструментом ContainerLab и методами работы с ним, изучить работу VLAN, IP адресации и т.д.

## Ход работы

Работа выполнялась на ubuntu в виртуальной машине. 

После установки всех необходимых компонентов, с помощью файла .yaml была задана топология трехуровневой сети связи классического предприятия:

<img src="./pic/topology.jpg">

Далее были настроены роутер и коммутаторы с помощью следующих команд:
(ssh admin@[ip-адрес]; пароль -- admin)

### RO:
```
/interface vlan
add name=RO_vlan10 vlan-id=10 interface=ether4 disabled=no
add name=RO_vlan20 vlan-id=20 interface=ether4 disabled=no

/ip address
add address=192.168.10.1/24 interface=RO_vlan10 
add address=192.168.20.1/24 interface=RO_vlan20 

/ip dhcp-server network
add address=192.168.10.0/24 gateway=192.168.10.1
add address=192.168.20.0/24 gateway=192.168.20.1

/ip pool
add name=pool_vlan10 ranges=192.168.10.10-192.168.10.250
add name=pool_vlan20 ranges=192.168.20.10-192.168.20.250

/ip dhcp-server
add name=server10 address-pool=pool_vlan10 interface=RO_vlan10 disabled=no
add name=server20 address-pool=pool_vlan20 interface=RO_vlan20 disabled=no
```

### SW1:
```
/interface vlan
add name=SW1_vlan10_1 vlan-id=10 interface=ether4 disabled=no
add name=SW1_vlan10_2 vlan-id=10 interface=ether5 disabled=no
add name=SW1_vlan20_1 vlan-id=20 interface=ether4 disabled=no
add name=SW1_vlan20_2 vlan-id=20 interface=ether6 disabled=no

/interface bridge
add name=SW1_br10
add name=SW1_br20

/interface bridge port
add interface=SW1_vlan10_1 bridge=SW1_br10
add interface=SW1_vlan10_2 bridge=SW1_br10
add interface=SW1_vlan20_1 bridge=SW1_br20
add interface=SW1_vlan20_2 bridge=SW1_br20

/ip dhcp-client
add interface=SW1_br10 disabled=no
add interface=SW1_br20 disabled=no
```

### SW2_1:
```
/interface vlan
add name=SW2_1_vlan10 vlan-id=10 interface=ether4 disabled=no

/interface bridge
add name=SW2_1_br10

/interface bridge port
add interface=SW2_1_vlan10 bridge=SW2_1_br10
add interface=ether5 bridge=SW2_1_br10

/ip dhcp-client
add interface=SW2_1_br10 disabled=no
```

### SW2_2:
```
/interface vlan
add name=SW2_2_vlan20 vlan-id=20 interface=ether4 disabled=no

/interface bridge
add name=SW2_2_br20

/interface bridge port
add interface=SW2_2_vlan20 bridge=SW2_1_br20
add interface=ether5 bridge=SW2_1_br20

/ip dhcp-client
add interface=SW2_2_br20 disabled=no
```

Настройка PC:
(docker exec -it [id docker] sh)

```
udhcpc -i eth3
ifconfig eth3 up
```

### Результаты пингов: 

На скриншотах видно, что PC имеют друг к другу доступ благодаря маршрутизации, при этом SW2_1 (vlan10) не имеет доступа к PC2 (vlan20), а SW2_2 (vlan20) к PC1 (vlan10).

### RO:

<img src="./pic/RO_ping.jpg" style="width:350px;">

### SW1:

<img src="./pic/SW1_ping.jpg" style="width:350px;">

### SW2_1:

<img src="./pic/SW2_1_ping.jpg" style="width:350px;">

### SW2_2:

<img src="./pic/SW2_2_ping.jpg" style="width:350px;">

### PC1:

<img src="./pic/PC1_ping.jpg" style="width:350px;">

### PC2:

<img src="./pic/PC2_ping.jpg" style="width:350px;">

