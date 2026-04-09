### **The Physical Layout**

- **R1 (Edge/NAT):** Connected to **DSW1** only (via `G0/0/1`).
- **R2 (DHCP Server):** Connected to **DSW2** only (via `G0/0/1`).
- **DSW1 & DSW2 (The Spine):** Connected to each other via a **Layer 3 EtherChannel** (Group 5).
- **ASW1:** Connected to DSW1 (`G0/0/1`) and DSW2 (`G0/0/2`).
- **Web Server:** Connected directly to **DSW1** (`G0/0/10`).

---

### **1. The Addressing (The Math Challenge)**

- **User LAN (VLAN 10):** `172.16.0.0 /23`
    - _Calculation Task:_ What is the **Last Usable IP** for the HSRP Virtual Gateway?
    - _Calculation Task:_ What is the **Wildcard Mask** for an ACL to permit this subnet?
- **Server DMZ (VLAN 20):** `172.16.2.0 /24`
    - The Web Server is at `.10`.
- **DSW-to-DSW Link (EtherChannel):** `10.5.5.0 /30`
- **R1-to-DSW1 Link:** `10.1.1.0 /30` (R1 is `.1`, DSW1 is `.2`)
- **R2-to-DSW2 Link:** `10.2.2.0 /30` (R2 is `.1`, DSW2 is `.2`)
- **Public IP Range:** `200.100.50.0 /30
    - ISP Gateway: `.1`
    - R1 WAN: `.2`
public IP range: 209.149.211.40/29 (i added this and changed things abit since it didnt make much sense)
---

### **2. The Requirements (The Logic Challenge)**

1. **HSRP:** DSW1 is Active for VLAN 10.
2. **DHCP:** R2 must provide IPs to VLAN 10. (Think: How does the DHCP request get from DSW1, across the EtherChannel, to DSW2, and then to R2?)
3. **STP:** ASW1 must see DSW1 as the Primary Root for VLAN 10.
4. **NAT:** * **Static NAT:** Map Web Server (`172.16.2.10`) to the **last usable** public IP.
    - **PAT:** Overload all other users using the R1 WAN interface IP.
5. **Routing:** Use **OSPF Area 0** for all internal links ($10.x.x.x$ and $172.x.x.x$). Use a **Static Default Route** on R1 for the Internet.

---

### **Why this tests you differently:**

- **No SW3:** You no longer have that "Transit Switch" to hide behind. You must configure **Routed Ports** (`no switchport`) on the DSWs to talk to the routers.
- **The DHCP Path:** Since the DHCP server (R2) is attached to DSW2, but the PC's gateway is DSW1, the `ip helper-address` on DSW1 must point all the way to R2's IP ($10.2.2.1$).
- **EtherChannel:** You have to build a Layer 3 EtherChannel between the two DSWs to allow OSPF to sync.

---

### **Quick "Nuance" Check:**

1. What is the **Wildcard Mask** for a `/23`?
2. On R1, if you use OSPF, do you still need to type `ip route 172.16.0.0...`? (Hint: No, OSPF does it for you).
3. In the `ip nat inside source static` command, which IP comes first—the private or the public?

**Does this layout feel "different" enough to challenge your routing logic? Give the math and the R1 NAT config a shot first.**

# change of plans no ospf involved, just stick to static routes.
---

# r1:
int g0/0/1 
ip add 10.1.1.1 255.255.255.252
no shut
ip nat inside

int g0/0/0
ip add 200.100.50.2 255.255.255.252
no shut
ip nat outside

ip nat inside source static 172.16.2.10 209.149.211.41

ip route 0.0.0.0 0.0.0.0 200.100.50.1
access-list 1 permit 172.16.0.0 0.0.1.255
ip nat pool my_pool 209.149.211.42 209.149.211.46 netmask 255.255.255.248
ip nat inside source list 1 pool my_pool overload

ip route 172.16.0.0 255.255.0.0 10.1.1.2
ip route 10.0.0.0 255.0.0.0 10.1.1.2
# r2:
int g0/0/1
ip add 10.2.2.1 255.255.255.252
no shut

ip dhcp excluded-addresses 172.16.1.252 172.16.1.254
ip dhcp pool ip_pool
network 172.16.0.0 255.255.254.0
default-router 172.16.1.254 
dns-server 8.8.8.8

ip route 0.0.0.0 0.0.0.0 10.2.2.2

# dsw1:
vlan 10
vlan 20
==ip routing==
int g1/0/1
==no sw==
ip add 10.1.1.2 255.255.255.252
no shut

int g1/0/10
sw mode access
sw access vlan 20

int g1/0/6
sw mode access
sw access vlan 10

int vlan 10
ip helper-address 10.2.2.1
ip add 172.16.1.252 255.255.254.0
no shut

int range g1/0/23-24
no sw
channel-group 1 mode on

int port-channel 1 
no sw (no sw dosent work)
ip add 10.5.5.1 255.255.255.252
no shut

int vlan 10
standby version 2 
standby 1 ip 172.16.1.254
standby 1 priority 150
standby 1 preempt

spanning-tree vlan 1 root primary

int vlan 20
ip helper-address 10.2.2.1
ip add 172.16.2.1 255.255.255.0
no shut

ip route 10.2.2.0 255.255.255.252 10.5.5.2

# dsw2:
vlan 10 
vlan 20
==ip routing==
int g1/0/1
==no sw==
ip add 10.2.2.2 255.255.255.252
no shut

int range g1/0/6
sw mode access
sw access vlan 10

int vlan 10
ip helper-address 10.2.2.1
ip add 172.16.1.253 255.255.254.0
no shut

int range g1/0/23-24
no sw
channel-group 1 mode on

int port-channel 1 
no sw
ip add 10.5.5.2 255.255.255.252
no shut

int vlan 10
standby version 2 
standby 1 ip 172.16.1.254
standby 1 priority 150
standby 1 preempt

int vlan 20
ip helper-address 10.2.2.1
ip add 172.16.2.2 255.255.255.0
no shut

spanning-tree ==vlan 10 root== secondary

ip route 10.1.1.0 255.255.255.252 10.5.5.1
# sw1:
int range g1/0/1-24
sw mode access
sw access vlan 10

small things im making clear:
```c
trunk vs access:
DSW to ASW: TRUNK to accomodate multiple vlans
ASW to PC: ACCESS
DSW to Routers: ROUTED PORTS (no sw)
DSW to DSW: ROUTED port channel

[ERROR]:
devices werent able to ping other devices. my key mistake was i forgot to do ip routing on the dsws which caused me alot of confusion

keep in mind routing concepts, in order to ping must be a route there and a route back! devices only know of the route they are directly connected to
```