# Cisco Utils

**Warning, use at your own risk. I created these scripts with an educational mindset while studying for my CCNA**

[Work in Progress] Bootstrapping, hardening, and cheatsheet scripts for Cisco routers.
Most of the content so far is on this README.md document. Simply copy and paste the command references for the features you need to fire up the FTP server to pull then directly onto the network appliance.

## Cheatsheet Navigation

* [Setup](#Setup)
  * [Intialize](#intialize)
  * [Basic Config](#basic-config)
  * [Configure SSH](#configure-ssh)
  * [Basic Hardening](#basic-hardening)
  * [Backup Config](#backup-config)
  * [Restore Config](#restore-config)
* [VLANs](#vlans)
  * [Creation](#vlan-creation)
  * [Port Assignment](#port-assignment)
  * [IP Assignment](#ip-assignment)
  * [Verification](#verification)
  * [Management VLAN](#management-vlan)
  * [Delete VLANs on File](#delete-vlans-on-file)
  * [Delete VLANs in Memory](#delete-vlans-in-memory)
* [DHCP](#dhcp)
  * [Management DHCP](#management-dhcp)
* [Trunks](#trunks)
  * [VLAN Trunk](#create-multi-switch-vlan-trunk)
  * [Trunk Verification](#trunk-verification)
* [Dynamic Trunking Protocol (DTP)](#dtp-dynamic-trunking-protocol)
  * [Configure DTP](#configure-dtp)
  * [Disable DTP](#disable-dtp)
  * [Verify DTP](#verify-dtp)

## FTP Server Usage

1. Clone the repo: 

    ```
    git clone https://github.com/grplyler/cisco-utils
    ```
    
2. Install python requirements (for ftp server):

    ```
    pip install -r requirements.txt
    ```
    
3. Run python ftp_server.py

    ```
    python3 ftp_server.py
    ```
    
4. Pull a script onto a network device (WARNING: Backup to avoid any losses)

    ```
    Switch#> copy ftp://192.168.1.10/sw_base.txt running-config
    ```
    
    *Replace 192.168.1.10 with the IP of the computer connected to the switch or router.*
    
## Cisco Cheatsheet & Snippets

### Setup
---

#### Intialize

```
erase startup-config
delete vlan.dat
reload
```

#### Basic Config

```
configure terminal
no ip domain-lookup
hostname S1
enable secret class
line console 0
logging synchronous
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd $ Authorized Access Only! And Godzilla will beat Kong any day $
exit
copy running-config startup-config
```

#### Configure SSH

```
show ip ssh
conf t
ip domain-name cisco.com
crypto key generate rsa

username admin secret ccna
line vty 0 15
transport input ssh
login local
exit
ip ssh version 2
exit
```

#### Basic Hardening (Work Needed)

```
conf t
! Logout timer
!
line con 0
 exec-timeout 5
line vty 0 4
 exec-timeout 5
 
exit

ip ssh time-out 60
ip ssh authentication-retries 3
end
```

#### Backup config
```
copy running-config startup-config
copy startup-config ftp://192.168.1.10/config.txt
```

#### Restore Config
```
copy ftp://192.168.1.10/config.txt running-config
```



### VLANs
---

#### VLAN Creation

```
conf t
vlan 10
name Faculty
exit
```

```
conf t
vlan 20
name Students
exit
```

#### Port Assignment

```
conf t
interface range Fa0/1-12
switchport mode access
switchport access vlan 10
end
```

```
conf t interface range Fa0/13-24
switchport mode access
switchport access vlan 20
end
```

```
conf t
interface Gi0/1
switchport mode access
switchport access vlan 99
```

#### IP Assignemnt

```
cont t
int vlan 99
ip address 10.0.0.1 255.255.255.0
end
```

#### Verification

```
show vlan brief
```

#### Voice and Data VLAN

*Assuming Data on VLAN 10, Voice on VLAN 20*

```
conf t
int Fa0/4
switchport mode access
switchport access vlan 10
switchport voice vlan 20
end
```

#### Management VLAN

```
conf t
vlan 99
name Management
exit
interface Fa0/24
switchport mode access
switchport access vlan 99
exit
int vlan 99
ip addr 10.0.0.1 255.255.255.0
end
```

#### Delete VLANS on file

```
delete vlan.dat
```

#### Delete VLANS in memory
*Warning: Make sure you move ports to another vlan or the will be unsable*

```
conf t
no vlan 10
no vlan 20
end
```

### DHCP
---

#### Management DHCP

*Workaround for CCNA labs at Liberty University since we can't change the LAB IP addresses*

```
conf t
ip domain name cisco.com
ip dhcp excluded-address 10.0.0.1
ip dhcp pool managementpool
network 10.0.0.1 255.255.255.0
default-router 10.0.0.1
end
```

### Trunks
---

#### Create multi-switch vlan trunk

*S1*
```
conf t
interface Gi0/1
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 99
end
```

*Note: Remember to set the native vlan (to 99 for instance) on each switch in the trunk so you don't get a native vlan mismatch warning*

#### Trunk Verification

```
show interface trunk
show interface g0/1 switchport
```

### DTP (Dynamic Trunking Protocol)
---

#### Configure DTP

```
conf t
int gi0/1
switchport mode dynamic auto
end
```

**or**

```
conf t
int gi0/1
switchport mode dynamic desirable
end
```


#### Disable DTP

*Usefull for connecting to devices that don't support Cisco propietary DTP or creating a static trunk*

```
conf t
int gi0/1
switchport mode trunk
switchport nonegotiate
end
```

#### Verify DTP

```
show dtp interface gi0/1
```

### Routing
---

#### Sub-Interface Configuration

*Creates a multiple sub-interfaces on a router switch to enable inter-vlan routing.*

```
conf t
interface G0/0/1.10
description Default Gateway for VLAN 10
encapsulation dot1Q 10
ip add 192.168.10.1 255.255.255.0
exit

interface G0/0/1.20
description Default Gateway for VLAN 20
encapsulation dot1Q 20
ip addr 192.168.20.1 255.255.255.0
exit

interface G0/0/1.99
description Default Gateway for VLAN 99
encapsulation dot1Q 99
ip addr 192.168.99.1 255.255.255.0
exit

interface G0/0/1
description Trunk link to S1
no shut
end
```
