# SETUP:

```c
Laptop password:

"would you like to enter the initial configuration dialog?"
- NO
  
secret: 

do not save secret: 
- select 0 
```

# RESET:
```c
enable

show flash:
// determine if vlan.dat has been created on switch

delete vlan.dat 

// confirm, then run:
erase startup-config

// and reload the switch:
reload

// say no to prompt to save running configuration prior
no

and confirm reload (takes awhile)

// and finally bypass initial configuration dialog 
no
```

initial configuration:
```c
no ip domain lookup

hostname s1
```

---
# Access Switch
## VLAN creation
creating and naming VLAN:
```c
S1(config)# vlan 10  
S1(config-vlan)# name Student  
S1(config-vlan)# vlan 20  
S1(config-vlan)# name Faculty  
S1(config-vlan)# end
```

Assign PC-A to the Student VLAN:
```c
S1(config)# interface g1/0/6  
S1(config-if)# switchport mode access  
S1(config-if)# switchport access vlan 10
```

## Access and Trunk ports 
setting up TRUNKs: Dynamic Trunking Protocol (DTP)
```c
# ACCESS
forces the port to stay as a non trunk port. ignores DTP frames

# TRUNK
forces the port to be a trunk
sends DTP frames to try and convince the other side to be a trunk

# DYNAMIC AUTO
willing to become a trunk but wont start the conversation
only becomes a trunk if the other side asks (desirable or trunk mode)

# DYNAMIC DESIRABLE
actively asks the other side to become a trunk
if the neighbour is auto, desirable or trunk, the link becomes a trunk

# OFF
disables DTP entirely 
```

```c
S1(config)# interface g1/0/1  
S1(config-if)# switchport mode trunk 
```

---
# Distribution Switch

## Switched Virtual Interface (SVI):
```c
// Create an SVI for VLAN 10.  
S3(config)# interface vlan 10  
// Configure the SVI with the IP address from the Address Table.  
S3(config-if)# ip address 192.168.10.1 255.255.255.0  
S3(config-if)# no shutdown

do the same for vlan 20

//enable ip routing: important or else it wont allow inter vlan routing
S3(config)# ip routing

// SVI is like a virtual router interface
```

must enable ip routing on both dsw 

---
# Router

configure IP Address:
```c
R1(config)# interface g0/0/0
R1(config-if)# ip address 192.168.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

## Router-On-A-Stick ([[Lab 4b (Legacy, ROAS, Layer 3 Switch)|ROAS]]):

```c
R1(config)# interface g0/0/1.10  
// Configure the subinterface to operate on VLAN 10.  
R1(config-subif)# encapsulation dot1q 10  
// basically tell the router what language (VLAN) it speaks
R1(config-subif)# ip address 192.168.10.1 255.255.255.0

// you have to do this on the router interface:
R1(config)# interface g0/0/1  
R1(config-if)# no shutdown

// in order to enable the interface
// make sure to create all the vlan on the asw
// make sure you run no shut on the sub interface as well as the parent ports
```

Loopback interface:
```c
R1(config)# interface G0/1/1
R1(config-if)# ip address 192.168.1.1 255.255.255.0 
R1(config-if)# no shutdown

R1(config-if)# interface Loopback 0 
R1(config-if)# ip address 209.165.200.225 255.255.255.224
R1(config-if)# no shutdown
```

## Static Route:
directly connected static route:
```c
R1(config)# ip route 172.16.2.0 255.255.255.0 s0/0/0
```

next-hop static route: (next-hop IP address)
```c
DSW2<config>#ip route 192.168.1.0 255.255.255.0 10.1.1.2
```

fully specified static route:
```c
R1<config>#ip route 192.168.0.0 255.255.255.0 g0/0/1 10.1.1.1 
```

default static route:
```c
R1<config>#ip route 0.0.0.0 0.0.0.0 10.1.1.2
```

## OSPF:

enable OSPF:
```c
R1(config)# router ospf 1

// Note: The OSPF process id is kept locally and has no meaning to other routers on the network.
```

use the network command to indirectly enable R1 interfaces to run OSPF:
```c
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0  
R1(config-router)# network 192.168.12.0 0.0.0.3 area 0  
R1(config-router)# network 192.168.13.0 0.0.0.3 area 0

