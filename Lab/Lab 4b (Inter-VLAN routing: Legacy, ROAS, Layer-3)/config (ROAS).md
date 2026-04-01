sw1:
int g1/0/6
sw mode access
sw access vlan 10
int g1/0/1
sw mode trunk
int g1/0/10
sw mode trunk
create vlan 20

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
int g0/0/1
no shut

```c
ERROR:

the router interfaces kept saying administratively down (and the port had no light) even though i went into the subinterfaces and ran no shutdown

the issue was that i did not run no shut on the parent port. (g0/0/1) 
```
