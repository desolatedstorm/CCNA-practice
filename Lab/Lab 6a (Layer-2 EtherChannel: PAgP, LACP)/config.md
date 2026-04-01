sw1:
interface range g1/0/3-4
channel-group 1 mode desirable
no shutdown
interface port-channel 1 
switchport mode trunk

sw2:
interface range g1/0/1-2
switchport mode trunk
channel-group 2 mode active

dsw1:
interface range g1/0/1-2
switchport mode trunk
channel-group 2 mode active
no shutdown
interface range g1/0/3-4
channel-group 1 mode desirable
no shutdown
interface port-channel 1 
switchport mode trunk
