# Day 17: Dynamic Routing: RIP & EIGRP

## Summary

Dynamic routing protocols allow routers to automatically learn about remote networks and adapt to topology changes. **RIP** is a legacy protocol suited for small, simple networks, while **EIGRP** is a powerful, "advanced" protocol designed for speed and scalability within enterprise environments.

---

## 1. RIP (Routing Information Protocol)

RIP is a **Distance Vector IGP** that uses **Hop Count** as its metric (the number of routers a packet must pass through). It is easy to configure but limited by its slow convergence and scale.

* **Metric Limit:** Maximum hop count is **15**. A network at 16 hops is considered unreachable (Infinity).
* **Update Timer:** By default, RIP-enabled routers broadcast/multicast their entire routing table every **30 seconds**.
*

**Message Types:**

* **Request:** Sent by a router to ask neighbors for updates.
* **Response:** Sent by a router containing the routing table (the update).

### RIP Versions Comparison

| Feature | RIPv1 | RIPv2 | RIPng |
| --- | --- | --- | --- |
| **Address Support** | IPv4 (Classful) | IPv4 (Classless) | IPv6 |
| **VLSM/CIDR** | Not Supported | **Supported** | Supported |
| **Subnet Mask Info** | Not included in updates | **Included** in updates | Included |
| **Method** | Broadcast (255.255.255.255) | Multicast (**224.0.0.9**) | Multicast (FF02::9) |

---

## 2. EIGRP (Enhanced Interior Gateway Routing Protocol)

EIGRP is a Cisco proprietary protocol (now partially published for other vendors) categorized as an **"Advanced" or "Hybrid" Distance Vector** protocol because it combines the best features of Distance Vector and Link-State protocols.

* **Fast Convergence:** Reacts much faster than RIP to network changes.
* **Multicast Address:** Uses **224.0.0.10** for updates.
* **Load Balancing:** The only IGP capable of **Unequal-cost load-balancing** (though it uses Equal-Cost Multi-Path (ECMP) by default).

### EIGRP Terminology

| Term | Definition |
| --- | --- |
| **Feasible Distance (FD)** | The best (lowest) metric to reach a destination. |
| **Reported Distance (RD)** | The metric to a destination as advertised by a neighbor. |
| **Successor** | The primary route/neighbor used to reach a destination (installed in the routing table). |
| **Feasible Successor** | A backup route that meets the feasibility condition (ready to take over if the Successor fails). |

---

## 3. Router ID Priority Order

When a routing process starts (like EIGRP or OSPF), the router needs a unique name (the Router ID). It selects this ID based on the following hierarchy:

1. **Manual Configuration:** Manually assigned ID (e.g., `router-id 1.1.1.1`).
2. **Highest Loopback IP:** The highest IP address configured on any loopback interface.
3. **Highest Physical IP:** The highest IP address on any active physical interface.

---

## Configuration & Commands

### RIP Configuration

```bash
Router(config)# router rip

# Switches to Version 2 (V1 is outdated/classful)
Router(config-router)# version 2

# Prevents the router from summarizing to classful boundaries
Router(config-router)# no auto-summary

# Advertise the network (Classful network)
Router(config-router)# network 192.168.10.0

# Stop sending RIP updates out of a specific interface (e.g., toward users)
Router(config-router)# passive-interface g0/0

# Sends a default route (0.0.0.0/0) to all RIP neighbors
Router(config-router)# default-information originate

```

### EIGRP Configuration

```bash
# '1' is the Autonomous System (AS) number; must match on all neighbors
Router(config)# router eigrp 1

Router(config-router)# no auto-summary

# Advertise the network
Router(config-router)# network 10.0.0.0

# Set the max number of parallel paths for load balancing
Router(config-router)# maximum-paths 4

# Change the Administrative Distance (Default EIGRP is 90)
Router(config-router)# distance 90

```

### Verification & Troubleshooting

```bash
# View general protocol info (timers, networks, filters)
Router# show ip protocols

# View the EIGRP topology table (Successors and Feasible Successors)
Router# show ip eigrp topology

# View the routing table
Router# show ip route

```

---

> [!Important]
> Always remember that **Administrative Distance (AD)** determines which protocol is trusted more. If a router learns about the same network via RIP (AD 120) and EIGRP (AD 90), it will always choose the EIGRP route.
