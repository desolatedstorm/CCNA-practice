chapter 1-7:
### **Question 1: EtherChannel & LACP Negotiation**

**Topology:**

- **Switch-A** (Cisco) is connected to **Switch-B** (Non-Cisco) via four GigabitEthernet links: `G0/1`, `G0/2`, `G0/3`, and `G0/4`.
- **Switch-A** Configuration: `channel-group 1 mode active`
- **Switch-B** Configuration: `channel-group 1 mode passive`
- A spanning-tree loop exists elsewhere in the network, making **Switch-A** the Root Bridge for VLAN 10.

**Which of the following is TRUE regarding this setup?** 

A) The EtherChannel will fail to form because the modes are incompatible. 
B) The EtherChannel will form, and STP will see all four physical links as individual paths, blocking three of them. 
C) The EtherChannel will form, and STP will treat the logical `Port-Channel 1` interface as a single forwarding link. 
D) The EtherChannel will fail to form because Switch-B is not a Cisco device and cannot run PAgP.

##### Answers
```
C.

three methods of configuring etherchannels:
1. Static Configuration
2. Port Aggregation Protocol (PAgP)
3. Link Aggregation Control Protocol (LACP)
   
PAgP is Cisco-only. (desirable/auto)
LACP is the industry standard (active/passive)

PAgP is Cisco-only. LACP (Active/Passive) is the Industry Standard (IEEE 802.3ad). Because Switch-B is non-Cisco, it must use LACP.
```

---

### **Question 2: RSTP Path Cost & Tie-Breakers**

**Topology:**

- **Root Bridge (S1)** is connected to **S2** and **S3**.
- **S2** and **S3** are connected to each other via two parallel 10Gbps links (Cost = 2000 each).
- **S2** connects to **S1** via a 1Gbps link (Cost = 20000).
- **S3** connects to **S1** via a 10Gbps link (Cost = 2000).
- **S2** has a higher Bridge ID than **S3**.


**On Switch S2, which port will be elected as the Root Port?** 

A) The interface connected to S1, because it is a direct link to the Root Bridge. 
B) The interface connected to S3, because the cumulative Root Path Cost (4000) is lower than the direct link (20000).
C) Both links to S3 will be Root Ports, and traffic will be load-balanced via EtherChannel automatically. 
D) No port will be a Root Port; S2 will become the Backup Root Bridge.

##### Answers
```
B.

port with lowest root path cost is elected as root port

if two or more ports are calculated to have the same interface root path cost, tie-breaker rule is applied in the order as follows:
1. Port receiving BPDU with lowest sender BID (not Root ID);
2. Port receiving BPDU with lowest sender port priority in Port ID;
3. Port receiving BPDU with lowest sender port number in Port ID.
   
Root Path Cost is the cumulative cost to reach the Root Bridge.

Path 1 (Direct S2 → S1): Cost = 20,000.

Path 2 (S2 → S3 → S1): Cost = 2,000 (S3 to S1)+2,000 (S2 to S3)= 4,000.

Since 4,000<20,000, S2 will actually choose to send traffic through S3 to get to the Root Bridge.

The Tie-Breaker: You asked if costs add up. No, costs don't add up parallelly. If you have two 10G links between S2 and S3, the cost is still 2000. S2 will then use the Tie-Breaker (Port ID) to pick which of those two physical wires becomes the Root Port
```

---

### **Question 3: OSPF DR/BDR Election & States**

**Topology:**

- Four routers (R1, R2, R3, R4) are connected to a single Ethernet Switch (Broadcast Multi-access).
- **R1:** Priority 255, Router ID 1.1.1.1
- **R2:** Priority 1, Router ID 2.2.2.2
- **R3:** Priority 1, Router ID 3.3.3.3
- **R4:** Priority 0, Router ID 4.4.4.4

- R1 is powered on first and becomes the DR. R2 is powered on second and becomes the BDR. Later, **R5** (Priority 255, Router ID 9.9.9.9) is added to the network.


**What is the status of R5 after it fully converges with the neighbors?** 

A) R5 preempts R1 and becomes the new DR because it has a higher Router ID. 
B) R5 remains a DROTHER because DR/BDR elections are non-preemptive. 
C) R5 becomes the BDR, demoting R2 to a DROTHER. 
D) R5 transits to the `Full` state with R1 and R2, but stays in the `2-Way`state with R3 and R4.

