### The "Ultimate" Topology

#### **The Layout**

- **The ISP (Internet):** Connected to **R1**.
- **The Edge (R1):** Handles **NAT/PAT**. Connected to **DSW1** via a **Layer 3 link** ($10.1.1.0/30$).
- **The Core (DSW1 & DSW2):** Connected by a **Layer 3 EtherChannel** (Group 10).
- **The Services "V-Shape":** **R2 (DHCP Server)** is connected to **both** DSW1 and DSW2.
    - _Challenge:_ Use **Router-on-a-Stick** (Sub-interfaces) on R2 to handle two separate service VLANs.
- **The Access Layer:** **ASW1** is dual-homed to DSW1 and DSW2 via a **Layer 2 EtherChannel** (Group 1).
- **The Servers:** **Web Server** is on ASW1 (VLAN 30). **PC-A** is on ASW1 (VLAN 10).

---

### 1. The Addressing (The "No Calculator" Test)

- **User LAN (VLAN 10):** `172.20.0.0/22`
    - _Task:_ Find the **last usable IP** for the HSRP VIP.
- **Server DMZ (VLAN 30):** `172.20.4.0/24`
    - _Task:_ Assign the **Web Server** to `.100`.
- **Service Transit (VLAN 100 & 200):** R2 uses `G0/0/1.100` and `G0/0/1.200`.
    - $10.2.2.0/30$ (R2 to DSW1)
    - $10.3.3.0/30$ (R2 to DSW2)
- **Core Link (EtherChannel):** `10.5.5.0/30`.
- **Public IP Range:** `202.10.10.32/29`. (R1 WAN is `.34`, ISP is `.33`).

---

### 2. Technical Objectives (The "Boss Level")

1. **Redundancy (HSRP + STP):**
    
    - DSW1 is **Active** for VLAN 10 and the **Primary Root** for STP.
    - DSW2 is **Active** for VLAN 30 and the **Secondary Root** for STP.
    - _Why?_ This balances the load so one switch isn't doing all the work.
2. **DHCP Relay (Cross-Core):**
    
    - R2 must provide IPs for **VLAN 10**.
    - Since R2 is connected to both DSWs, if the link to DSW1 fails, the DHCP request must travel through DSW2, over the L3 EtherChannel, and still reach R2.
3. **NAT/PAT Requirements:**
    
    - **Static NAT:** The Web Server (`172.20.4.100`) must be reachable via the **second usable** public IP.
    - **PAT:** All internal users must use the **third to fifth** public IPs for internet access.
4. **Routing (The "Brain" Test):**
    
    - **No OSPF.**
    - Use **Floating Static Routes** on R1. If DSW1 dies, R1 should automatically start sending traffic to the LAN via DSW2 through the L3 EtherChannel.

---

### 3. Your Checklist (Don't forget these!)

- **The "Brain" Switch:** Did you type `ip routing` on DSW1 and DSW2?
- **The VLAN 10/30 Trunk:** Did you allow both VLANs on the L2 EtherChannel to ASW1?
- **The R2 Return Path:** Does R2 know how to find the `172.20.x.x` networks? (It needs a route pointing back to the DSWs).
- **The NAT Inside/Outside:** Did you remember to mark the DSW-facing interface on R1 as `ip nat inside`?

### The "Culmination" Question:
In this setup, if **DSW1** is the HSRP Active for VLAN 10, but the cable between **DSW1 and R1** is cut, how does the PC get to the internet?
_Hint: It involves the PC talking to DSW1, DSW1 looking at its routing table, and seeing the L3 EtherChannel to DSW2 as the "next hop" to get to R1._
**Do you want the math answers for the subnets first, or do you want to try drafting the R1 NAT and the DSW Static Routes?**

# removed the ROAS part

# R1:
int g0/0/2
ip add 10.1.1.1 255.255.255.252
no shut
ip nat inside

int g0/0/0
ip add 202.10.10.34 255.255.255.248
no shut
ip nat outside

ip nat inside source static 172.20.4.100 202.10.10.35
access-list 1 permit 172.20.0.0 0.0.3.255
ip nat pool my_pool 202.10.10.36 202.10.10.38 netmask 255.255.255.248
ip nat inside source list 1 pool my_pool overload

ip route 0.0.0.0 0.0.0.0 202.10.10.33
ip route 172.20.0.0 255.255.0.0 10.1.1.2
ip route 10.0.0.0 255.0.0.0 10.1.1.2

# R2:
int g0/0/1
ip add 10.2.2.1 255.255.255.252
no shut

int g0/0/2
ip add 10.3.3.1 255.255.255.252
no shut

ip dhcp excluded-address 172.20.3.252 172.20.3.254
ip dhcp pool ip_pool
network 172.20.0.0 255.255.252.0
default-router 172.20.3.254
dns-server 8.8.8.8

ip route 0.0.0.0 0.0.0.0 10.2.2.2
ip route 0.0.0.0 0.0.0.0 10.3.3.2 10

# DSW1:
ip routing
vlan 10 
vlan 30

int g1/0/2
no sw
ip add 10.1.1.2 255.255.255.252
no shut

int g1/0/1
no sw
ip add 10.2.2.2 255.255.255.252
no shut

int vlan 10
ip add 172.20.3.252 255.255.252.0
no shut
standby version 2
standby 10 ip 172.20.3.254
standby 10 priority 150
standby 10 preempt
ip helper-address 10.2.2.1

int vlan 30
ip add 172.20.4.253 255.255.255.0
no shut
standby version 2
standby 30 ip 172.20.4.254
standby 30 preempt

int range g1/0/23-24
no sw
channel-group 10 mode on
int port-channel 10
no sw
ip add 10.5.5.1 255.255.255.252

int range g1/0/3-4
channel-group 1 mode on
no shut
int port-channel 1
sw mode trunk

spanning-tree vlan 10 root primary
spanning-tree vlan 30 root secondary
ip route 10.3.3.==0== 255.255.255.252 10.5.5.2
ip route 0.0.0.0 0.0.0.0 10.1.1.1 (i dont think i need to specify this)
# DSW2:
ip routing
vlan 10 
vlan 30

int g1/0/2
no sw
ip add 10.3.3.2 255.255.255.252
no shut

int vlan 10
ip add 172.20.3.253 255.255.252.0
no shut
standby version 2
standby 10 ip 172.20.3.254
standby 10 preempt
ip helper-address 10.3.3.1

int vlan 30
ip add 172.20.4.252 255.255.255.0
no shut
standby version 2
standby 30 ip 172.20.4.254
standby 30 priority 150
standby 30 preempt

int range g1/0/23-24
no sw
channel-group 10 mode on
int port-channel 10
no sw
ip add 10.5.5.2 255.255.255.252

int range g1/0/5-6
channel-group 2 mode on
no shut
int port-channel 2
sw mode trunk

spanning-tree vlan 10 root secondary
ip route 0.0.0.0 0.0.0.0 10.5.5.1
# SW1:

int range g1/0/3-4
channel-group 1 mode on
no shut
int port-channel 1
sw mode trunk

int range g1/0/5-6
channel-group 2 mode on
no shut
int port-channel 2
sw mode trunk

int g1/0/1
sw mode access
sw access vlan 10

int g1/0/2
sw mode access
sw access vlan 30

```c
take note of default routes. make sure you can visualise the path to and fro in your mind and if there are pathways for them to get from point to point

i currently use ping to troubleshoot
```