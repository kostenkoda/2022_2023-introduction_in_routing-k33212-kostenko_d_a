# Отчет по лабораторной работе №1 "Установка ContainerLab и развертывание тестовой сети связи"
University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33212

Author: Kostenko Darina Alekseevna

Lab: Lab1

Date of create: 20.09.2022

Date of finished: 21.10.2022

**Цель работы:** ознакомиться с инструментом ContainerLab и методами работы с ним, изучить работу VLAN, IP адресации и т.д; развернуть тестовую сеть связи, настроить оборудование на базе Linux и RouterOS.

**Итог выполнения работы:**

1. Текст файла, который был использован для развертывания тестовой сети с расширением .yaml

```
name: lab1

mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24

topology:
  nodes:
    R01.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.11

    SW01.L3.01.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.21

    SW02.L3.01.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.31

    SW02.L3.02.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.32

    PC1:
      kind: linux
      image: ubuntu:latest
      mgmt_ipv4: 172.20.20.41

    PC2:
      kind: linux
      image: ubuntu:latest
      mgmt_ipv4: 172.20.20.42
  
  links:
    - endpoints: ["R01.TEST:eth1","SW01.L3.01.TEST:eth1"]
    - endpoints: ["SW01.L3.01.TEST:eth2","SW02.L3.01.TEST:eth1"]
    - endpoints: ["SW02.L3.01.TEST:eth2","PC1:eth1"]
    - endpoints: ["SW01.L3.01.TEST:eth3","SW02.L3.02.TEST:eth1"]
    - endpoints: ["SW02.L3.02.TEST:eth2","PC2:eth1"]
```

2. Схема связи

![](https://github.com/kostenkoda/2022_2023-introduction_in_routing-k33212-kostenko_d_a/blob/main/lab1/lab1.drawio.png)

3. Топология ContainerLab

![](https://github.com/kostenkoda/2022_2023-introduction_in_routing-k33212-kostenko_d_a/blob/main/lab1/containerlab_topology_lab1.jpeg)

4. Текст конфигураций сетевых устройств
- Роутер R01.TEST

```
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
add interface=ether2 name=vlan20 vlan-id=20
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool10 ranges=192.168.10.10-192.168.10.254
add name=pool20 ranges=192.168.20.10-192.168.20.254
/ip dhcp-server
add address-pool=pool10 disabled=no interface=vlan10 name=dhcp10
add address-pool=pool20 disabled=no interface=vlan20 name=dhcp20
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.1/24 interface=vlan10 network=192.168.10.0
add address=192.168.20.1/24 interface=vlan20 network=192.168.20.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=192.168.10.0/24 gateway=192.168.10.1
add address=192.168.20.0/24 gateway=192.168.20.1
/system identity
set name=R01.TEST
```

- Свитч первого уровня SW01.L3.01.TEST

```
/interface bridge
add name=bridge10
add name=bridge20
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
add interface=ether3 name=vlan10+ vlan-id=10
add interface=ether2 name=vlan20 vlan-id=20
add interface=ether4 name=vlan20+ vlan-id=20
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/interface bridge port
add bridge=bridge10 interface=vlan10
add bridge=bridge20 interface=vlan20
add bridge=bridge10 interface=vlan10+
add bridge=bridge20 interface=vlan20+
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=bridge10
add disabled=no interface=bridge20
/system identity
set name=SW01.L3.01.TEST
```

- Первый свитч второго уровня SW02.L3.01.TEST

```
/interface bridge
add name=bridge10
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/interface bridge port
add bridge=bridge10 interface=vlan10
add bridge=bridge10 interface=ether3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=bridge10
/system identity
set name=SW02.L3.01.TEST
```

- Второй свитч второго уровня SW02.L3.02.TEST

```
/interface bridge
add name=bridge20
/interface vlan
add interface=ether2 name=vlan20 vlan-id=20
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/interface bridge port
add bridge=bridge20 interface=vlan20
add bridge=bridge20 interface=ether3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=bridge20
/system identity
set name=SW02.L3.02.TEST

```

5. Результаты пингов для проверки локальной связности

- Проверка на роутере

![](https://github.com/kostenkoda/2022_2023-introduction_in_routing-k33212-kostenko_d_a/blob/main/lab1/router_check.jpeg)

- Проверка на свитче первого уровня

![](https://github.com/kostenkoda/2022_2023-introduction_in_routing-k33212-kostenko_d_a/blob/main/lab1/switch11_check.jpeg)

- Проверка на первом свитче второго уровня

![](https://github.com/kostenkoda/2022_2023-introduction_in_routing-k33212-kostenko_d_a/blob/main/lab1/switch21_check.jpeg)

- Проверка на втором свитче второго уровня

![](https://github.com/kostenkoda/2022_2023-introduction_in_routing-k33212-kostenko_d_a/blob/main/lab1/switch22_check.jpeg)

