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
- Connect a console cable to the console port of the device and connect the other end to the PC.
- Open the terminal and issue the command `minicom`.

### Minicom

#### In case of troubles

In order to be able to see all the data that is transmitted through the console port, you should start the modem control and terminal emulation program : `minicom`.  
If there is a problem, you can either issue : `minicom -s` or press the following hotkeys : `Ctrl-A Z` In the event that minicom is already running.

#### Configure the console port.

```
/dev/modem -> /dev/ttyS1 or /dev/ttyS0 (test the 2 because there is no other way to find out)
Press enter
Change the Flow Rate to 9600
Press echap
Save config as dfl
"Enter initial configuration dialog ?" -> No
Press enter
```

## Miscellaneous

### Debugging

#### Generic debugging and logging commands

The command `tracert` can be used on Windows to display the route and measure transit delays of packets across an Internet Protocol network.

Down below is written a set of commands which can be entered on any CISCO device for debugging purposes.

```
show running-config
show running-config interface fastethernet0/1
show running-config | section interface g0/0
show log | include changed state to up
show startup-config
terminal monitor                    (Let the logs appear on a remote management session)
show version                        (Show the IOS version)
show access-lists                   (Show ACLs)
show port-security interface fe0/1  (Show the port security policy on a specific interface)
show controllers serial             (Figure out which side on the point-to-point link is the DCE side or the DTE end)
undebug all                         (Stop all the debugs)
```

#### Debugging commands on a router

##### IP debug

```
traceroute                          (Determine the route to a destination by sending ICMP echo packets)
ping                                (Send ICMP echo packets to a specified host)
debug ip icmp                       (Enable ICMP log messages)
no debup ip icmp                    (Disable the ICMP log messages)
debug ip rip                        (Enable RIP log messages)
no debup ip rip                     (Disable the RIP log messages)
```

##### Routing debug

```
show ip route                       (Show the IPv4 routing table)
show ipv6 route                     (Show the IPv6 routing table)
show ip route | begin Gateway       (Show the gateway of last resort)
show ip route static                (Show the static routes)
```

##### CDP debug

```
show cdp                            (Display global CDP information)
show cdp neighbors                  (Display detailed information about neighboring devices discovered using CDP)
show cdp interface fe0/1            (Display CDP information about the interface fe0/1)
```

##### NAT & PAT debug

```
debug ip nat                        (Enable NAT log messages)
show ip nat translations            (Show information about NAT translations that are active on the router)
show ip nat statistics              (Show IP NAT translation statistics)
clear ip nat translation            (To clear dynamic NAT translations from the translation table)
```

##### Interfaces debug

```
show ip interface brief             (IPv4 interface details)
show ipv6 interface brief           (IPv6 interface details)
```

##### DHCP debug

```
show running-config | section dhcp  (Show the current DHCP configuration)
show ip dhcp binding                (To display IP-to-MAC address bindings)
show ip dhcp server statistics      (Display extended DHCP local server statistics.)
show ip dhcp conflict               (When hosts have statically assigned IP addresses that are within the DHCP range)
```

#### Debugging commands on a switch

```
show interface vlan 10              (Display brief descriptive information about a specific VLAN)
show vlan                           (Detailed display of VLANs)
show vlan brief                     (Summary display of VLANs)
```

### Passwords and security features

#### Basic commands

```
enable secret password                      (Set up the enable password)
service password-encryption                 (Encrypting passwords)
login block-for 120 attempts 3 within 60    (Block access after a certain number of connections)
```

