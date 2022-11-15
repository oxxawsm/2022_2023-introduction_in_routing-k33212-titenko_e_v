University: ITMO University

Faculty: FICT

Course: Introduction in routing

Year: 2022/2023

Group: K33212

Author: Titenko Elena Vitalevna

Lab: Lab2

Date of create: 1.11.2022

Date of finished: 18.11.2022

# Отчёт по лабораторной работе №2 "Эмуляция распределенной корпоративной сети связи, настройка статической маршрутизации между филиалами"

***Цель:*** Ознакомиться с принципами планирования IP адресов, настройке статической маршрутизации и сетевыми функциями устройств.

### Ход работы

1. Текст файла, который был использован для развертывания тестовой сети с расширением .yaml:

        name: lab2

        mgmt: 
          network: statics
          ipv4_subnet: 172.20.22.0/24


        topology:

          nodes:
            R01.MSK:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.22.2

            R01.FRT:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.22.3

            R01.BRL:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.22.4

            PC1:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.22.5

            PC2:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.22.6

            PC3:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.22.7


          links:
            - endpoints: ["R01.BRL:eth2", "R01.FRT:eth1"]
            - endpoints: ["R01.FRT:eth2", "R01.MSK:eth2"]
            - endpoints: ["R01.MSK:eth1", "R01.BRL:eth1"]
            - endpoints: ["R01.BRL:eth3", "PC3:eth1"]
            - endpoints: ["R01.FRT:eth3", "PC2:eth1"]
            - endpoints: ["R01.MSK:eth3", "PC1:eth1"]
 
2. Конфигурации устройств:

**Роутер R01.MSK**

    /ip pool
    add name=pool1 ranges=192.168.10.10-192.168.10.254
    /ip dhcp-server
    add address-pool=pool1 disabled=no interface=ether4 name=dhcp1
    /ip address
    add address=172.15.255.30/30 interface=ether1 network=172.15.255.28
    add address=10.10.1.1/30 interface=ether2 network=10.10.1.0
    add address=10.10.2.1/30 interface=ether3 network=10.10.2.0
    add address=192.168.10.1/24 interface=ether4 network=192.168.10.0
    /ip dhcp-client
    add disabled=no interface=ether1
    /ip route
    add distance=1 dst-address=192.168.20.0/24 gateway=10.10.2.2
    add distance=1 dst-address=192.168.30.0/24 gateway=10.10.1.2
    /system identity
    set name=R01.MSK

**Роутер R01.FRT**

    /ip pool
    add name=pool2 ranges=192.168.20.10-192.168.20.254
    /ip dhcp-server
    add address-pool=pool2 disabled=no interface=ether4 name=dhcp2
    /ip address
    add address=172.15.255.30/30 interface=ether1 network=172.15.255.28
    add address=10.10.2.2/30 interface=ether3 network=10.10.2.0
    add address=10.10.3.1/30 interface=ether2 network=10.10.3.0
    add address=192.168.20.1/24 interface=ether4 network=192.168.20.0
    /ip dhcp-client
    add disabled=no interface=ether1
    /ip route
    add distance=1 dst-address=192.168.10.0/24 gateway=10.10.2.1
    add distance=1 dst-address=192.168.30.0/24 gateway=10.10.3.2
    /system identity
    set name=R01.FRT
    
**Роутер R01.BRL**

    /ip pool
    add name=pool3 ranges=192.168.30.10-192.168.30.254
    /ip dhcp-server
    add address-pool=pool3 disabled=no interface=ether4 name=dhcp3
    /ip address
    add address=172.15.255.30/30 interface=ether1 network=172.15.255.28
    add address=10.10.1.2/30 interface=ether2 network=10.10.1.0
    add address=10.10.3.2/30 interface=ether3 network=10.10.3.0
    add address=192.168.30.1/24 interface=ether4 network=192.168.30.0
    /ip dhcp-client
    add disabled=no interface=ether1
    /ip route
    add distance=1 dst-address=192.168.10.0/24 gateway=10.10.1.1
    add distance=1 dst-address=192.168.20.0/24 gateway=10.10.3.1
    /system identity
    set name=R01.BRL
    
**PC1**

    /ip address
    add address=172.15.255.30/30 interface=ether1 network=172.15.255.28
    /ip dhcp-client
    add disabled=no interface=ether1
    add disabled=no interface=ether2
    /ip route
    add distance=1 dst-address=10.10.1.0/30 gateway=192.168.10.1
    add distance=1 dst-address=10.10.2.0/30 gateway=192.168.10.1
    add distance=1 dst-address=192.168.20.0/24 gateway=192.168.10.1
    add distance=1 dst-address=192.168.30.0/24 gateway=192.168.10.1
    /system identity
    set name=PC1
 
**PC2**

    /ip address
    add address=172.15.255.30/30 interface=ether1 network=172.15.255.28
    /ip dhcp-client
    add disabled=no interface=ether1
    add disabled=no interface=ether2
    /ip route
    add distance=1 dst-address=10.10.2.0/30 gateway=192.168.20.1
    add distance=1 dst-address=10.10.3.0/30 gateway=192.168.20.1
    add distance=1 dst-address=192.168.10.0/24 gateway=192.168.20.1
    add distance=1 dst-address=192.168.30.0/24 gateway=192.168.20.1
    /system identity
    set name=PC2
    
**PC3**

    /ip address
    add address=172.15.255.30/30 interface=ether1 network=172.15.255.28
    /ip dhcp-client
    add disabled=no interface=ether1
    add disabled=no interface=ether2
    /ip route
    add distance=1 dst-address=10.10.1.0/30 gateway=192.168.30.1
    add distance=1 dst-address=10.10.3.0/30 gateway=192.168.30.1
    add distance=1 dst-address=192.168.10.0/24 gateway=192.168.30.1
    add distance=1 dst-address=192.168.20.0/24 gateway=192.168.30.1
    /system identity
    set name=PC3
    
3. Примеры пингов, проверка работоспособности сети
![Снимок экрана от 2022-11-15 16-22-09](https://user-images.githubusercontent.com/63160594/201935116-6d9ba29c-9bd4-4112-8f65-59507309188b.png)

![Снимок экрана от 2022-11-15 16-23-51](https://user-images.githubusercontent.com/63160594/201935212-004234e2-60f8-4544-b85f-5f6531c61fe9.png)

![Снимок экрана от 2022-11-15 16-23-22](https://user-images.githubusercontent.com/63160594/201935266-d81e6619-5180-4f49-8639-51b56a325d4d.png)


### Вывод
В ходе лабораторной работы №2 я познакомилась с принципами планирования IP адресов, настройке статической маршрутизации и сетевыми функциями устройств, настроила статические маршруты для трёх ПК, находящихся в разных подсетях, связанных тремя роутерами, построила схему сети.
