router r1 at top with lo0 
connected to both dsw
(g0/0/0, g0/0/1)
(g1/0/2, g1/0/23)

two dsw with network redundancy one wire between them
(g1/0/12)

both connected to sw1 (g1/0/1, g1/0/24)
sw1 connected to any port vlan 10 g1/0/2-23

r1 - dsw1 10.0.0.0/30
r1 - dsw2 10.1.1.0/30
lo0 172.16.20.1/24
default gateway of pc = 192.168.10.1

requirements: dhcp, network redundancy, all ping lo0

R1:
int g0/0/0 
ip add 10.0.0.1 255.255.255.252
no shut
int g0/0/1
ip add 10.1.1.1 255.255.255.252
no shut

int loopback 0
ip add 172.16.20.1 255.255.255.0
no shut

ip dhcp excluded-address 192.168.10.1 192.168.10.3
ip dhcp pool my_pool
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8

ip route 192.168.10.0 255.255.255.0 10.0.0.2
ip route 192.168.10.0 255.255.255.0 10.1.1.2 10

DSW1:
ip routing
vlan 10

int g1/0/2
no sw
ip add 10.0.0.2 255.255.255.252
no shut

int range g1/0/1, g1/0/12
sw mode trunk

int vlan 10
ip add 192.168.10.2 255.255.255.0
no shut
standby version 2
standby 1 ip 192.168.10.1
standby 1 priority 150

ip helper-address 10.0.0.1

ip route 0.0.0.0 0.0.0.0 10.0.0.1
ip route 172.16.20.0 255.255.255.0 10.0.0.1

ip route 0.0.0.0 0.0.0.0 192.168.10.3 10
ip route 172.16.20.0 255.255.255.0 192.168.10.3 10

DSW2:
ip routing
vlan 10

int g1/0/23
no sw
ip add 10.1.1.2 255.255.255.252
no shut

int vlan 10
ip add 192.168.10.3 255.255.255.0
no shut
standby version 2
standby 1 ip 192.168.10.1

ip helper-address 10.1.1.1

ip route 0.0.0.0 0.0.0.0 10.1.1.1
ip route 172.16.20.0 255.255.255.0 10.1.1.1

ip route 0.0.0.0 0.0.0.0 192.168.10.2 10
ip route 172.16.20.0 255.255.255.0 192.168.10.2 10

int g1/0/24
sw mode trunk 

SW1:
int g1/0/2-23
sw mode access
sw access vlan 10

int range g1/0/1, g1/0/24
sw mode trunk

```c
NOTE:
the only issue was that i used a differnet router port since on the paper it was g0/1/0 but i used g0/0/1 for convenience sake

they check for: dhcp packets, what is the ip of the dhcp server, port numbers, redundancy (sh standby br), pinging and they used another device to see if it can get the dhcp from there

in order to test the dhcp he just deactivated and activated the ethernet connection there

how they quickly reset:
you dont have to wait for the whole thing to reload before resetting another device you can reset then immediately switch the console cable to another device and reset etc.


WIN + R: ncpa.cpl (to open network panel)
```

