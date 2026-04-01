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
no shut
ip dhcp excluded-addresses 192.168.1.1
ip dhcp pool my_pool
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
exit
ip route 192.168.1.0 255.255.255.0 g0/0/1 192.168.0.2

```c
ERROR:
default-router: tells the PC who they have to reach in order to connect to the internet

not sure where to put the interface of the dsw2 relay agent command as well
```
