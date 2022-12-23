University: ITMO University

Faculty: FICT

Course: Introduction in routing

Year: 2022/2023

Group: K33212

Author: Titenko Elena Vitalevna

Lab: Lab3

Date of create: 16.12.2022

Date of finished: 26.12.2022


# Отчёт по лабораторной работе №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

***Цель:*** Изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.

### Ход работы

#### Схема сети

1. Текст файла, который был использован для развертывания тестовой сети с расширением .yaml:

        name: lab4

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

            R01.LBN:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.24.6

            R01.SVL:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.24.7

            PC1:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.24.8

            PC2:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.24.9

            PC3:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.20.24.10   


          links:
            - endpoints: ["PC2:eth1","R01.NY:eth1"]
            - endpoints: ["R01.NY:eth2","R01.LND:eth1"]
            - endpoints: ["R01.LND:eth2","R01.HKI:eth1"]
            - endpoints: ["R01.HKI:eth2","R01.SPB:eth1"]
            - endpoints: ["R01.SPB:eth2","PC1:eth1"]
            - endpoints: ["R01.LND:eth3","R01.LBN:eth1"]
            - endpoints: ["R01.HKI:eth3","R01.LBN:eth1"]
            - endpoints: ["R01.LBN:eth2","R01.SVL:eth1"]
            - endpoints: ["R01.SVL:eth2","PC3:eth1"]


2. Конфигурации устройств:

**Роутер R01.NY**

        /interface bridge
        add name=L0
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default client-to-client-reflection=no
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=10.0.10.1 interface=L0 network=10.0.10.1
        add address=172.16.1.1 interface=ether3 network=172.16.1.0
        add address=192.168.66.1 interface=ether2 network=192.168.66.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /ip route vrf
        add export-route-targets=65530:666 import-route-targets=655:666 interfaces=ether2 route-distinguisher=65530:666 \
            routing-mark=VRF_DEVOPS
        /mpls ldp
        set enabled=yes lsr-id=10.0.10.1 transport-address=10.0.10.1
        /mpls ldp interface
        add interface=ether3
        /routing bgp instance vrf
        add redistribute-connected=yes routing-mark=VRF_DEVOPS
        /routing bgp network
        add network=172.16.0.0/16
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.0.11.1 remote-as=65530 update-source=L0
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.NY



**Роутер R01.LND**


**Роутер R01.HKI**


**Роутер R01.SPB**


**Роутер R01.LBN**


**Роутер R01.SVL**


**PC1**


**PC2**


**PC3**


3. Примеры пингов:



### Вывод
