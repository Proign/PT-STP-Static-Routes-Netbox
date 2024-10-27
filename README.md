# Преднастройка маршрутизаторов и коммутаторов
По примеру конфигурации роутера 2600A составим конфигурации для роутеров 2600B и 2600C
## Конфигурация роутера 2600A
```
Router>enable
Router#config t
Router(config)#hostname 2600A
2600A(config)#enable secret todd
2600A(config)#line console 0
2600A(config-line)#password todd
2600A(config-line)#login
2600A(config-line)#line vty 0 4
2600A(config-line)#password todd
2600A(config-line)#login
2600A(config-line)#interface fastethernet 0/0
2600A(config-if)#ip address 172.16.40.1 255.255.255.0
2600A(config-if)#description connection to LAN 40
2600A(config-if)#no shutdown
2600A(config-if)#interface serial 0/0
2600A(config-if)#ip address 172.16.20.2 255.255.255.0
2600A(config-if)#description connection to 2600C
2600A(config-if)#no shutdown
2600A(config-if)#exit
2600A(config)#banner motd #
This is the 2600A router
#
2600A(config)#exit
2600A#copy run start
```
## Конфигурация роутера 2600B
```
Router>enable
Router#config t
Router(config)#hostname 2600A
2600B(config)#enable secret todd
2600B(config)#line console 0
2600B(config-line)#password todd
2600B(config-line)#login
2600B(config-line)#line vty 0 4
2600B(config-line)#password todd
2600B(config-line)#login
2600B(config-line)#interface fastethernet 0/0
2600B(config-if)#ip address 172.16.50.1 255.255.255.0
2600B(config-if)#description connection to LAN 50
2600B(config-if)#no shutdown
2600B(config-if)#interface serial 0/0
2600B(config-if)#ip address 172.16.21.1 255.255.255.0
2600B(config-if)#description connection to 2600C
2600B(config-if)#no shutdown
2600B(config-if)#exit
2600B(config)#banner motd #
This is the 2600B router
#
2600B(config)#exit
2600B#copy run start
```
## Конфигурация роутера 2600C
```
Router>enable
Router#config t
Router(config)#hostname 2600A
2600C(config)#enable secret todd
2600C(config)#line console 0
2600C(config-line)#password todd
2600C(config-line)#login
2600C(config-line)#line vty 0 4
2600C(config-line)#password todd
2600C(config-line)#login
2600C(config-line)#interface fastethernet 0/0
2600C(config-if)#ip address 172.16.10.1 255.255.255.0
2600C(config-if)#description connection to 3550SwitchB
2600C(config-if)#no shutdown
2600C(config-if)#interface serial 0/0
2600C(config-if)#ip address 172.16.20.1 255.255.255.0
2600C(config-if)#description connection to 2600A
2600C(config-if)#encapsulation ppp
2600C(config-if)#clock rate 2000000
2600C(config-if)#no shutdown
2600C(config-if)#interface serial 0/1
2600C(config-if)#ip address 172.16.21.2 255.255.255.0
2600C(config-if)#description connection to 2600B
2600C(config-if)#encapsulation ppp
2600C(config-if)#clock rate 2000000
2600C(config-if)#no shutdown
2600C(config-if)#exit
2600C(config)#banner motd #
This is the 2600C router
#
2600C(config)#exit
2600C#copy run start
```
# Установим имена, пароли и баннеры для коммутаторов
## 2950SwitchA
```
Switch>enable
Switch#config t
Switch(config)#hostname 2950SwitchA
2950SwitchA(config)#enable secret todd
2950SwitchA(config)#line console 0
2950SwitchA(config-line)#password todd
2950SwitchA(config-line)#login
2950SwitchA(config-line)#line vty 0 4
2950SwitchA(config-line)#password todd
2950SwitchA(config-line)#login
2950SwitchA(config-line)#exit
2950SwitchA(config)#banner motd #
This is 2950SwitchA
#
2950SwitchA(config)#exit
2950SwitchA#copy run start
2950SwitchA#
```

Настройка коммутаторов 2950SwitchB, 3550SwitchA, 3550SwitchB, 3550SwitchC производится аналогично
# STP
Для настройки STP и выполнения требований нужно настроить приоритеты на коммутаторах и выставить PortFast на порты, подключенные к оконечным устройствам.
## Настройка 3550SwitchB
```
enable
configure terminal
spanning-tree vlan 1 priority 4096
```
## Настройка 3550SwitchC
```
enable
configure terminal
spanning-tree vlan 1 priority 8192
```
# Настройка PortFast
## Настройки на 3550SwitchA
```
enable
configure terminal
interface FastEthernet0/1
spanning-tree portfast
exit

interface FastEthernet0/2
spanning-tree portfast
exit
```

Настройка коммутаторов 3550SwitchB, 3550SwitchC производится аналогично

## Работоспособность STP при отключении FastEthernet0/3 на 3550SwitchA
### До
![Port UP](https://github.com/Proign/PT-STP-Static-Routes-Netbox/blob/main/screenshots/STP-1.PNG)

### После
![Port DOWN](https://github.com/Proign/PT-STP-Static-Routes-Netbox/blob/main/screenshots/STP-2-shutdown.PNG)

# Статическая маршрутизация
## 2600RouterA
```
ip route 172.16.50.0 255.255.255.0 172.16.20.1 
ip route 172.16.10.0 255.255.255.0 172.16.20.1 
```
## 2600RouterB
```
ip route 172.16.40.0 255.255.255.0 172.16.21.2 
ip route 172.16.10.0 255.255.255.0 172.16.21.2 
```
## 2600RouterC
```
ip route 172.16.40.0 255.255.255.0 172.16.20.2 
ip route 172.16.50.0 255.255.255.0 172.16.21.1 
```

На хостах HostA, HostB, HostC, HostD и HostG необходимо установить шлюз 172.16.10.1

## Ping от HostE до HostB
![Ping HostB from HostE](https://github.com/Proign/PT-STP-Static-Routes-Netbox/blob/main/screenshots/HostE2HostB.PNG)

## Ping от HostG до HostF
![Ping HostB from HostE](https://github.com/Proign/PT-STP-Static-Routes-Netbox/blob/main/screenshots/HostE2HostB.PNG)

# Составление IP плана
![HostA](https://github.com/Proign/PT-STP-Static-Routes-Netbox/blob/main/screenshots/Netbox-1.PNG)

![HostA interface](https://github.com/Proign/PT-STP-Static-Routes-Netbox/blob/main/screenshots/Netbox-2.PNG)

![HostA IP](https://github.com/Proign/PT-STP-Static-Routes-Netbox/blob/main/screenshots/Netbox-3.PNG)