##### Answers
```
B.

OSPF elections are non-preemptive. Even if a "better" router (R5) shows up, it won't kick the current King (DR) off the throne
```

---

### **Question 4: HSRP and Gratuitous ARP**

**Topology:**

- **Dist-1** (Active) and **Dist-2** (Standby) provide a Virtual IP `192.168.1.1` (VMAC `0000.0c07.ac01`) for VLAN 1.
- The physical link between **Access-Switch** and **Dist-1** is severed.
- **Dist-2** misses 4 consecutive Hello packets from **Dist-1**.


**How does the Access-Switch learn to send traffic to Dist-2 instead of Dist-1?** 

A) The PC hosts perform a new ARP request for the Virtual IP. 
B) **Dist-2** sends a Gratuitous ARP (GARP) telling the switch to map the Virtual MAC to **Dist-2's** physical port. 
C) The switch's MAC table times out after 300 seconds, and it begins flooding traffic until **Dist-2**responds. 
D) **Dist-1** sends a "Resign" message to the Access-Switch before the link fails.

##### Answers
```
B.

the standby switch updates the switch's mac address table using GARP so traffic is redirected without any issues.
```

---

### **Question 5: OSPF Cost Calculation & Reference Bandwidth**

**Topology:**

- **Router-X** has two paths to Subnet A.
- **Path 1:** via a 10Gbps Fiber link.
- **Path 2:** via a 1Gbps Copper link.
- The network administrator has **not** changed the default `auto-cost reference-bandwidth`.


**Which path will OSPF install in the routing table?** 

A) Path 1 only, because it has significantly more bandwidth. 
B) Path 2 only, because Copper interfaces have a lower default administrative distance. 
C) Both Path 1 and Path 2, because OSPF sees them both as having a cost of 1. 
D) Neither; OSPF requires a manual cost configuration for links over 100Mbps.

##### Answers
```
C.

10Gbps = 100/10000=0.01 rounded to 1.
    
1Gbps = 100/1000=0.1 rounded to 1.
    
Since both paths have a cost of 1, OSPF thinks they are equal speed and will perform Equal Cost Multi-Path (ECMP)load balancing. This is bad because you're wasting the 10G link's potential!
```


### **Question 6: EtherChannel Misconfiguration & STP**

**Topology:**

- **Switch-A** and **Switch-B** are connected by two 1Gbps links (`G0/1` and `G0/2`).
- **Switch-A** is configured with `channel-group 1 mode desirable`.
- **Switch-B** has **no** EtherChannel configuration; its ports are just standard access ports in VLAN 10.
- **Switch-A** is the Root Bridge.

**How will Spanning Tree (PVST+ or RSTP) handle this specific link?** 
A) The EtherChannel will form anyway because "desirable" is an aggressive mode. 
B) Switch-A will bundle the ports, but Switch-B will see a loop and put one of its ports (`G0/1` or `G0/2`) into a Blocking/Discarding state. 
C) Switch-A will fail to bundle the ports. Both `G0/1` and `G0/2` will operate as individual links, and Switch-B will block one of them to prevent a loop. 
D) Both switches will shut down the ports automatically due to an "EtherChannel Guard" error.

##### Answers
```
C.

If one side is "Desirable" (Cisco PAgP) and the other side has no configuration, the negotiation fails. The ports fall back to being individual physical links. Since they are individual links, STP sees a loop and blocks one of them
```

---

### **Question 7: RSTP Convergence & Port Roles**

**Topology:**

- **S1 (Root Bridge)** connects to **S2** and **S3**.
- **S2** and **S3** are connected to each other.
- All links are 1Gbps (Cost 20,000).
- **Bridge IDs:** S1 (32768:AAAA), S2 (32768:BBBB), S3 (32768:CCCC).
- A PC is connected to **S3** on port `G0/10`.

**Which of the following describes the port roles correctly after convergence?** 
A) The port on **S3** connecting to **S2** will be a **Designated Port** because S3 has a higher MAC address. 
B) The port on **S2** connecting to **S3** will be a **Designated Port** because S2 has a lower Bridge ID than S3. 
C) The PC port on **S3** will be an **Alternate Port** until it receives a BPDU from the PC. 
D) All ports on **S1** are **Root Ports** because it is the Root Bridge.

