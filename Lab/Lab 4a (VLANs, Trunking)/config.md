first, set up the ip addresses on the different PCs

on the access switches:
set up the switches, erase startup-config, reload.. 

sw1:
interface g1/0/6
switchport mode access
switchport access vlan 10
interface g1/0/1
switchport mode trunk

sw2:
interface g1/0/1
switchport mode trunk
interface g1/0/11
switchport mode access
switchport access vlan 10
interface g1/0/18
switchport mode access
switchport access vlan 20

```c
ERROR: (physical error)

did not put the wire that connects the switches
```