#### Detailed password setup

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
banner motd $21 is a prime number$
```

### Configure the clock for wired networks with serial cables

```
enable
show ip interface brief     (to find out which serial to change the clock on)
config t
interface Serial 0/0/X      (with X the correct serial number)
clock rate X                (with X the flow rate in bauds, for example 2 000 000)
no shutdown
exit
```

### Maintenance of router and switch files

```
show file system
dir
cd nvram:
pwd
copy running-config usbflash0:
```

### Erase the configuration of a device

```
enable
erase startup-config
reload
```

> **Warning** :
>
> - On a switch, issue the following command before reloading : `delete flash:vlan.dat`
> - In any case, never backup when it asks to!

## Router configuration

### IPv4

#### Configure the IPv4 address of an interface

```
enable
configure terminal
interface FastEthernet0/0
ip address 192.168.1.0 255.255.255.0
description link to lan 1
no shutdown
exit
```

#### Configure the loopback address

```
interface loopback 0
ip address 10.0.0.1 255.255.255.0
exit
```

#### Define the default Gateway when IP routing is disabled

```
ip default-gateway 192.168.10.1
```

#### IPv4 static routing

```
enable
config t
ip route 172.16.1.0 255.255.255.0 172.16.2.2       (next hop)
ip route 172.16.1.0 255.255.255.0 G0/1             (on link)
ip route 172.16.1.0 255.255.255.0 G0/1 172.16.2.2  (fully qualified)
ip route 0.0.0.0 0.0.0.0 172.16.2.2                (gateway of last resort)
exit
show ip route
```

#### IPv4 dynamic routing

##### Enable RIPv2

> **Prerequisite** : The IP addresses of the interfaces must have been configured. [See above](#-Configure-the-IPv4-address-of-an-interface)

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

> **Little tip** : `no auto-summary` allows to obtain subnet masks according to the configuration of the interfaces.

##### Configuring passive interfaces

```
router rip
passive-interface g0/0
end
```

#### DHCPv4

##### Set up a DHCP server

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

> **Notice** : It is also possible to exclude an address pool by issuing for example  
> `ip dhcp excluded-address 192.168.10.1 192.168.10.9`.

##### Make a dhcp relay

```
conf t
int g0/0
ip helper-addresse 192.168.11.6
```

> **Notice** :
>
> - g0/0 is the interface from which DHCP requests are received.
> - 192.168.11.6 is the address of the DHCP server to which requests are relayed.

##### Configuring the router as a DHCP client

```
conf t
int g0/1
ip address dhcp
no shutdown
end
```

#### NAT

##### Setting up a static NAT translation

```
conf t
ip nat inside source static 192.168.11.99 209.165.201.5
interface serial0/0/0
ip address 192.168.1.2 255.255.255.252
ip nat inside
exit
int serial0/1/0
ip address 209.165.100.1 255.255.255.252
ip nat outside
```

##### Set up Dynamic NAT

```
conf t
ip nat pool NAT-POOL1 209.165.200.226 209.165.200.240 netmask 255.255.255.224
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 pool NAT-POOL1
int serial 0/0/0
ip nat inside
int serial 0/1/0
ip nat outside
```

#### PAT

##### PAT with a pool of external addresses

```
conf t
ip nat pool NAT-POOL2 209.165.200.226 209.165.200.240 netmask 255.255.255.224
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 pool NAT-POOL2 overload
int serial0/0/0
ip nat inside
int serial0/1/0
ip nat outside
```

##### PAT configuration with only one external address

```
conf t
ip nat inside source list 1 interface FastEthernet4 overload   (FastEthernet4 is the outbound interface)
access-list 1 permit 192.168.20.0 0.0.0.255                    (Define which addresses are eligible to be translated)
exit
conf t
interface Vlan1                                                (Vlan1 is the inbound interface)
ip nat inside
exit
interface FastEthernet4                                        (FastEthernet4 is the outbound interface)
ip nat outside
```

##### Setting up port forwarding

The goal is to establish static translation between an inside local address and local port and an inside global address and global port.

```
conf t
ip nat inside source static tcp 192.168.10.254 80 209.165.200.225 8080
int serial0/0/0
ip nat inside
int serial0/1/0
ip nat outside
```

### IPv6

#### Configure the IPv6 addresses of the interfaces

```
enable
conf t
int fastethernet0/1
ipv6 address 2001:db8:acab:1::1/64
no shutdown
exit
```

#### Configure the IPv6 link local address

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

#### IPv6 static routing

```
enable
conf t
ipv6 route 2001:0DB8:ACAB:1::/64 2001:0DB8:ACAD:3::1    (next hop)
ipv6 route 2001:DB8:ACAD:2::/64 s0/0/0                  (on link)
ipv6 route 2001:DB8:ACAD:2::/64 fe80::2                 (next hop with link-local address)
ipv6 route ::/0 s0/0/0                                  (gateway of last resort on link)
ipv6 route ::/0 2001:DB8:ACAD:4::2                      (gateway of last resort by address)
```

#### DHCPv6

##### Stateless DHCPv6

```
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
```

##### Stateful DHCPv6

```
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
```

##### Allow the router to receive a local link address via DHCPv6.

```
int g0/1
ipv6 enable
ipv6 address dhcp
```

##### Configuring a router as a DHCPv6 relay agent

```
int g0/0
ipv6 dhcp relay destination 2001:db8:cafe:1::6
end
show ipv6 dhcp interface g0/0       (To check the configuration)
```

> **Notice** :
>
> - g0/0 is the interface from which DHCP requests are received.
> - 2001:db8:cafe:1::6 is the address of the DHCPv6 server to which requests are relayed.

## Switch configuration

### VLANs

#### Creation

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

#### Attribution

```
enable
show vlan                     (to find out in which VLAN each interface is located)
config t
interface FastEthernet0/X     (X = interface number)
switchport mode access
switchport access vlan X      (X = the number of the VLAN to which we want the interface to belong)
exit
```

#### Remove an interface

```
int fe0/1
no switchport access vlan
end
```

#### Deletion

```
conf t
no vlan 10
end
```

#### Trunk link

Please refer to the section on [Router-on-a-stick configuration](#-Router-on-a-stick-configuration-:-create-a-trunk-link-between-a-switch-and-a-router)

#### Remote management through SSH

##### Allow remote management

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

##### Securing the VTY port

```
conf t
line vty 0 4
login local
transport input ssh
access-class 21 in
exit
access-list 21 permit 19.168.10.0 0.0.0.255
access-list 21 deny any
```

## Router-on-a-stick configuration : create a trunk link between a switch and a router

### Router side

1. Clear the IP address of the master interface

```
interface FastEthernet 0/1
no ip address
no shutdown
exit
```

2. Set up a vlan number on each sub interface

```
interface FastEthernet 0/1.1
encapsulation dot1q 11                  (We are encapsulating the VLAN number 11)
ip adddress 192.168.1.1 255.255.255.0
no shutdown
exit
```

You should repeat the commands above for each VLAN that needs to be encapsulated.

> **Notice** : The native vlan must not be specified on the router although it must be set up on the switch.

### Switch side

#### Setting up the trunk link with the router

> **Prerequisites** : The encapsulated VLANs (11 and 2 in the following example) must have been created and assigned to an interface. [See above](#-VLANs)

```
conf t
int fe0/4                               (The interface which is linked to the router)
switchport mode trunk
switchport trunk native vlan 99         (The native VLAN does not need to be created beforehand)
switchport trunk allowed vlan 11,2,99
end
```

#### Deleting a trunk link

```
int fe0/1
no switchport trunk allowed vlan
no switchport trunk native vlan
switchport mode access
end
```

## Other functionalities

### Advanced features of a router

#### CDP (Cisco Discovery Protocol)

##### Activation

```
enable
conf
int giga0/1
cdp enable
```

##### Deactivation

```
no cdp run
exit
```

#### LLDP (Link Layer Discovery Protocol)

```
conf t
lldp run
int giga0/1
lldp transmit
lldp receive
show lldp
```

#### NTP (Network Time Protocol)

##### Setting the date and time on a router

```
clock set 20:36:00 dec 11 2015
```

##### Configuring the NTP protocol

```
show click detail
ntp server 209.165.200.225
end
show clock detail
show ntp associations
```

### Full duplex switch configuration

```
conf t
int fe0/1
duplex full
speed 100
end
```

### MDIX

```
conf t
int fe0/1
duplex auto
speed auto
mdix auto
end
```

### Port security policy

```
conf t
int fe0/1
switchport mode access
switchport port-security
```

### ACLs

#### ACL handling

##### Generic wildcard

```
conf t
access-list 1 permit 0.0.0.0 255.255.255.255
```

OR

```
conf t
access-list 1 permit any
```

##### Allow only one host

```
access-list 1 permit 192.168.10.10 0.0.0.0
```

OR

```
access-list 1 permit host 192.168.10.10
```

##### Deny only one host

```
access-list 1 deny host 192.168.10.10
access-list 1 permit any
```

##### Allow a range of IPs

```
access-list 10 permit 192.168.10.0 0.0.0.255
```

##### Deleting an ACL

```
no access-list 10
```

##### Clear counters for an access list

```
clear access-lists 1
```

#### Configure an ACL on a specific interface

```
conf t
access-list 1 permit 192.168.10.0 0.0.0.255
int serial0/0/0
ip access-group 1 out|in
```

> **Caption** :
>
> - When you apply an ACL "in", the switch examines all traffic it RECEIVES on the interface against the ACL.
> - When you apply an ACL "out" on an interface the switch examines any traffic attempting to leave that interface against the ACL.

#### Define a standard IP ACL

Defines a standard IP access list and enters standard named access list configuration mode.

```
ip access-list standart NO_ACCESS
deny host 19.168.11.10
permit any
exit
int g0/0
ip access-group NO_ACCESS out
```