##### Answers
```
B.

ROOT BRIDGE:
- lowest priority (default 32768)
- lowest mac address (if priority tied)
  
all ports on root port are Designated Ports. (DP)

ROOT PORT:
- lowest root path cost
- lowest neighbour BID
- lowest neighbout PID (if there is two wires to the same neighbour, use the one plugged into the neighbour's lower port number)

DESIGNATED PORT:
- lowest root path cost
- lowest BID 

```

---

### **Question 8: OSPF Passive Interfaces & Adjacencies**

**Topology:**

- **R1** and **R2** are connected via their `G0/0` interfaces on the `10.1.1.0/24` network.
- OSPF is enabled on both routers for this subnet.
- The admin executes `passive-interface G0/0` on **R1** to "reduce CPU load."

**What is the immediate result of this configuration change?** 
A) R1 will stop sending Hellos but will still process Hellos from R2, maintaining the adjacency. 
B) The OSPF adjacency between R1 and R2 will fall to `Down`, and R2 will remove R1's routes from its table. 
C) R1 will still advertise the `10.1.1.0/24` network to other OSPF neighbors, but it will no longer accept OSPF updates from R2. 
D) Both A and C are correct.

##### Answers
```
C.

a passive interface ignores OSPF traffic
the interface will no longer send or receive OSPF routing updates
```

---

### **Question 9: HSRP Load Balancing (MHSRP)**

**Topology:**

- Two Distribution switches, **D1** and **D2**, serve two VLANs (VLAN 10 and VLAN 20).
- **D1** is configured as HSRP Priority 110 for Group 10 (VLAN 10) and Priority 90 for Group 20 (VLAN 20).
- **D2** is configured as HSRP Priority 90 for Group 10 and Priority 110 for Group 20.
- All priorities are default otherwise.

**If D1's uplink to the Core fails, but the switch stays powered on, what happens to the traffic for VLAN 20?** 
A) VLAN 20 traffic remains on **D2** because **D2** was already the Active router for that group. 
B) VLAN 20 traffic shifts to **D1**because HSRP groups fail over together. 
C) VLAN 20 traffic is dropped because **D1** is the primary switch for the building. 
D) **D2** becomes Active for both VLAN 10 and VLAN 20 simultaneously.

##### Answers
```
A.

for vlan 20, D2 was already the active router. 
```

---

### **Question 10: OSPF Path Selection (The Cost Tie)**

**Topology:**

- **Router-A** needs to reach network `172.16.1.0/24`.
- **Path 1:** A single Static Route with Administrative Distance (AD) of 110 and a metric of 10.
- **Path 2:** An OSPF-learned route with AD 110 and a metric of 50.

**Which route will Router-A install in its Routing Table?** 
A) Path 2, because OSPF is a dynamic protocol and more reliable than a manual static route. 
B) Path 1, because when the AD is a tie, the router chooses the route with the lowest metric. 
C) Both routes, and the router will load-balance traffic between them. 
D) Neither; this is a configuration conflict that will cause the "Route Flap" error in the logs.

##### Answers
```
B.

router will pick a path in the following order:
1. longest prefix match (e.g /24 better than /16)
2. lowest AD
	- connected = 0
	- static = 1
	- OSPF = 110
3. smallest metric
   
OSPF when are the routes uploaded into the routing table?
- if there is an OSPF and static route for the same route, only the static route is kept.
- the router will put multiple paths for the same route if there are different paths for the same network with the same AD and metric, OSPF will do this automatically for up to 4 paths.
  
OSPF only puts the fastest way to get to a subnet.
```

<hr>

### **Question 11:**

**Topology:** * **S1** is the Root Bridge.

- **S2** and **S3** are both 1 hop away from S1 via 1Gbps links (Cost 20,000).
- **S2** (MAC: 0000.AAAA.AAAA) and **S3** (MAC: 0000.BBBB.BBBB) are connected to each other via a single 1Gbps link.

**On that link between S2 and S3, which port is the Designated Port?** 
A) S2's port, because it has the lower MAC address. 
B) S3's port, because it has the higher MAC address. 
C) Both are Designated Ports because they have the same path cost. 
D) There is no Designated Port; one is a Root Port and one is an Alternate Port.

##### Answers
```
A.

cost is tied, lower BID won
```

<hr>

### **Question 12 (AD vs Metric):**

A router learns about network `10.5.5.0/24` from three sources:

1. **OSPF:** Metric 50, AD 110.
2. **Static Route:** Metric 0, AD 150 (Floating static).
3. **EIGRP:** Metric 1000, AD 90.

