# Day 19: OSPF Cost, Neighbors, and Adjacency States

## 1. OSPF Metric: Cost

In OSPF, the metric used to determine the best path is called **Cost**. By default, this is an automatically calculated value based on the bandwidth of the interface.

### The Calculation Formula

The router calculates cost by dividing a **Reference Bandwidth** by the **Interface Bandwidth**:

$$Cost = \frac{\text{Reference Bandwidth}}{\text{Interface Bandwidth}}$$

* **Default Reference Bandwidth:** 100 Mbps ($10^8$ bps).
* **The "Cost of 1" Limitation:** Because OSPF cost must be a whole number and cannot be less than 1, any interface with a speed of 100 Mbps or higher results in a cost of **1**.
* **FastEthernet (100 Mbps):** $100 / 100 = 1$
* **GigabitEthernet (1000 Mbps):** $100 / 1000 = 0.1 \rightarrow \mathbf{1}$
* **10GigabitEthernet (10,000 Mbps):** $100 / 10000 = 0.01 \rightarrow \mathbf{1}$

> [!Note]
> In modern networks, you should change the reference bandwidth to a higher value (like 100,000) so OSPF can differentiate between a 1G and a 10G link.

### Loopback Interfaces

By default, a **Loopback interface** is assigned a cost of **1**. Since these are virtual interfaces and do not represent physical hardware, they are considered "perfect" links.

---

## 2. OSPF Neighbors

Establishing neighbor relationships is the most critical part of OSPF operation. Routers must "see" each other before they can share any routing information.

* **Hello Messages:** Once OSPF is activated on an interface, the router begins sending **Hello packets** every **10 seconds**.
* **Multicast Address:** Hellos are sent to the OSPF multicast address **224.0.0.5** (All OSPF Routers).
* **Encapsulation:** OSPF does not use TCP or UDP. It is encapsulated directly in the IP header with **Protocol Number 89**.

---

## 3. OSPF Neighbor States

To reach a "Full" adjacency where routing information is shared, two routers must progress through seven distinct states:

1. **Down:** The initial state. OSPF is enabled on the interface, but no Hellos have been received from neighbors yet. The router sends out its own Hello.
2. **Init:** The router receives a Hello from a neighbor, but its own Router ID (RID) is **not** listed in that Hello packet. It adds the sender’s RID to its table.
3. **2-Way:** The router receives a Hello that *does* include its own RID. Bidirectional communication is confirmed. On multi-access networks (like Ethernet), the **DR (Designated Router)** and **BDR** election occurs here.
4. **Exstart:** Routers prepare to share database information. They determine a **Master** and **Slave** relationship based on the highest RID; the Master initiates the exchange.
5. **Exchange:** Routers exchange **DBD (Database Description)** packets. These are essentially "summaries" of their Link-State Databases. Routers compare these summaries against what they already know.
6. **Loading:** If a router realizes it is missing information (based on the DBDs), it sends a **Link State Request (LSR)**. The neighbor responds with the actual data in a **Link State Update (LSU)**. The receiving router then sends a **Link State Acknowledgment (LSAck)**.
7. **Full:** The routers are fully adjacent. Their LSDBs are now **identical**.

### Maintenance and Timers

Once in the **Full** state, routers continue to send Hellos every 10 seconds to maintain the relationship.

* **Dead Timer:** Every time a Hello is received, a 40-second "Dead Timer" resets.
* **Timeout:** If the timer reaches **0** without receiving a Hello, the neighbor is considered "dead" and is removed from the OSPF routing table.

---

## 4. Configuration & Verification Commands

### Adjusting Cost and Bandwidth

```bash
# Change the reference bandwidth (applied to the entire OSPF process)
# Recommended to set to 100000 (for 100G) to support modern speeds
Router(config-router)# auto-cost reference-bandwidth 100000

# Manually set the OSPF cost on a specific interface (overrides auto-calculation)
Router(config-if)# ip ospf cost 10

# Change the logical bandwidth of an interface (affects cost calculation)
Router(config-if)# bandwidth 10000

```

### Interface and Process Configuration

```bash
# Enable OSPF directly on an interface (Modern method)
Router(config-if)# ip ospf 1 area 0

# Set all interfaces to passive by default, then manually enable the ones needed
Router(config-router)# passive-interface default
Router(config-router)# no passive-interface g0/0

```

### Verification

```bash
# Summary view of OSPF-enabled interfaces, their area, and cost
Router# show ip ospf interface brief

# Filter the running configuration to see only OSPF-related settings
Router# show running-config | section ospf

```

---

> [!Tip]
> When troubleshooting neighbor relationships, always ensure that the **Hello/Dead timers**, **Area ID**, and **Subnet Mask** match on both sides, or the adjacency will never reach the "2-Way" state.
