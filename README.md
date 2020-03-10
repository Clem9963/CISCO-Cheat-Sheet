# CISCO Cheat Sheet

## Foreword

What are reliable networks?

- Fault tolerance
- Scalability
- Quality of Service (QoS)
- Security

## Initial device configuration

### Initial hardware cabling

- Turn on the device.
- Connect it with a RJ45 cable between a PC and the console port.
- Open the terminal and issue the command `minicom`.

### Minicom

#### In case of problems

In order to be able to see all the data that are transmitted through the console port, you should start `minicom`.  
If there is a problem, you can issue : `minicom -s` or press the following hotkeys : `Ctrl-A Z` if minicom is already running.

#### Configure the console port.

```
/dev/modem -> /dev/ttyS1 or /dev/ttyS0 (test the 2 beacause no there is no other way to find out)
Press enter
Change the Flow Rate to 9600
Press echap
Save config as dfl
"Enter initial configuration dialog ?" -> No
Press enter
```

## Miscellaneous

### Debugger's arsenal

```
tracert             (Trace route on Windows.)
traceroute          (Determine the route to a destination by sending ICMP echo packets.)
ping                (Send ICMP echo packets to a specified host.)
debug ip icmp       (Enable ICMP log messages on the console port.)
undebug all         (Stop all the debugs.)
no debup ip icmp    (Disable the ICMP log messages.)
terminal monitor    (Let the logs appear on a remote management session.)
```

### Display the IP addresses of the different interfaces

```
show ip interface brief
```

### Display all the VLANs

```
show vlan
```

### Figure out which side on the point-to-point link is the DCE side or the DTE end

```
show controllers serial
```

### Setting the enable password

```
enable secret password
```

### Encrypting passwords

```
service password-encryption
```

### Block access after a certain number of connections

```
login block-for 120 attempts 3 within 60
```

### Detailed password setup

```
enable
enable secret class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
```

### Rename a host

```
enable
configure terminal
hostame MyCoolRouter
exit
```

### Set up a banner

```
banner motd $21 is a prime number.$
```

### Displays information for each translation table entry

```
show ip nat translations
```

## Routing

### Configure the IPv4 addresses of the interfaces

```
enable
configure terminal
interface FastEthernet0/0
ip address 192.168.X.X 255.X.X.X
description link to lan 1
no shutdown
exit
```

### Configure the loopback address

```
interface loopback 0
ip address 10.0.0.1 255.255.255.0
exit
```

### Configure the IPv6 addresses of the interfaces

```
enable
conf t
int fastethernet0/1
ipv6 address 2001:db8:acab:1::1/64
no shutdown
exit
```

### Configure the IPv6 link local address

```
enable
conf t
int fastethernet0/1
description link to lan 2
ipv6 addresse fe80::1 link-local
clock rate 200000
no shutdown
exit
```

### Define the default Gateway

```
ip default-gateway 192.168.10.1
```

### Enable RIPv2

> **Prerequisite** : The IP addresses of the interfaces must have been configured. _(see above)_

```
enable
config t
router rip
version 2
no auto-summary
network X.X.X.X     (address of the 1st network in which the router has an interface)
network X.X.X.X     (address of the 2nd network in which the router has an interface)
exit
```

> **Observation** : `no auto-summary` allows to obtain subnet masks according to the configuration of the interfaces.

### Configure static routes

```
enable
config t
ip route X.X.X.X Y.Y.Y.Y Z.Z.Z.Z
exit
show ip route
```

> **Caption** :  
> X.X.X.X the network address  
> Y.Y.Y.Y the subnet mask  
> Z.Z.Z.Z the next hop address OR the outbound interface (FastEthernet0/0 for example)

### Configure the clock for wired networks with serial cables

```
enable
show ip interface brief     (to find out which serial to change the clock on)
config t
interface Serial 0/0/X      (with X the correct serial number)
clock rate X                (with X the flow rate in bauds, for example 2000000)
no shutdown
exit
```

### Configure a trunk link

#### Clear the IP address of the master interface

```
interface FastEthernet 0/1
no ip address
no shutdown
exit
```

#### Set up a vlan number on a sub interface

```
interface FastEthernet 0/1.1
encapsulation dot1q 11
ip adddress 192.168.1.1 255.255.255.0
no shutdown
exit
```

> **Notice** : The native vlan must not be specified on the router although it must be set up on the switch.

### Set up a DHCP server

```
enable
ip dhcp excluded-address 192.168.10.1
ip dhcp pool LAN-POOL-1
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1            (default gateway)
(optional) dns-server 192.168.11.5
(optional) domain-name example.com
end
```

### PAT configuration with only one external address

```
conf t
ip nat inside source list 1 interface FastEthernet4 overload   (FastEthernet4 is the outbound interface)
access-list 1 permit 192.168.20.0 0.0.0.255
exit
conf t
interface Vlan1                                                (Vlan1 is the inbound interface)
ip nat inside
exit
interface FastEthernet4                                        (FastEthernet4 is the outbound interface)
ip nat outside
```

## Switching

### Erase the configuration

```
enable
delete flash:vlan.dat
erase startup-config
reload
```