**Which route is placed in the routing table?** 
A) OSPF, because it has the lowest metric. 
B) Static, because static routes are always preferred. 
C) EIGRP, because it has the lowest AD. 
D) All three, and the router will load-balance.

##### Answers
```
C.

EIGRP has lowest AD. although it has a high metric, the router trusts it more
```

<hr>

### **Question 13:** A router has the following three entries in its OSPF database for the same destination:

1. `192.168.1.0/24` via G0/0 (Cost 20)
2. `192.168.1.0/24` via G0/1 (Cost 10)
3. `192.168.1.0/24` via G0/2 (Cost 10)

**How many routes will be installed in the Routing Table?** 
A) 1 
B) 2 
C) 3 
D) 0

##### Answers
```
B.

two routes with cost = 10 were tied for best so both go into the table.
path with cost = 20 is ignored
```

<hr>

### **Question 14:** 

Switch-A and Switch-B are connected by two cables (not in an EtherChannel). Switch-A is the Root Bridge. On Switch-B, both ports have the same path cost to the Root. 

How does Switch-B decide which port to make the **Root Port**? 
A) It picks the port connected to the higher port number on Switch-A. 
B) It picks the port connected to the lower port number on Switch-A. 
C) It picks the port with the lower MAC address on Switch-B. 
D) It blocks both ports to be safe.

##### Answers
```
B.

tie breaker: look at sender port number and picks the lowest one
```

---

### **Question 15**

This one combines **Redundancy, EtherChannel, and OSPF**.

**Topology:**

1. **Router-A** is connected to **Router-B** via an **EtherChannel** made of two 100Mbps links.
2. **Router-A** is also connected to **Router-B** via a single **1Gbps** fiber link.
3. OSPF is running on all interfaces.
4. The `reference-bandwidth` is set to **1000** (1Gbps).

**Which path will OSPF choose to reach Router-B?**

- _Hint 1:_ Calculate the cost of the 1Gbps link (1000/1000).
- _Hint 2:_ Calculate the cost of the EtherChannel (1000/200).

A) The EtherChannel, because it has two physical wires and is more redundant. 
B) The 1Gbps Fiber link, because its OSPF cost is lower. 
C) Both, because OSPF will load-balance between different link types. 
D) Neither; OSPF will crash because you can't mix EtherChannel and Fiber.

##### Answer:
```
B.

Here is why:

- The 1Gbps Path: Cost = 1000/1000=1.
    
- The EtherChannel Path: Since it is two 100Mbps links bundled, the logical bandwidth is 200Mbps. Cost = 1000/200=5.
    
- The Result: Since 1 is lower than 5, OSPF ignores the redundant EtherChannel and puts the Fiber link in the routing table.
```

<hr>
### **Question 16: The OSPF Reference Bandwidth Trap**

**Scenario:** A network has a mix of 10Gbps and 1Gbps links. The administrator wants OSPF to prefer the 10Gbps links. He executes the following command on all routers: `Router(config-router)# auto-cost reference-bandwidth 1000`

**Topology:**

- **Path A:** A single 10Gbps link.
- **Path B:** Two 1Gbps links bundled in an EtherChannel.

**What are the OSPF costs for Path A and Path B?** 
A) Path A = 1, Path B = 1 
B) Path A = 0.1, Path B = 0.5 
C) Path A = 1, Path B = 5 
D) Path A = 10, Path B = 5

##### Answer:
```
A.

default reference bandwidth = 100 MBps
1 Gbps = 1000 MBps
10 Gbps = 10000 MBps

Path A: 1000/10000 = 0.1
Path B: 1000/2000 = 0.5

since OSPF cannot have a cost below 1, it rounds them both up to 1.
```

---

### **Question 17: RSTP Cumulative Path Cost**

**Topology:**

- **S1** (Root Bridge)
- **S1** is connected to **S2** via a 100Mbps link.
- **S2** is connected to **S3** via a 1Gbps link.
- **S1** is also connected directly to **S3** via a 10Mbps link.

**What is the Root Path Cost for S3, and which port will it choose as its Root Port?** 
_(STP Costs: 10Mbps=100, 100Mbps=19, 1Gbps=4)_

A) Cost 100; the direct link to S1. 
B) Cost 23; the path through S2. 
C) Cost 4; the path through S2. 
D) Cost 119; S3 will become a second Root Bridge.

