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