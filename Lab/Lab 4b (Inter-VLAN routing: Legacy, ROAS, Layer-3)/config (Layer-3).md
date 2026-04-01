
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