##### Answer:
```
B.

S3 > S1 (100)
S3 > S2 > S1 (4 + 19 = 23)

23 is cheaper than 100, so S3 will block its direct link to the root bridge and send its traffic via S2 instead.
```

---

### **Question 18: LACP Election & Priority**

**Topology:** Two switches are connected via four links. You want to ensure that `G0/1` and `G0/2` are always the active links in the bundle, while `G0/3` and `G0/4` stay in "hot-standby" (LACP supports max 8 active links, but let's assume we limited this bundle to 2).

**Which parameter would you change to force the switch to prefer G0/1 and G0/2?** 
A) LACP System Priority 
B) LACP Port Priority 
C) Spanning-tree Port Priority 
D) EtherChannel Group ID

##### Answer:
```
B.

If you have 4 links in an EtherChannel but only want 2 to be active, you change the LACP Port Priority. The links with the lowest priority numbers are chosen to be active first.
```

---

### **Question 19: HSRP Priority Calculation**

**Scenario:**

- **Dist-1** (Active) has a priority of **105**.
- **Dist-2** (Standby) has a priority of **100**.
- **Dist-1** is tracking its uplink to the Core. The tracking command is: `track 1 interface Gig0/0 line-protocol`.
- The default decrement for HSRP tracking is **10**.
- **Preempt** is enabled on both routers.

**The uplink on Dist-1 goes down. What happens?** 
A) Dist-1 priority becomes 95; Dist-2 becomes Active. 
B) Dist-1 priority becomes 105; Dist-2 stays Standby because 5 is not a big enough difference. 
C) Dist-1 priority becomes 95; Dist-1 stays Active because it was already the boss. 
D) Dist-2 becomes Active immediately, regardless of priority, because the link is physically down.

##### Answer:
```
A.

- The Math: 105(Active)−10(Decrement)=95.
- The Switch: Since 95 is less than 100 (Standby), and preempt is on, the Standby router will immediately take over.
```

---

### **Question 20: Summary Route Logic**

**Scenario:** A router needs to summarize the following networks into one single static route to send to the ISP:

- `10.1.0.0/24`
- `10.1.1.0/24`
- `10.1.2.0/24`
- `10.1.3.0/24`

**What is the most efficient summary route?** 
A) `10.1.0.0 255.255.252.0` 
B) `10.1.0.0 255.255.255.0` 
C) `10.1.0.0 255.255.0.0` 
D) `10.0.0.0 255.0.0.0`

##### Answer:
```
A.

- **The Math:** You are looking for the common bits in the third octet.
    
    - 0: `00000000`
        
    - 1: `00000001`
        
    - 2: `00000010`
        
    - 3: `00000011`
        
- They all share the first 6 bits (`000000`). Since the first two octets are also the same (16 bits), the total mask is 16+6=22.
    
- A `/22` mask is `255.255.252.0`. Perfect!
```

---
### **Question 24: Ethernet Framing (The "Size" Logic)**

**Scenario:** A Switch receives a frame where the `Type/Length` field contains the hexadecimal value **0x0800** (which is **2048** in decimal). **What does this value indicate to the Switch?** 

A) The frame is an IEEE 802.3 frame, and the data field is 2048 bytes long. 
B) The frame is an Ethernet II frame, and it is carrying an IPv4 packet. 
C) The frame is corrupted because the maximum data length is 1500 bytes. 
D) The frame is a Multicast frame because the value is above 1536.

##### Answer:
```
B. 

ethernet II vs IEEE 802.3

if the value is 1536 (0x0600) or higher, it represents Type
in Ethernet II, 0x0800 is the specific code for IPv4

if it was a Length field (IEEE) the maximum value allowed would be 1500
anything higher must be a type.
```

---

### **Question 25: The Layer 3 Switch "SVI" Logic**

**Topology:**

- A Multi-Layer Switch (MLS) has **VLAN 10** and **VLAN 20** configured.
- `Interface VLAN 10` has IP `10.1.10.1/24`.
- `Interface VLAN 20` has IP `10.1.20.1/24`.
- A PC in VLAN 10 wants to ping a PC in VLAN 20.

**Which of the following must be true for the ping to succeed?** 

A) The PCs must be connected to "Trunk" ports on the MLS. 
B) The MLS must have "IP Routing" enabled to move traffic between the SVIs. 
C) The PCs do not need a Default Gateway because they are on the same physical switch. 
D) The Destination MAC address of the frame sent by the PC will be the MAC address of the destination PC.

