dsw2:
ip routing
int vlan 1
ip add 192.168.0.1 255.255.255.0
no shut
int g1/0//1
no switchport
ip add 10.1.1.1 255.255.255.252
no shut
ip route 192.168.1.0 255.255.255.0 g1/0/1 10.1.1.2 

r1:
int g0/0/0 (what i connected to)
ip add 10.1.1.2 255.255.255.252
no shut
int g0/0/1 (what i connected to)
ip add 192.168.1.1 255.255.255.0
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
