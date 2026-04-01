(continued from Lab 6a)

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