##### Answer
```
B.

by default a layer 3 switch acts like a layer 2 switch until told otherwise using "ip routing"
```

---

### **Question 26: ARP & The Default Gateway**

**Scenario:** PC-A (`192.168.1.5/24`) wants to send data to a Web Server (`8.8.8.8`). PC-A has no entries in its ARP cache. **Who will PC-A send an ARP Request for?** 

A) The MAC address of the Web Server (`8.8.8.8`). 
B) The MAC address of its Default Gateway (e.g., `192.168.1.1`). 
C) A Broadcast ARP for all devices on the Internet. 
D) It won't use ARP; it will use DNS to find the MAC address.

##### Answer
```
B.
```

---

### **Question 27: IPv4 Multicast MAC Mapping**

**Scenario:** A host is listening to the Multicast IP **224.1.2.3**. **Based on your notes, what will be the corresponding Destination MAC address for frames sent to this group?** 

A) `01:00:5E:01:02:03` 
B) `FF:FF:FF:01:02:03` 
C) `01:00:5E:81:02:03` 
D) `00:00:5E:01:02:03`

##### Answer
```
A.

The first 25 bits of the MAC are fixed: `01-00-5E` + a `0` bit.

You take the last 23 bits of the IP address:

IP: `224 . 1 . 2 . 3`
Last three octets: `1 . 2 . 3`
Convert to Hex: `01 : 02 : 03`

Combine `01-00-5E-01-02-03`
```

---

### **Question 28: IPv4 Header (The TTL "Loop" Protection)**

**Scenario:** A packet is sent with a **TTL of 2**. It must pass through **Router-1**, then **Router-2**, to reach **PC-B**.

**What happens when the packet reaches Router-2?** 

A) Router-2 decrements the TTL to 1 and forwards it to PC-B. 
B) Router-2 decrements the TTL to 0 and forwards it to PC-B. 
C) Router-2 decrements the TTL to 0, drops the packet, and sends an ICMP error back to the source. 
D) Router-2 ignores the TTL because the destination is on a directly connected subnet.

##### Answer
```
C.

basically if its not at its destination by the time ttl = 0 then it will get dropped
```

---


### **Question 29: Subnetting (The Overlap Check)** 

You are configuring **Router-1** G0/0 with the IP `10.1.1.193 255.255.255.224`. The router rejects the command, saying it overlaps with G0/1. **Which of the following IPs is likely already configured on G0/1?** 
A) `10.1.1.160 255.255.255.240` 
B) `10.1.1.200 255.255.255.224` 
C) `10.1.1.190 255.255.255.252` 
D) `10.1.1.225 255.255.255.248`

##### Answer:
```
B.

`10.1.1.193 /27` belongs to the subnet `10.1.1.192` to `10.1.1.223`

- A is wrong: Subnet `160/28` ends at `.175`. No overlap.
    
- C is wrong: Subnet `188/30` ends at `.191`. No overlap.
    
- D is wrong: Subnet `224/29` starts at `.224`. No overlap.
    
- B is Correct: Subnet `192/27` includes `.200`. Overlap!
```

---

### **Question 30: Ethernet Error Detection (FCS)** 

When a switch receives a frame, it performs a **Cyclic Redundancy Check (CRC)**. **Based on your notes, which part of the frame is NOT included in the CRC calculation?** 
A) Destination MAC address 
B) Source MAC address 
C) Preamble and Start of Frame Delimiter (SOF) 
D) Data/Payload

##### Answer:
```
C.

The Preamble and SOF are for hardware synchronization (Layer 1). The NIC strips them off before doing the math. The CRC only protects the actual frame (MACs, Type, Data).
```

---

### **Question 31: Legacy Inter-VLAN vs. ROAS** 

In a **Legacy Inter-VLAN** setup (one physical cable per VLAN), a switch port connected to the router must be configured as: 
A) A Trunk port. 
B) An Access port. 
C) A Dynamic Desirable port. 
D) A Layer 3 routed port.

##### Answer:
```
B.

In Legacy Inter-VLAN, you have a separate physical cable for _every_ VLAN. Since each cable only carries one VLAN, the ports on the switch and the router act as simple Access ports.
```

---

### **Question 32: The "Unknown Unicast" Flood** 

