University: ITMO University

Faculty: FICT

Course: Introduction in routing

Year: 2022/2023

Group: K33212

Author: Titenko Elena Vitalevna

Lab: Lab3

Date of create: 16.12.2022

Date of finished: 13.01.2023


# Отчёт по лабораторной работе №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

***Цель:*** Изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.

### Ход работы

#### Схема сети

![lab4_routing](https://user-images.githubusercontent.com/63160594/209479654-7051e134-9328-4831-8d43-a136495b594a.jpg)


1. Текст файла, который был использован для развертывания тестовой сети с расширением .yaml:

        name: lab4

        mgmt:
          network: statics
          ipv4_subnet: 172.21.24.0/24

        topology:

          nodes:
            R01.NY:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.2

            R01.LND:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.3

            R01.HKI:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.4

            R01.SPB:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.5

            R01.LBN:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.6

            R01.SVL:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.7

            PC1:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.8

            PC2:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.9

            PC3:
              kind: vr-ros
              image: vrnetlab/vr-routeros:6.47.9
              mgmt_ipv4: 172.21.24.10   


          links:
            - endpoints: ["PC2:eth1","R01.NY:eth1"]
            - endpoints: ["R01.NY:eth2","R01.LND:eth1"]
            - endpoints: ["R01.LND:eth2","R01.HKI:eth1"]
            - endpoints: ["R01.HKI:eth2","R01.SPB:eth1"]
            - endpoints: ["R01.SPB:eth2","PC1:eth1"]
            - endpoints: ["R01.LND:eth3","R01.LBN:eth1"]
            - endpoints: ["R01.HKI:eth3","R01.LBN:eth2"]
            - endpoints: ["R01.LBN:eth3","R01.SVL:eth1"]
            - endpoints: ["R01.SVL:eth2","PC3:eth1"]


#### Часть первая

2. Конфигурации устройств:

**Роутер R01.NY**

        /interface bridge
        add name=Lo
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default redistribute-connected=yes router-id=1.1.1.1
        /routing ospf instance
        set [ find default=yes ] router-id=1.1.1.1
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=1.1.1.1 interface=Lo network=1.1.1.1
        add address=172.16.1.1/30 interface=ether3 network=172.16.1.0
        add address=192.168.20.1/30 interface=ether2 network=192.168.20.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /ip route vrf
        add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether2 \
            route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether3
        /routing bgp instance vrf
        add redistribute-connected=yes routing-mark=VRF_DEVOPS
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.NY


**Роутер R01.LND**

        /interface bridge
        add name=Lo
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default router-id=2.2.2.2
        /routing ospf instance
        set [ find default=yes ] router-id=2.2.2.2
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=2.2.2.2 interface=Lo network=2.2.2.2
        add address=172.16.1.2/30 interface=ether2 network=172.16.1.0
        add address=172.16.2.1/30 interface=ether3 network=172.16.2.0
        add address=172.16.3.1/30 interface=ether4 network=172.16.3.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether2
        add interface=ether3
        add interface=ether4
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=5.5.5.5 remote-as=65530 \
            route-reflect=yes update-source=Lo
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=3.3.3.3 remote-as=65530 \
            route-reflect=yes update-source=Lo
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=1.1.1.1 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.LND


**Роутер R01.HKI**

        /interface bridge
        add name=Lo
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing ospf instance
        set [ find default=yes ] router-id=3.3.3.3
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=172.16.2.2/30 interface=ether2 network=172.16.2.0
        add address=3.3.3.3 interface=Lo network=3.3.3.3
        add address=172.16.5.1/30 interface=ether3 network=172.16.5.0
        add address=172.16.4.1/30 interface=ether4 network=172.16.4.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether2
        add interface=ether3
        add interface=ether4
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 \
            route-reflect=yes update-source=Lo
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=5.5.5.5 remote-as=65530 \
            route-reflect=yes update-source=Lo
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=4.4.4.4 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.HKI


**Роутер R01.SPB**

        /interface bridge
        add name=Lo
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default router-id=4.4.4.4
        /routing ospf instance
        set [ find default=yes ] router-id=4.4.4.4
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=4.4.4.4 interface=Lo network=4.4.4.4
        add address=172.16.5.2/30 interface=ether2 network=172.16.5.0
        add address=192.168.10.1/30 interface=ether3 network=192.168.10.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /ip route vrf
        add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether3 \
            route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether2
        /routing bgp instance vrf
        add redistribute-connected=yes routing-mark=VRF_DEVOPS
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=3.3.3.3 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.SPB


**Роутер R01.LBN**

        /interface bridge
        add name=Lo
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing ospf instance
        set [ find default=yes ] router-id=5.5.5.5
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=5.5.5.5 interface=Lo network=5.5.5.5
        add address=172.16.3.2/30 interface=ether2 network=172.16.3.0
        add address=172.16.6.1/30 interface=ether4 network=172.16.6.0
        add address=172.16.4.2/30 interface=ether3 network=172.16.4.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether2
        add interface=ether3
        add interface=ether4
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 \
            route-reflect=yes update-source=Lo
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=3.3.3.3 remote-as=65530 \
            route-reflect=yes update-source=Lo
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=6.6.6.6 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.LBN



**Роутер R01.SVL**

        /interface bridge
        add name=Lo
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default redistribute-connected=yes
        /routing ospf instance
        set [ find default=yes ] router-id=6.6.6.6
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=6.6.6.6 interface=Lo network=6.6.6.6
        add address=172.16.6.2/30 interface=ether2 network=172.16.6.0
        add address=192.168.30.1/30 interface=ether3 network=192.168.30.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /ip route vrf
        add export-route-targets=65530:100 import-route-targets=65530:100 interfaces=ether3 \
            route-distinguisher=65530:100 routing-mark=VRF_DEVOPS
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether2
        /routing bgp instance vrf
        add redistribute-connected=yes routing-mark=VRF_DEVOPS
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=5.5.5.5 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.SVL

3. Проверка связности по VRF:

![Снимок экрана от 2022-12-25 15-20-13](https://user-images.githubusercontent.com/63160594/209468117-637e24ca-1ecd-41ef-a177-10d554ae079f.png)

![Снимок экрана от 2022-12-25 15-24-09](https://user-images.githubusercontent.com/63160594/209468127-88270f56-da83-491f-a612-c9b754b8d110.png)

![Снимок экрана от 2022-12-25 21-08-41](https://user-images.githubusercontent.com/63160594/209478173-f4122013-a577-4740-8a58-8ec8e7fcaf7f.png)

![Снимок экрана от 2022-12-25 21-07-36](https://user-images.githubusercontent.com/63160594/209478176-89161aab-b2ce-4d31-94ca-4d716e6ff2fe.png)

![Снимок экрана от 2022-12-25 21-09-35](https://user-images.githubusercontent.com/63160594/209478178-fa8752fc-59c9-4061-bb96-8b51d2130c14.png)



#### Часть вторая       

1. Для настройки VPLS на роутерах с интерфейсов был снят VRF.
2. На роутерах NY, SPB и SVL настроен VPLS:

**Роутер R01.NY**

        /interface bridge
        add name=Lo
        add name=VPLS
        /interface vpls
        add disabled=no l2mtu=1500 mac-address=02:3F:7D:D0:E8:46 name=vpls1 remote-peer=4.4.4.4 vpls-id=10:0
        add disabled=no l2mtu=1500 mac-address=02:FD:62:EA:D0:A9 name=vpls2 remote-peer=6.6.6.6 vpls-id=10:0
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default router-id=1.1.1.1
        /routing ospf instance
        set [ find default=yes ] router-id=1.1.1.1
        /interface bridge port
        add bridge=VPLS interface=ether2
        add bridge=VPLS interface=vpls1
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=1.1.1.1 interface=Lo network=1.1.1.1
        add address=172.16.1.1/30 interface=ether3 network=172.16.1.0
        add address=192.168.20.1/30 interface=ether2 network=192.168.20.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether3
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=2.2.2.2 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.NY


**Роутер R01.SPB**

        /interface bridge
        add name=Lo
        add name=VPLS
        /interface vpls
        add disabled=no l2mtu=1500 mac-address=02:2D:B2:04:58:B5 name=vpls1 remote-peer=1.1.1.1 vpls-id=10:0
        add disabled=no l2mtu=1500 mac-address=02:07:F9:C5:13:11 name=vpls2 remote-peer=6.6.6.6 vpls-id=10:0
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default router-id=4.4.4.4
        /routing ospf instance
        set [ find default=yes ] router-id=4.4.4.4
        /interface bridge port
        add bridge=VPLS interface=ether3
        add bridge=VPLS interface=vpls1
        add bridge=VPLS interface=vpls2
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=4.4.4.4 interface=Lo network=4.4.4.4
        add address=172.16.5.2/30 interface=ether2 network=172.16.5.0
        add address=192.168.10.1/30 interface=ether3 network=192.168.10.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether2
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=3.3.3.3 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.SPB
        
  **Роутер R01.SVL**
  
        /interface bridge
        add name=Lo
        add name=VPLS
        /interface vpls
        add disabled=no l2mtu=1500 mac-address=02:A3:F4:DF:B8:5F name=vpls1 remote-peer=1.1.1.1 vpls-id=10:0
        add disabled=no l2mtu=1500 mac-address=02:44:7B:1A:38:5A name=vpls2 remote-peer=4.4.4.4 vpls-id=10:0
        /interface wireless security-profiles
        set [ find default=yes ] supplicant-identity=MikroTik
        /routing bgp instance
        set default router-id=6.6.6.6
        /routing ospf instance
        set [ find default=yes ] router-id=6.6.6.6
        /interface bridge port
        add bridge=VPLS interface=ether3
        add bridge=VPLS interface=vpls1
        add bridge=VPLS interface=vpls2
        /ip address
        add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
        add address=6.6.6.6 interface=Lo network=6.6.6.6
        add address=172.16.6.2/30 interface=ether2 network=172.16.6.0
        add address=192.168.30.1/30 interface=ether3 network=192.168.30.0
        /ip dhcp-client
        add disabled=no interface=ether1
        /mpls ldp
        set enabled=yes
        /mpls ldp interface
        add interface=ether2
        /routing bgp peer
        add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=5.5.5.5 remote-as=65530 \
            update-source=Lo
        /routing ospf network
        add area=backbone
        /system identity
        set name=R01.SVL
  
3. Также комьютерам были назначены ip-адреса из одной сети 192.168.0.0:

**PC1**

        [admin@PC1] > /ip address
        [admin@PC1] /ip address> add address=192.168.0.1/24 interface=ether2 network=192.168.0.0

**PC2**

        [admin@PC2] > /ip address
        [admin@PC2] /ip address> add address=192.168.0.2/24 interface=ether2 network=192.168.0.0

**PC3**

        [admin@PC3] > /ip address
        [admin@PC3] /ip address> add address=192.168.0.3/24 interface=ether2 network=192.168.0.0

4. Проверка связности сети:

![Снимок экрана от 2023-01-12 19-45-31](https://user-images.githubusercontent.com/63160594/212131603-f5153565-1d63-403d-9851-646dacb8699e.png)

![Снимок экрана от 2023-01-12 19-46-07](https://user-images.githubusercontent.com/63160594/212131627-ddd7b4de-74fd-4d2e-8d02-90b43252065d.png)

![Снимок экрана от 2023-01-12 19-46-35](https://user-images.githubusercontent.com/63160594/212131672-accf0d9d-c45f-4a38-aa6e-1c39bd357c73.png)


### Вывод
В ходе лабораторной работы №4 изучены протоколы BGP, MPLS, правила организации L3VPN и VPLS, настроен VRF и VPLS.
