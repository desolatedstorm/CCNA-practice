dsw1:
ip routing
int g1/0/3 (to hosts)
no sw
ip add 192.168.3.1 255.255.255.0
no shut
ip ospf 1 area 0
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
