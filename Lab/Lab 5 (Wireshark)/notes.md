trunk at port 1
pc-B at switch 2 port 24
pc-A at switch 1 port 24

s1 ports 3 to router port 0


im pc a 192.168.1.3 "switch2"
tristan is pc b 192.168.1.2

What command would be used to display all entries in the ARP cache?  
arp -a

What command would be used to delete all ARP cache entries (flush ARP cache)?  
arp -d * (with elevated priviledge)

What command would be used to delete the ARP cache entry for 192.168.1.11?
arp -d 192.168.1.11

What is the physical address for the host with IP address of 192.168.1.2?
74-d4-dd-81-5c-a8

What is the physical address for switch s2?
b0-8d-57-73-70-47

What was the first ARP packet?
the first ARP packet was a broadcast packet.
sender MAC address: 74-d4-dd-81-5c-de
sender IP address: 192.168.1.3
target MAC address: ff-ff-ff-ff-ff-ff
target IP address: 192.168.1.1

the second packet was a reply packet?
sender MAC address: 60-b9-c0-3b-97-40
sender IP address: 192.168.1.1
target MAC address: 74-d4-dd-81-5c-de
target IP address: 192.168.1.3