// this is the area id not the process id
```

configure for the other devices as well.
for DSW (go on to the interface and allow ospf):
```c
DSW1(config)# interface interface-id  
DSW1(config-if)# ip ospf 1 area 0
```

optional:
changing router-id:
```c
R1(config)# router ospf 1  
R1(config-router)# router-id 1.1.1.1  
// Reload or use "clear ip ospf process" command, for this to take effect  
R1(config)# end
```
change router-id using loopback adddress:
```c
R1(config)# interface lo0  
R1(config-if)# ip address 11.11.11.11 255.255.255.255  
R1(config-if)# end
```

passive interfaces:
```c
R1(config)# router ospf 1  
R1(config-router)# passive-interface g0/1/0
```

change OSPF metrics:
```c
R1(config)# router ospf 1  
R1(config-router)# auto-cost reference-bandwidth 10000  
// % OSPF: Reference bandwidth is changed. Please ensure reference bandwidth is consistent across all routers.
```

change bandwidth of an interface:
```c
R1(config)# interface g0/0/0  
R1(config-if)# bandwidth 128
```

change interface cost:
```c
R1(config)# interface G0/0/1  
R1(config-if)# ip ospf cost 1565
```


## NAT, PAT:
configure default static route from your router to the ISP:
```c
R1(config)# ip route 0.0.0.0 0.0.0.0 172.17.9.114
```

STATIC NAT: 
select one public IP address to be used for static NAT for your e.g web server running at (private IP):
```c
R1(config)# ip nat inside source static private-ip public-ip  
R1(config)# interface g0/0/1  
R1(config-if)# ip nat inside  
R1(config-if)# interface g0/0/0  
R1(config-if)# ip nat outside

// e.g ip nat inside source static 192.168.1.20 203.149.210.225
```

DYNAMIC NAT:
pool of public addresses
configure the local private address block (to be translated by dynamic NAT):
```c
R1(config)# access-list src-id permit subnet-block wildcard-mask  


```

configure the public IP address pool:
```c
R1(config)# ip nat pool pool-name pool-1st-public-ip pool-last-  
public-ip netmask pool-network-mask  

```

bind the local private address block to the public ip address pool:
(allows IPs inside access list 1 to have an IP to access internet)
```c
R1(config)# ip nat inside source list src-id pool pool-name  

```

PAT:
- Note that PAT cannot be used for an enterprise network if the ISP were to allocate a private IP address for your edge router to connect to its ISP router
configure local private address block to be translated by PAT (same as prev):

```c
R1(config)# access-list src-id permit subnet-block wildcard-mask  
```

configure PAT:
```c
R1(config)# ip nat inside source list src-id int int-id overload  

//e.g R1(config)# ip nat inside source list 1 int g0/0/0 overload  
```

NAT/PAT Pool Overload:
access list again:
```c
R1(config)# access-list src-id permit subnet-block wildcard-mask  
```

the public address pool:
```c
R1(config)# ip nat pool pool-name pool-1st-public-ip pool-last-public-ip netmask pool-network-mask  
```

bind them together:
```c
R1(config)# ip nat inside source list src-id pool pool-name overload  
```

## DHCP:

configure on DHCP router:
```c
DHCP(config)# ip dhcp excluded-addresses 1st-ip-addr last-ip-addr

DHCP(config)# ip dhcp pool pool-name  
DHCP(dhcp-config)# network network-block subnet-mask  
DHCP(dhcp-config)# default-router ip-address  
DHCP(dhcp-config)# dns-server ip-address  
DHCP(dhcp-config)# exit  
DHCP(config)#

/* e.g
ip dhcp excluded-addresses 192.168.1.1 (ip of the default gateway)

ip dhcp pool poop
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1 (ip of the default gateway)
dns-server 8.8.8.8
exit
*/
```

configure switch DSW2 to function as a DHCP relay agent
```c
DSW2(config)# interface int-num  
DSW2(config-if)# ip helper-address [dhcp-server-ip-address]

// e.g ip helper-address 192.168.0.1
```


---
# Redundancy:

## [[Lab 6a (PAgP, LACP)|Layer-2 EtherChannel]]:

### PAgP:
```c
desirable
// actively asks to bundle
auto 
// will bundle if asked
```

```c
SW1(config)# interface range g1/0/3-4  
SW1(config-if-range)# channel-group 1 mode desirable  
>>> // Creating a port-channel interface Port-channel 1 