> **Warning** : Never backup when it asks to!

### Give a name to a VLAN

```
enable
conf t
vlan 4
name MyVLAN
```

> **Warning** :
>
> - The `conf t` command should be replaced by the `vlan database` command on old switches.
> - NEVER modify vlan 1 since it is the default vlan.

### Configuring the ports of a VLAN

```
enable
show vlan                     (to find out in which VLAN each interface is located)
config-t
interface FastEthernet0/X     (X = interface number)
switchport mode access
switchport access vlan X      (X = the number of the VLAN to which we want the interface to belong)
exit
```

### Set up a trunk link with a router

> **Prerequisite** : The encapsulated VLANs (11 and 2 in the following example) must have been configured.

```
conf t
int fe0/4
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 11,2,99
end
```

### Remote management through SSH

```
conf t
ip domain-name span.com
crypto key generate rsa general-keys modulus 1024
username admin secret ccna
line vty 0 15
transport input ssh
login local
exit
ip ssh version 2
exit
```

## End devices (PCs)

### Configure the ip address of a PC (under CentOS 7) :

```
Click on the icon of the 2 cables connected at the bottom right (Or System Settings then Network)
Click on the gear
IPv4
In "Addresses" Change Automatic (DHCP) to "Manual".
Fill in the fields that appear
Apply
Then disconnect, reconnect with the widget that slides from left to right
```

---

## The part below still needs to be translated!

supprimer config d'un routeur / switch
erase startup-config
delete flash:vlan.dat (seulement pour les switch)
reload

voir les interfaces :
show ip interface brief
show ipv6 interface brief

