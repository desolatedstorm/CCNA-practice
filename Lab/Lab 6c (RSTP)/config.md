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