University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2022/2023

Group: K33212

Author: Titenko Elena Vitalevna

Lab: Lab1

Date of create: 14.10.2022

Date of finished: 21.10.2022

# Отчёт по лабораторной работе №1 "Установка ContainerLab и развертывание тестовой сети связи"

***Цель:*** ознакомиться с инструментом ContainerLab и методами работы с ним, изучить работу VLAN, IP адресации и т.д.

### Ход работы

1. Текст файла, который был использован для развертывания тестовой сети с расширением .yaml:

       name: lab1

        mgmt:
        network: statics
        ipv4_subnet: 172.20.20.0/24

       topology: 
        nodes: 
          R01.TEST:
            kind: vr-ros
            image: vrnetlab/vr-routeros:6.47.9
            mgmt_ipv4: 172.20.20.2`

          SW01.01.TEST:
            kind: vr-ros
            image: vrnetlab/vr-routeros:6.47.9
            mgmt_ipv4: 172.20.20.3

          SW02.01.TEST:
            kind: vr-ros
            image: vrnetlab/vr-routeros:6.47.9
            mgmt_ipv4: 172.20.20.4

          SW02.02.TEST:
            kind: vr-ros
            image: vrnetlab/vr-routeros:6.47.9
            mgmt_ipv4: 172.20.20.5

          PC1:
            kind: linux
            image: ubuntu:latest
            mgmt_ipv4: 172.20.20.6

          PC2:
            kind: linux
            image: ubuntu:latest
            mgmt_ipv4: 172.20.20.7


        links: 
          - endpoints: ["R01.TEST:eth1", "SW01.01.TEST:eth1"]
          - endpoints: ["SW01.01.TEST:eth2", "SW02.01.TEST:eth1"]
          - endpoints: ["SW01.01.TEST:eth3", "SW02.02.TEST:eth1"]
          - endpoints: ["SW02.01.TEST:eth2", "PC1:eth1"]
          - endpoints: ["SW02.02.TEST:eth2", "PC2:eth1"]
    
    
2. Конфигурации устройств:
**Роутер R01.TEST**

![Снимок экрана от 2022-10-26 13-05-36](https://user-images.githubusercontent.com/63160594/198003750-f235cc6f-c7cd-4b8b-9289-d987ae5d5c82.png)


**Коммутатор первого уровня SW01.L3.01.TEST**

![Снимок экрана от 2022-10-26 13-05-01](https://user-images.githubusercontent.com/63160594/198003841-ed9ed2b6-1dde-473e-aea6-052c32de9ec1.png)


**Первый коммутатор второго уровня SW02.L3.01.TEST**

![Снимок экрана от 2022-10-26 13-06-24](https://user-images.githubusercontent.com/63160594/198004007-68f43527-c31e-4bea-b7fa-371a8ffadf32.png)


**Второй коммутатор второго уровня SW02.L3.02.TEST**

![Снимок экрана от 2022-10-26 13-06-49](https://user-images.githubusercontent.com/63160594/198004120-78669251-a72b-4435-a2b9-0304c549f8b6.png)


3. Результаты пингов

![Снимок экрана от 2022-10-26 12-58-28](https://user-images.githubusercontent.com/63160594/198004826-022ec4c4-53c9-4ce4-963d-baf1ed2ff3b9.png)

![Снимок экрана от 2022-10-26 13-36-48](https://user-images.githubusercontent.com/63160594/198005289-6209cf8d-8871-40ae-8ff6-130586a09e66.png)

### Вывод
В ходе лабораторной работы №1 я познакомилась с инструментом для создания сетевых лаб на основе контейнеров - Сontainerlab. С помощью файла конфигурации сети с расширением .yaml была развернута сеть из роутера, 3-х коммутаторов и 2х ПК. Также в сети настроены vlan'ы и dhcp-сервер для автоматической раздачи ip адресов. Работоспособность сети подтвердили успешные пинги устройств внутри сети. 
