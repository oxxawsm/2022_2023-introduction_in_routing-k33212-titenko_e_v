University: ITMO University

Faculty: FICT

Course: Introduction in routing

Year: 2022/2023

Group: K33212

Author: Titenko Elena Vitalevna

Lab: Lab3

Date of create: 28.11.2022

Date of finished: 2.12.2022


# Отчёт по лабораторной работе №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

***Цель:*** Изучить протоколы OSPF и MPLS, механизмы организации EoMPLS.

### Ход работы

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
           - endpoints: ["R01.LND:eth2","R01.NY:eth1"]
           - endpoints: ["R01.HKI:eth3","R01.LND:eth1"]
           - endpoints: ["R01.HKI:eth2","R01.LBN:eth2"]
           - endpoints: ["R01.MSK:eth1","R01.SPB:eth1"]
           - endpoints: ["R01.MSK:eth2","R01.LBN:eth1"]
           - endpoints: ["R01.SPB:eth2","PC1:eth1"]
           - endpoints: ["R01.SPB:eth3","R01.HKI:eth1"]
           - endpoints: ["R01.LBN:eth3","R01.NY:eth2"]
           - endpoints: ["R01.NY:eth3","SGI.Prism:eth1"]


 
2. Конфигурации устройств:

**Роутер R01.NY**

   

**Роутер R01.LND**

    
**Роутер R01.HKI**


**Роутер R01.SPB**


**Роутер R01.MSK**


**Роутер R01.LBN**


**Роутер R01.MSK**
   
    
**PC1**

  
 
**SGI.Prism**

  

   
    
3. Примеры пингов, проверка работоспособности сети



### Вывод
В ходе лабораторной работы №3 ...
