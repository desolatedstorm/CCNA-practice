table 6C
my computer configuring SW1, ethernet cable is connected to SW2
# 2.1
## PC-A
OUI of the MAC address of the NIC of PC-A: 74-D4-DD
serial number portion: 81-5C-CE
name of vendor that manufactured the NIC: Quanta Computer Inc.

## PC-B (me)
OUI of the MAC address of the NIC of PC-B: 74-D4-DD
serial number portion: 81-5C-DC 
name of vendor that manufactured the NIC: Quanta Computer Inc.
# 2.2
## PC-A
base ethernet MAC address of switch S2: 
MAC address for port G1/0/23 on your switch SW2: 
MAC address for port G1/0/24 on your switch SW2: 
## PC-B (me)
base ethernet MAC address of switch S1: b0:8d:57:6d:a5:80
MAC address for port G1/0/6 on your switch SW1: b08d.576d.a586

# 3.3
#### What is the MAC address ffff.ffff.ffff used for?  
it is the broadcast mac address

#### What type of MAC address does 0180.c200.0000 belong to? What is it used for?
it is a multicast mac address, specifically the spanning tree protocol for bridges (IEEE 802.1 D)
spanning tree protocol (SPT)
- prevents networks loops and protects network from broadcast storms

# 3.5
#### dynamically learned MAC addresses and corresponding ports
74d4.dd81.5cce G1/0/6
74d4.dd81.5cdc G1/0/1
b08d.5773.e281 G1/0/1

# 3.7
pc-a connected to switch 1 via port 23
pc-b connected to switch 2 via port 6
switch 1 and switch 2 connected via port 1

when pc-a pings pc-b there is a broadcast to all the ports except for where it came from, pc-b replies to switch 2 and then switch 2 will send the reply to switch 1 via the port 1, so to switch 1, it learns the pc-b's mac address and relates it to port 1

# 3.8
it is empty 


# 4.4
highlighted bytes:
74 d4 dd 81 5c dc 74 d4 dd 81 5c ce 08 00

destination:
74 d4 dd 81 5c dc 

source:
74 d4 dd 81 5c ce

type:
08 00

you can see its arranged in (destination) (source) (type)
https://www.iana.org/assignments/ieee-802-numbers/ieee-802-numbers.xhtml
08 00 is the hex value of the ether type for icmp

#### Can you deduce why the need for Type field in Ethernet II header?
required since there are so many different ether types