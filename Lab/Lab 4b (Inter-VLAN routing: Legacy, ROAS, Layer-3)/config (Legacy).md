r1:
interface g0/0/0
ip address 192.168.20.1 255.255.255.0
no shut
interface g0/0/1
ip address 192.168.10.1 255.255.255.0
no shut

s1:
int g1/0/10
sw mode access
sw access vlan 10
s2:
int g1/0/10
sw mode access
sw access vlan 20

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