Switch S1 has an **empty** MAC address table. PC-A sends a unicast frame to PC-B. **What action does S1 take regarding the MAC table and forwarding?** 
A) S1 records PC-B's MAC address and floods the frame. 
B) S1 records PC-A's MAC address and floods the frame. 
C) S1 drops the frame because the destination is unknown. 
D) S1 sends an ARP request to find PC-B.

##### Answer:
```
B.

switches learn from source.
they look at source mac to build their table.
```

---

### **Question 33: VLSM Calculation** 

You need to divide `192.168.1.0/24` for three subnets:

- LAN A: 100 hosts
- LAN B: 50 hosts
- LAN C: 2 hosts **What is the correct subnet mask for LAN B to minimize wasted addresses?** 
A) `/25` B) `/26` C) `/27`D) `/30`

##### Answer:
```
- For 50 hosts, you need 6 host bits (26=64 addresses, minus 2 for Network/Broadcast = 62 usable).
    
- A 32-bit address minus 6 host bits = a 26-bit mask (/26).
```

---
### **Question 39: The "Native VLAN" Mismatch** 

Switch S1 (Native VLAN 1) is connected via a Trunk to Switch S2 (Native VLAN 99). PC-A is in **VLAN 1** and sends a broadcast frame. 
**What happens when the frame reaches S2?** 

A) S2 receives the untagged frame and assumes it belongs to VLAN 1. 
B) S2 receives the untagged frame and places it into VLAN 99. 
C) S2 drops the frame because the tags don't match. 
D) S2 automatically changes its Native VLAN to 1 to match S1.

##### Answer:
```
B.

s1 sends the packet without any tag because its its native vlan
s2 receives the untagged frame and thinks that "anything without a tag belongs to my native vlan", which is vlan 99.

vlan 1 traffic has leaked into vlan 99.
```

---

### **Question 40: OSPF Path Selection with ECMP** 

A router has two paths to a network.

- Path 1: OSPF (Cost 20)    
- Path 2: OSPF (Cost 20) The command `maximum-paths 1` is configured under the OSPF process. **How will the router handle traffic for this network?** 

A) It will still load-balance because the costs are equal. 
B) It will only install Path 1 (the one learned first) into the routing table. 
C) It will install both but only use Path 2 as a backup. 
D) It will drop all traffic due to a configuration conflict.

##### Answer:
```
B.

Normally, OSPF loves to load-balance (ECMP). However, the `maximum-paths 1` command tells the router: "I don't care if the costs are tied; only put ONE route in the routing table." It will pick the one it learned first and keep the other in the Link State Database (LSDB) as a backup.
```

---

### **Question 41: STP State Transition** 

A port on a switch moves from **Blocking** to **Forwarding**. In RSTP (Rapid STP), what is the name of the temporary state where the port learns MAC addresses but does not yet forward data frames? 

A) Listening 
B) Learning 
C) Discarding 
D) Passive

##### Answer:
```
B.

- Discarding: (Old name: Blocking) No traffic, no MAC learning.

- Learning: No traffic yet, but it starts looking at Source MACs to build the table.

- Forwarding: Full operation.
```

---

### **Question 42: Ethernet II "Data" Minimum** 

An Ethernet II frame has a payload of only **20 bytes**. **According to your notes, what must happen before this frame is sent onto the wire?** 

A) The frame is sent as-is; Ethernet II has no minimum size. 
B) "Padding" bits are added to the data field to reach the minimum length of 46 bytes. 
C) The frame is dropped because it is a "runt" frame. 
D) The frame is converted to an IEEE 802.3 frame automatically.

##### Answer:
```
B.

the minimum data length is 46 bytes. Why? To ensure the frame is long enough for CSMA/CD (Collision Detection) to work. If your data is only 20 bytes, the NIC adds "Padding" (junk bits) to stretch it to 46 bytes.
```
---

### **Question 43: HSRP Virtual MAC** 

You see a frame on your network with the Destination MAC **0000.0c07.ac0a**. **Which HSRP group is this frame destined for?** 

A) Group 1 
B) Group 7 
C) Group 10 
D) Group 100

##### Answer:
```
C. 

You can calculate the HSRP Group ID directly from the MAC address!

- The HSRP Virtual MAC always follows this pattern: `0000.0c07.acXX`.
    
- The XX is the HSRP Group Number in Hexadecimal.
    
- In the question, I gave you `0a`.
    
- In Hex: `a` = 10. So, it belongs to Group 10.
```
