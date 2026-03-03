# Day 18: Link-State Routing: OSPF

## Summary

**OSPF (Open Shortest Path First)** is a powerful **Link-State Interior Gateway Protocol (IGP)**. Unlike Distance Vector protocols (like RIP) that only know the distance and direction to a destination, OSPF allows every router to maintain a complete "map" or topology of the entire network. This makes it highly efficient, fast to converge, and the industry standard for most enterprise networks.

---

## 1. How OSPF Works

In a Link-State protocol, routers don't just share their routing tables; they share information about their **links** (interfaces) and their states.

* **Connectivity Map:** Every router creates a complete map of the network. To do this, each router advertises information about its local interfaces to its neighbors.
* **Path Selection:** Each router uses this map to independently find the best (shortest) path to every destination.
* **Performance Trade-off:** OSPF reacts **faster** to network changes than RIP, but it requires **more CPU and Memory** because calculating the map is computationally intensive.
* **The Algorithm:** OSPF uses the **Shortest Path First (SPF)** algorithm, developed by the Dutch computer scientist **Edsger Dijkstra**.

### OSPF Versions

| Version | Released | Usage |
| --- | --- | --- |
| **OSPFv1** | 1989 | Historical; no longer in use. |
| **OSPFv2** | 1998 | Standard version for **IPv4** networks. |
| **OSPFv3** | 2008 | Updated version for **IPv6** networks. |

---

## 2. LSAs and the LSDB

OSPF routers communicate using specific data structures to keep everyone in sync:

1. **LSA (Link State Advertisement):** Small packets of data containing information about a router's links and their status.
2. **LSDB (Link State Database):** A "folder" or database where a router stores all the LSAs it has received. All routers within the same area must have an **identical LSDB** to ensure they see the same map.

### The 3-Step Process

A router goes through three main phases to start routing traffic:

1. **Become Neighbors:** Form a relationship with directly connected OSPF routers.
2. **Exchange LSAs:** Share link-state information to build the map.
3. **Calculate Routes:** Run the Dijkstra algorithm to find the best paths and install them in the routing table.

---

## 3. OSPF Areas & Hierarchy

OSPF uses **Areas** to divide the network into manageable sections. In a small network, a single area is fine. However, in large "flat" networks (one single area), performance can suffer.

### Why use Areas?

A single large area causes several issues:

* **SPF Overhead:** The algorithm takes significantly longer to calculate routes.
* **Resource Drain:** High memory and CPU consumption.
* **Instability:** A small change (a "flapping" interface) triggers every single router in the network to reflood LSAs and recalculate the entire map.

### Area Terminology

* **Backbone Area (Area 0):** The central hub. All other areas must connect to it.
* **Internal Router:** A router where all interfaces belong to the same area.
* **Area Border Router (ABR):** A router connected to multiple areas (e.g., Area 0 and Area 1). *Note: It is recommended to limit an ABR to a maximum of 2 areas.*
* **Backbone Router:** Any router with at least one interface in Area 0.
* **Intra-area Route:** A destination within the same area.
* **Inter-area Route:** A destination in a different OSPF area.

---

## 4. Design Rules & Guidelines

To keep OSPF stable, you must follow these rules:

1. **Contiguous Backbone:** Area 0 must be physically continuous.
2. **ABR Connection:** All non-backbone areas must have at least one ABR connected directly to Area 0.
3. **Subnet Consistency:** OSPF interfaces on the same subnet must reside in the same area.

---

## Configuration & Commands

### OSPF Configuration

```bash
# Start the OSPF process (Process ID is local only; doesn't have to match neighbors)
Router(config)# router ospf 10

# Manually set the Router ID (Recommended for stability)
Router(config-router)# router-id 1.1.1.1

# Advertise a specific network into an area (using Wildcard Mask)
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0

# Stop sending OSPF Hello packets to user-facing interfaces
Router(config-router)# passive-interface g0/0

# Inject a default route (0.0.0.0/0) into OSPF updates
Router(config-router)# default-information originate

# Set the maximum number of equal-cost paths for load balancing
Router(config-router)# maximum-paths 4

# Change the Administrative Distance (Default OSPF is 110)
Router(config-router)# distance 110

```

### Management & Troubleshooting

```bash
# View the OSPF neighbor table (Crucial for verifying connectivity)
Router# show ip ospf neighbor

# View the OSPF Database (LSAs)
Router# show ip ospf database

# View OSPF-specific routes in the routing table
Router# show ip route ospf

# Resets the OSPF process (Use with caution: drops all adjacencies)
Router# clear ip ospf process

```

---

> [!Warning]
> Be careful with the `clear ip ospf process` command in a production environment. It will cause a temporary network outage while the routers renegotiate their neighbor relationships and rebuild their maps.