SW1(config-if-range)# no shutdown  

DSW1(config)# interface range g1/0/3-4  
DSW1(config-if-range)# channel-group 1 mode auto  
>>> // Creating a port-channel interface Port-channel 1 

DSW1(config-if-range)# no shutdown
```

configure trunk ports:
```c
SW1(config)# interface port-channel 1  
SW1(config-if)# switchport mode trunk

DSW1(config)# interface port-channel 1  
DSW1(config-if)# switchport mode trunk
```

### LACP:
```c
active
// actively asks to bundle
passive
// will bundle if asked
```

```c
SW2(config)# interface range g1/0/1-2  
SW2(config-if-range)# switchport mode trunk  
SW2(config-if-range)# channel-group 2 mode active  
Creating a port-channel interface Port-channel 2  
SW2(config-if-range)# no shutdown  

DSW1(config)# interface range g1/0/1-2  
DSW1(config-if-range)# switchport mode trunk  
DSW1(config-if-range)# channel-group 2 mode passive  
Creating a port-channel interface Port-channel 2  
DSW1(config-if-range)# no shutdown

```

## [[Lab 6b (Layer-3 EtherChannel)|Layer-3 EtherChannel]]:

```c
DSW1(config)# interface range g1/0/5-6  
DSW1(config-if-range)# no switchport  
DSW1(config-if-range)# channel-group 3 mode on  

Creating a port-channel interface Port-channel 3  

DSW2(config)# interface range g1/0/5-6  
DSW2(config-if-range)# no switchport  
DSW2(config-if-range)# channel-group 3 mode on  

Creating a port-channel interface Port-channel 3
```

```c
DSW1(config)# interface port-channel 3  
DSW1(config-if)# no switchport  
DSW1(config-if)# ip address 10.0.0.2 255.255.255.252
  
DSW2(config)# interface port-channel 3  
DSW2(config-if)# no switchport  
DSW2(config-if)# ip address 10.0.0.1 255.255.255.252
```
show etherchannel summary
	
## [[Lab 6c (RSTP) |RSTP]]:

configure root bridge:
```c
DSW1(config)# spanning-tree vlan 1 priority 24576  
or  
DSW1(config)# spanning-tree vlan 1 root primary

/* note:
you can configure any priority value as long as it is the lowest among all connected switches and is in multiples of 4096 from 0 to 61440

The command option root primary will auto-configure itself with a priority value of 24576 or a value in multiples of 4096 lower than 24576 which will be just below the current root priority to take over the root bridge.

default priority = 32768
*/
```

secondary root bridge:
```c
DSW2(config)# spanning-tree vlan 1 root secondary

/* note:
The command option root secondary will auto-configure itself with a priority  
value higher than root primary but lower than all other switches, typically 28672.
*/
```

## [[Lab 6d (HSRP)|HSRP]]
used on the default gateway
configure HSRP on DSW1: (active HSRP router)
```c
DSW1(config)# interface vlan 10  
DSW1(config-if)# standby version 2  
DSW1(config-if)# standby 1 ip 192.168.10.254  
DSW1(config-if)# standby 1 priority 150
```

configure HSRP on DSW2: (standby HSRP router)
```c
DSW2(config)# interface vlan 10  
DSW2(config-if)# standby version 2  
DSW2(config-if)# standby 1 ip 192.168.10.254
```

Similarly, configure HSRP on DSW1 and DSW2 for VLAN 20 using a different HSRP group number, say 2.
```c
DSW1(config)# interface vlan 20  
DSW1(config-if)# standby version 2  
DSW1(config-if)# standby 2 ip 192.168.20.254  
DSW1(config-if)# standby 2 priority 150  
DSW2(config)# interface vlan 20  
DSW2(config-if)# standby version 2  
DSW2(config-if)# standby 2 ip 192.168.20.254

```

Configure HSRP preempt.
```c
DSW1(config)# interface vlan 10  
DSW1(config-if)# standby 1 preempt  
DSW1(config-if)# interface vlan 20  
DSW1(config-if)# standby 2 preempt
```

