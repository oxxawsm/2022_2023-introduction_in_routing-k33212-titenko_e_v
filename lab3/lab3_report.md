University: ITMO University

Faculty: FICT

Course: Introduction in routing

Year: 2022/2023

Group: K33212

Author: Titenko Elena Vitalevna

Lab: Lab3

Date of create: 2.12.2022

Date of finished: 16.12.2022


# Отчёт по лабораторной работе №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

***Цель:*** Изучить протоколы OSPF и MPLS, механизмы организации EoMPLS.

### Ход работы

#### Схема сети
![lab3_routing](https://user-images.githubusercontent.com/63160594/208048646-378a4cb6-a353-43dc-bc48-4d641cdb7b86.jpg)


1. Текст файла, который был использован для развертывания тестовой сети с расширением .yaml:

       name: lab3

       mgmt:
         network: statics
         ipv4_subnet: 172.20.24.0/24

       topology:

         nodes:
           R01.NY:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.2

           R01.LND:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.3

           R01.HKI:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.4

           R01.SPB:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.5

           R01.MSK:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.6

           R01.LBN:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.7

           PC1:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.8

           SGI.Prism:
             kind: vr-ros
             image: vrnetlab/vr-routeros:6.47.9
             mgmt_ipv4: 172.20.24.9


         links:
           - endpoints: ["SGI.Prism:eth1","R01.NY:eth1"]
           - endpoints: ["R01.NY:eth2","R01.LND:eth1"]
           - endpoints: ["R01.NY:eth3","R01.LBN:eth1"]
           - endpoints: ["R01.LND:eth2","R01.HKI:eth1"]
           - endpoints: ["R01.LBN:eth2","R01.HKI:eth2"]
           - endpoints: ["R01.LBN:eth3","R01.MSK:eth1"]
           - endpoints: ["R01.HKI:eth3","R01.SPB:eth1"]
           - endpoints: ["R01.SPB:eth2","R01.MSK:eth2"]
           - endpoints: ["R01.SPB:eth3","PC1:eth1"]


 
2. Конфигурации устройств:

**Роутер R01.NY**

       /interface bridge
       add name=EoMPLS_B
       add name=Lo0
       /interface vpls
       add cisco-style=yes cisco-style-id=100 disabled=no l2mtu=1500 mac-address=02:BD:27:52:DB:8E name=EoMPLS remote-peer=4.4.4.4
       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /routing ospf instance
       set [ find default=yes ] router-id=1.1.1.1
       /interface bridge port
       add bridge=EoMPLS_B interface=ether2
       add bridge=EoMPLS_B interface=EoMPLS
       /ip address
       add address=172.16.1.1/30 interface=ether3 network=172.16.1.0
       add address=172.16.2.1/30 interface=ether4 network=172.16.2.0
       add address=1.1.1.1 interface=Lo0 network=1.1.1.1
       /ip dhcp-client
       add disabled=no interface=ether1
       /mpls ldp
       set enabled=yes transport-address=1.1.1.1
       /mpls ldp interface
       add interface=ether3
       add interface=ether4
       /routing ospf network
       add area=backbone
       /system identity
       set name=R01.NY
   

**Роутер R01.LND**

       /interface bridge
       add name=Lo0
       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /routing ospf instance
       set [ find default=yes ] router-id=2.2.2.2
       /ip address
       add address=2.2.2.2 interface=Lo0 network=2.2.2.2
       add address=172.16.1.2/30 interface=ether2 network=172.16.1.0
       add address=172.16.3.1/30 interface=ether3 network=172.16.3.0
       /ip dhcp-client
       add disabled=no interface=ether1
       /mpls ldp
       set enabled=yes transport-address=2.2.2.2
       /mpls ldp interface
       add interface=ether2
       add interface=ether3
       /routing ospf network
       add area=backbone
       /system identity
       set name=R01.LND

  
**Роутер R01.HKI**

       /interface bridge
       add name=Lo0
       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /routing ospf instance
       set [ find default=yes ] router-id=3.3.3.3
       /ip address
       add address=172.16.3.2/30 interface=ether2 network=172.16.3.0
       add address=172.16.4.1/30 interface=ether3 network=172.16.4.0
       add address=172.16.5.1/30 interface=ether4 network=172.16.5.0
       add address=3.3.3.3 interface=Lo0 network=3.3.3.3
       /ip dhcp-client
       add disabled=no interface=ether1
       /mpls ldp
       set enabled=yes transport-address=3.3.3.3
       /mpls ldp interface
       add interface=ether4
       add interface=ether3
       add interface=ether2
       /routing ospf network
       add area=backbone
       /system identity
       set name=R01.HKI


**Роутер R01.SPB**

       /interface bridge
       add name=EoMPLS_B
       add name=Lo0
       /interface vpls
       add cisco-style=yes cisco-style-id=100 disabled=no l2mtu=1500 mac-address=02:96:AE:D1:9C:3F name=EoMPLS remote-peer=1.1.1.1
       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /routing ospf instance
       set [ find default=yes ] router-id=4.4.4.4
       /interface bridge port
       add bridge=EoMPLS_B interface=ether4
       add bridge=EoMPLS_B interface=EoMPLS
       /ip address
       add address=4.4.4.4 interface=Lo0 network=4.4.4.4
       add address=172.16.5.2/30 interface=ether2 network=172.16.5.0
       add address=172.16.6.1/30 interface=ether3 network=172.16.6.0
       /ip dhcp-client
       add disabled=no interface=ether1
       /mpls ldp
       set enabled=yes transport-address=4.4.4.4
       /mpls ldp interface
       add interface=ether3
       add interface=ether2
       add interface=ether4
       /routing ospf network
       add area=backbone
       /system identity
       set name=R01.SPB


**Роутер R01.MSK**

       /interface bridge
       add name=Lo0
       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /routing ospf instance
       set [ find default=yes ] router-id=5.5.5.5
       /ip address
       add address=172.16.7.1/30 interface=ether2 network=172.16.7.0
       add address=172.16.6.2/30 interface=ether3 network=172.16.6.0
       add address=5.5.5.5 interface=Lo0 network=5.5.5.5
       /ip dhcp-client
       add disabled=no interface=ether1
       /mpls ldp
       set enabled=yes transport-address=5.5.5.5
       /mpls ldp interface
       add interface=ether2
       add interface=ether3
       /routing ospf network
       add area=backbone
       /system identity
       set name=R01.MSK


**Роутер R01.LBN**

       /interface bridge
       add name=Lo0
       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /routing ospf instance
       set [ find default=yes ] router-id=6.6.6.6
       /ip address
       add address=6.6.6.6 interface=Lo0 network=6.6.6.6
       add address=172.16.2.2/30 interface=ether2 network=172.16.2.0
       add address=172.16.4.2/30 interface=ether3 network=172.16.4.0
       add address=172.16.7.2/30 interface=ether4 network=172.16.7.0
       /ip dhcp-client
       add disabled=no interface=ether1
       /mpls ldp
       set enabled=yes transport-address=6.6.6.6
       /mpls ldp interface
       add interface=ether4
       add interface=ether2
       add interface=ether3
       /routing ospf network
       add area=backbone
       /system identity
       set name=R01.LBN

    
**PC1**

       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /ip address
       add address=10.0.0.1/24 interface=ether2 network=10.0.0.0
       /ip dhcp-client
       add disabled=no interface=ether1
       /system identity
       set name=PC1

 
**SGI.Prism**

       /interface wireless security-profiles
       set [ find default=yes ] supplicant-identity=MikroTik
       /ip address
       add address=10.0.0.2/24 interface=ether2 network=10.0.0.0
       /ip dhcp-client
       add disabled=no interface=ether1
       /system identity
       set name=SGI.Prism
   
    
3. Пример пинга двух крайних хостов

![Снимок экрана от 2022-12-15 20-33-25](https://user-images.githubusercontent.com/63160594/207934755-288a032f-e6c7-40c4-a885-29b56f6b8317.png)


4. Информация об MPLS метках
![Снимок экрана от 2022-12-15 20-37-39](https://user-images.githubusercontent.com/63160594/207934894-848ca4c1-d1a1-4d5f-a084-dfbc6fe05d45.png)

5. Таблица маршрутизации на R01.NY
![Снимок экрана от 2022-12-15 20-35-50](https://user-images.githubusercontent.com/63160594/207935049-9ac2bfe2-6e44-49e6-8b6d-a0499263c7d6.png)



### Вывод
В ходе лабораторной работы №3 изучены протоколы OSPF и MPLS, механизмы организации EoMPLS; реализована IP/MPLS сеть связи для "RogaIKopita Games".