voir la table de routage :
show startup-config
show running-config
show ip route
show ipv6 route
show running-config interface fastethernet0/1
show ip route | begin Gateway
show ip route static
show port-security interface fe0/1
show vlan brief
show interfaces vlan 10
show access-lists
clear access-lists counter (num de l'acl)
show ip nat translations
show cdp
show cdp neighbors
show cdp interface
show ip nat statistics
clear ip nat
debug ip nat

voir la version de IOS :
show version

faire une route statique :
ip route 192.168.10.0 255.255.255.0 s0/0/0
ipv6 route ::/0 s0/0/0
ipv6 route 2001:0DB8:ACAB:1::/64 2001:0DB8:ACAD:3::1
exit

configurer une route statique de tronçon suivant :
ip route 172.16.1.0 255.255.255.0 172.16.2.2
ip route 192.168.1.0. 255.255.255.0 172.16.2.2

configurer un route statique connectée directement :
ip route 172.16.1.0 255.255.255.0 s0/0/0
ip route 192.168.1.0 255.255.255.0 s0/0/0

configurer des routes statiques entièrement spécifiées :
ip route 172.16.1.0 255.255.255.0 G0/1 172.16.2.2
ip route 192.168.1.0 255.255.255.0 G0/1 172.16.2.2

route statique par défaut :
ip route 0.0.0.0 0.0.0.0 172.16.2.2

configurer des routes ipv6 de troncon suivant :
ipv6 route 2001:DB8:ACAD:2::/64 2001:DB8:ACAD:4::2

configurer un route statique ipv6 connectée directement :
ipv6 route 2001:DB8:ACAD:2::/64 s0/0/0

configurer des routes statiques ipv6 entièrement spécifiées :
ipv6 rpite 2001:DB8:ACAD:2::/64 fe80::2

route statique ipv6 par défaut :
ipv6 route ::/0 2001:DB8:ACAD:4::2

configurer une route statique flottante :
ip route 0.0.0.0 0.0.0.0 172.16.2.2

configurer une route statique ipv6 flottante :
ipv6 route ::/0 2001:DB8:ACAD:4::2

faire du routage rip :
enable
conf t
router rip
version 2
no auto-summary
network x.x.x.x (1er réseau)
network y.y.y.y (2eme reseau)
..
exit

configurer des interfaces passives :
router rip
passive-interface g0/0
end

configurer un switch :
conf t
vlan 10
exit
interface fe0/1
switchport mode access
switchport access vlan 10
exit

configuration commutateur en mode full duplex :
conf t
int fe0/1
duplex full
speed 100
end

mettre le MDIX :
conf t
int fe0/1
duplex auto
speed auto
mdix auto
end

commutateur securiser le port :
conf t
int fe0/1
switchport mode access
switchport port-security

1. création d'un vlan ;
   conf t
   vlan 10
   name student
   end

2. attribution d'un port à un vlan
   conf t
   int fe0/1
   swicthport mode access
   switchport access vlan 10
   end

3. modification de l'appartenance des ports aux vlan
   int fe0/1
   no switchport access vlan
   end

4. suppression de vlan
   conf t
   no vlan 10
   end

configurer liaison trunk :
conf t
int fe0/4
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,30,99
end

réinitialisation de trunk par defaut :
int fe0/1
no switchport trunk allowed vlan
no switchport trunk native vlan
switchport mode access
end

ACL et mot cle de masque générique :
conf t
access-list 1 permit 0.0.0.0 255.255.255.255
!OR
access-listt 1 permit any

access-list 1 permit 192.168.10.10 0.0.0.0
!OR
access-list 1 permit host 192.168.10.10

autoriser une addresse ipv4 dans une ACL :
access-list 10 permit 192.168.10.0 0.0.0.255

supprimer les acl
no access-list 10

configurer une acl sur une interface :
conf t
access-list 1 permit 192.168.10.0 0.0.0.255
int serial0/0/0
ip access-group 1 out

autoriser tout le monde sauf une addresse :
conf t
no access-list 1
access-list 1 deny host 192.168.10.10
access-list 1 permit any
int g0/0
ip access-group 1 in

nommer une ACL
ip access-list standart NO_ACCESS
deny host 19.168.11.10
permit any
exit
int g0/0
ip access-group NO_ACCESS out

sécuriser le port VTY
conf t
line vty 0 4
login local
transport input ssh
access-class 21 in
exit
access-list 21 permit 19.168.10.0 0.0.0.255
access-list 21 deny any

Configurer du DHCP sur un router :

exclure une addresse ip :
enable
ip dhcp excluded-address 192.168.10.254

exclure un pool d'addresse
ip dhcp excluded-address 192.168.10.1 192.168.10.9

configurer le dhcp :
enable
ip dhcp excluded-address 192.168.10.1
ip dhcp pool LAN-POOL-1
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1 //pesserelle par default
(facul) dns-server 192.168.11.5
(facul) domain-name example.com
end

afficher le dhcp configuré :
show running-config | section dhcp
show ip dhcp binding
show ip dhcp server statistics
show running-config |section interface g0/0

faire un relai dhcpv4 :
conf t
int g0/0
ip helper-addresse 192.168.11.6

configurer un router en tant que client dhcp :
conf t
int g0/1
ip address dhcp
no shutdown
end

afficher les conflits :
show ip dhcp conflict

Configurer le dhcpv6 sans etat :
conf t
ipv6 unicast-routing
ipv6 dhcp IPV6-STATELESS
dns-server 2001:db8:cafe:aaaa::5
domain-name example.com
exit
int g0/1
ipv6 address 2001:db8:cafe:1::1/64
ipv6 dhcp server IPV6-STATELESS
ipv6 nb other-config-flag

configurer le dhcpv6 avec etat :
conf t
ipv6 unicast-routing
ipv6 dhcp IPV6-STATEFUL
address prefix 2001:db8:cafe:1::/64 lifetime infinite
dns-server 2001:db8:cafe:aaaa::5
domain-name example.com
exit
int g0/1
ipv6 address 2001:db8:cafe:1::1/64
ipv6 dhcp server IPV6-STATEFUL
ipv6 nb managed-config-flag

permettre au routeur de recevoir une addresse link local :
int g0/1
ipv6 enable
ipv6 address dhcp

Configurer un routeur en tant qu'agent de relais DHCPV6 :
int g0/1
ipv6 dhcp relay destination 2001:db8:cafe:1::6
end
show ipv6 dhcp interface g0/0

Configurer une traduction NAT
conf t
ip nat inside source static 192.168.11.99 209.165.201.5
interface serial0/0/0
ip address 192.168.1.2 255.255.255.252
ip nat inside
exit
int serial0/1/0
ip address 209.165.100.1 255.255.255.252
ip nat outside

configurer NAT dynamique :
conf t
ip nat pool NAT-POOL1 209.165.200.226
netmask 255.255.255.224
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 pool NAT-POOL1
int serial 0/0/0
ip nat inside

Configuration de PAT :
conf t
ip nat pool NAT-POOL2 209.165.200.226
netmask 255.255.255.224
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 pool NAT-POOL2 overload
int serial0/0/0
ip nat inside
int serial0/1/0
ip nat outside

Configuration de PAT avec une seule addresse externe :
conf t
ip nat inside source list 1 interface FastEthernet4 overload (FastEthernet4 est l'interface du réseau externe)
access-list 1 permit 192.168.20.0 0.0.0.255
exit
conf t
interface Vlan1 (Vlan1 est l'interface du réseau interne)
ip nat inside
exit
interface FastEthernet4
ip nat outside

autres exemple :
conf t
access-list permit 192.16.0.0 0.0.255.255 (configurer acl1 pour permettre trad NAT)
int serial0/0/0
ip nat inside
int serial0/1/0
ip nat outside

Configurer transfert de port
conf t
ip nat inside source static tcp 192.168.10.254 80 209.165.200.225
int serial0/0/0
ip nat inside
int serial0/1/0
ip nat outside

Configuration du protocole CDP :
enable
conf
int giga0/1
cdp enable

desactivation globale :
no cdp run
exit

Configuration du protocole LLDP :
conf t
lldp run
int giga0/1
lldp transmit
lldp receive
show lldp

clock set 20:36:00 dec 11 2015

Configuration du protocole NTP (pour l'heure) :
show click detail
ntp server 209.165.200.225
end
show clock detail
show ntp associations

show logging | include changed state to up

maintenance des fichiers du routeur et commutateurs :
show file system
dir
cd nvram:
pwd
copy running-config usbflash0:
