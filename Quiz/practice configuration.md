![[Pasted image 20260129161101.png|650]]
![[Pasted image 20260129161111.png|650]]

set up the ip addresses on the different PCs
on the access switches:
first set up the switches, erase config, reload etc
enable
conf t

sw1:
interface g1/0/6
switchport mode access
switchport access vlan 10
interface g1/0/1
switchport mode trunk

sw2:
interface g1/0/1
switchport mode trunk
interface g1/0/11
switchport mode access
switchport access vlan 10
interface g1/0/18
switchport mode access
switchport access vlan 20

```c
ERROR: (physical error)

did not put the wire that connects the switches
```

legacy inter-vlan routing:
![[Pasted image 20260129164445.png|650]]
![[Pasted image 20260129164455.png|650]]

keep the same set up as the prev
r1:
interface g0/0/0
ip address 192.168.20.1 255.255.255.0
no shut
interface g0/0/1
ip address 192.168.10.1 255.255.255.0
no shut

==s1:
int g1/0/10
sw mode access
sw access vlan 10
s2:
int g1/0/10
sw mode access
sw access vlan 20==

```c
ERROR:

for legacy, we need to do sw access on the ports connecting from the asw to the router. i.e:

s1:
int g1/0/10
sw mode access
sw access vlan 10

s2:
int g1/0/10
sw mode access
sw access vlan 20

why?:
in legacy mode, the router does not expect a vlan tag and will drop the packet if it has one
access port req, auto remove vlan tag.
1 interface = 1 vlan

```

![[Pasted image 20260129165826.png|650]]
![[Pasted image 20260129173114.png|650]]

on the sw1:
int g1/0/6
sw mode access
sw access vlan 10
int g1/0/1
sw mode trunk
int g1/0/10
sw mode trunk
==create vlan 20==

sw2:
int g1/0/1
sw mode trunk
int g1/0/11
sw mode access
sw access vlan 10
int g1/0/18
sw mode access
sw access vlan 20

r1:
int g0/0/1.10
encapsulation dot1q 10
ip address 192.168.10.1 255.255.255.0
no shut
int g0/0/1.20
encapsulation dot1q 20
ip address 192.168.20.1 255.255.255.0
no shut
==int g0/0/1
no shut==

```c
ERROR:

the router interfaces kept saying administratively down (and the port had no light) even though i went into the subinterfaces and ran no shutdown

the issue was that i did not run no shut on the parent port. (g0/0/1) 
```

![[Pasted image 20260129173016.png|650]]![[Pasted image 20260129173041.png|601]]

sw1:
int g1/0/1
sw mode trunk
int g1/0/6
sw mode access
sw access vlan 10
int g1/0/10
sw mode trunk

sw2:
int g1/0/1
sw mode trunk
int g1/0/10
sw mode trunk
int g1/0/11
sw mode access
sw access vlan 10
int g1/0/18
sw mode access
sw access vlan 20

dsw1:
ip routing (to enable inter vlan routing)
interface vlan 10
ip address 192.168.10.1 255.255.255.0
interface vlan 20
ip address 192.168.20.1 255.255.255.0

```c
ERROR:

I did not create the vlans in the DSW first. i.e:

vlan 10
vlan 20

also need to make the ports on the dsw trunk ports
```

![[Pasted image 20260212162445.png|650]]
![[Pasted image 20260215140845.png|650]]

sw1:
interface range g1/0/3-4
channel-group 1 mode desirable
no shutdown
interface port-channel 1 
switchport mode trunk

sw2:
interface range g1/0/1-2
switchport mode trunk
channel-group 2 mode active

dsw1:
interface range g1/0/1-2
switchport mode trunk
channel-group 2 mode active
no shutdown

interface range g1/0/3-4
channel-group 1 mode desirable
no shutdown
interface port-channel 1 
switchport mode trunk

![[Pasted image 20260217031107.png|650]]

dsw2:
int range g1/0/5-6
no switchport
channel-group 3 mode on
interface port-channel 3
no switchport
ip address 10.0.0.1 255.255.255.252

dsw1:
int range g1/0/5-6
no switchport
channel-group 3 mode on
interface port-channel 3
no switchport
ip address 10.0.0.2 255.255.255.252

![[Pasted image 20260223102908.png|650]]
rstp with dsw1 as root

dsw1:
interface range g1/0/1-2
channel-group 1 mode desirable
no shut
interface port-channel 1
switchport mode trunk
spanning-tree vlan 1 root primary

dsw2:
interface range g1/0/1-2
channel-group 1 mode desirable
no shut
interface port-channel 1
switchport mode trunk
spanning-tree vlan 1 root secondary

![[Pasted image 20260223151658.png|650]]
![[Pasted image 20260228155405.png|650]]

sw1:
int range g1/0/1-2
sw mode trunk
int g1/0/6
sw mode access
sw access vlan 10
vlan 20

sw2:
int range g1/0/1-2
sw mode trunk
int g1/0/6
sw mode access
sw access vlan 20
vlan 10

dsw1:
int range g1/0/1-2, g1/0/12
sw mode trunk
int vlan 10 
ip address 192.168.10.1 255.255.255.0
no shut
int vlan 20 
ip address 192.168.20.1 255.255.255.0
no shut
exit
ip routing
int vlan 10
standby version 2
standby 1 ip 192.168.10.254
standby 1 priority 150
standby 1 preempt
int vlan 20
standby version 2
standby 2 ip 192.168.20.254
standby 2 priority 150
standby 2 preempt

dsw2:
int range g1/0/1-2, g1/0/12
sw mode trunk
int vlan 10 
ip address 192.168.10.2 255.255.255.0
no shut
int vlan 20 
ip address 192.168.20.2 255.255.255.0
no shut
exit
ip routing
int vlan 10
standby version 2
standby 1 ip 192.168.10.254
int vlan 20 
standby version 2 
standby 2 ip 192.168.20.254

![[Pasted image 20260226125257.png|650]]![[Pasted image 20260226125227.png|650]]

dsw2:
ip routing
int vlan 1
ip add 192.168.0.1 255.255.255.0
no shut
int g1/0//1
==no switchport==
ip add 10.1.1.1 255.255.255.252
no shut
ip route 192.168.1.0 255.255.255.0 g1/0/1 10.1.1.2 

r1:
int g0/0/0 (what i connected to)
ip add 10.1.1.2 255.255.255.252
no shut
int g0/0/1 (what i connected to)
ip add 192.168.1.1 ==255.255.255.0==
no shut
interface Loopback 0
ip address 209.165.20.225 255.255.255.224
no shut
interface Loopback 1
ip address 198.133.219.1 255.255.255.0
no shut
exit
ip route 192.168.0.0 255.255.255.0 g0/0/0 10.1.1.1

```c
ERROR:
no switchport
this command turns a switch port into a routed port, enabling it to have an ip address
```


![[Pasted image 20260226154304.png|650]]![[Pasted image 20260226155427.png|650]]

dsw1:
ip routing
int g1/0/3 (to hosts)
no sw
ip add 192.168.3.1 255.255.255.0
no shut
==ip ospf 1 area 0==
int g1/0/1 (to R1)
no sw
ip add 192.168.13.2 255.255.255.252
no shut
ip ospf 1 area 0
int g1/0/2 (to R2)
no sw
ip add 192.168.23.2 255.255.255.252
no shut
ip ospf 1 area 0
router ospf 1
passive-interface g1/0/3

r1:
int g0/0/0 (to R2)
ip add 192.168.12.1 255.255.255.252
no shut
int g0/0/1 (to dsw1)
ip add 192.168.13.1 255.255.255.252
no shut
int g0/0/2 (to hosts)
ip add 192.168.1.1 255.255.255.0
no shut
router ospf 1
network 192.168.1.0. 0.0.0.255 area 0
network 192.168.12.0. 0.0.0.3 area 0
network 192.168.13.0. 0.0.0.3 area 0
passive-interface g0/0/2

r2:
int g0/0/0 (to R1)
ip add 192.168.12.2 255.255.255.252
no shut
int g0/0/1 (to dsw)
ip add 192.168.23.1 255.255.255.252
no shut
int g0/0/2 (to hosts)
ip add 192.168.2.1 255.255.255.0
no shut
router ospf 1
network 192.168.2.0 0.0.0.255 area 0
network 192.168.12.0 0.0.0.3 area 0
network 192.168.23.0 0.0.0.3 area 0
passive-interface g0/0/2

```c
you have to run ospf on all interfaces for them to know all the diff subnets
run passive interface after (for security reasons)
```

![[Pasted image 20260307162509.png|650]]

r1:
int g0/0/0
ip add 172.17.9.113 255.255.255.252
no shut
int g0/0/1
ip add 192.168.1.1 255.255.255.0
no shut

203.149.210.224/29 -> 203.149.210.225-203.149.210.230

static NAT:
ip route 0.0.0.0 0.0.0.0 172.17.9.114
ip nat inside source ==static== 192.168.1.20 203.149.210.225
==int g0/0/1
ip nat inside
int g0/0/0
ip nat outside==

dynamic NAT:
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat pool my_pool 203.149.210.226 203.149.210.230 netmask 255.255.255.255.248
ip nat inside source list 1 pool my_pool

NAT/PAT pool overload:
no ip nat inside source list 1 pool my_pool
ip nat inside source list 1 pool my_pool overload


![[Pasted image 20260309223431.png|650]]

dsw2:
ip routing
int g1/0/1
no sw
ip add 192.168.0.2 255.255.255.252
no shut
int vlan 1
ip add 192.168.1.1 255.255.255.0
no shut
int g1/0/2
ip helper-address 192.168.0.1

dhcp:
int g0/0/1 
ip add 192.168.0.1 255.255.255.252
==no shut==
ip dhcp excluded-addresses 192.168.1.1
ip dhcp pool my_pool
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
exit
==ip route 192.168.1.0 255.255.255.0 g0/0/1 192.168.0.2==

```c
ERROR:
default-router: tells the PC who they have to reach in order to connect to the internet

not sure where to put the interface of the dsw2 relay agent command as well
```
