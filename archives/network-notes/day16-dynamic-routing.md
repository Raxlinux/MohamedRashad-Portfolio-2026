# Day 16: Dynamic Routing Overview

## Summary

While static routing requires manual entry for every network path, **Dynamic Routing** allows routers to communicate with one another to share information about remote networks. Routers automatically learn paths, maintain routing tables, and adapt to topology changes (like a failed link) without human intervention.

---

## 1. Routing Protocol Classification

Dynamic routing protocols are categorized based on where they operate and the underlying logic they use to determine the "best path."

### Interior Gateway Protocol (IGP)

Used for routing **within** an Autonomous System (AS)—typically a single organization’s internal network.

* **Distance Vector:** Determines the best path based on distance (hops) or a vector (direction).
* **RIP (Routing Information Protocol)**
* **EIGRP (Enhanced Interior Gateway Routing Protocol)**

* **Link State:** Each router creates a complete map of the network topology.
* **OSPF (Open Shortest Path First)**
* **IS-IS (Intermediate System to Intermediate System)**

### Exterior Gateway Protocol (EGP)

Used for routing **between** different Autonomous Systems. The Internet is essentially built on EGP.

* **Path Vector:** Includes the path information (AS-Path) to reach a destination to prevent loops.
* **BGP (Border Gateway Protocol)**

---

## 2. Metrics & Protocol Logic

A **Metric** is the value assigned by a routing protocol to a specific path. If multiple paths exist to the same destination, the router chooses the path with the **lowest** metric.

| Protocol | Type | Metric Basis |
| --- | --- | --- |
| **RIP** | Distance Vector | **Hop Count:** Each router passed is 1 hop. Max 15 hops. |
| **EIGRP** | Adv. Distance Vector | **Bandwidth & Delay:** Uses a composite formula (Bandwidth by default). |
| **OSPF** | Link State | **Cost:** Calculated based on bandwidth (100Mbps / Bandwidth). |
| **IS-IS** | Link State | **Cost:** Sum of link costs. All links default to a cost of **10** (not auto-calculated). |
| **BGP** | Path Vector | **Attributes:** Uses a complex list of attributes (Weight, Local Pref, AS-Path). |

### ECMP (Equal Cost Multi-Path)

If a router learns two or more paths to the same destination with the **exact same metric**, it doesn't just pick one. It adds both to the routing table and uses **ECMP** to load balance traffic across all available paths, maximizing bandwidth efficiency.

---

## 3. Administrative Distance (AD)

If a router learns about a network from two *different* protocols (e.g., OSPF and RIP), it uses **Administrative Distance** to decide which one to trust. AD represents the "trustworthiness" of the source. **Lower is better.**

| Route Source / Protocol | Administrative Distance (AD) |
| --- | --- |
| **Directly Connected** | 0 |
| **Static Route** | 1 |
| **External BGP (eBGP)** | 20 |
| **EIGRP (Internal)** | 90 |
| **IGRP** | 100 |
| **OSPF** | 110 |
| **IS-IS** | 115 |
| **RIP** | 120 |
| **EIGRP (External)** | 170 |
| **Internal BGP (iBGP)** | 200 |
| **Unusable Route** | 255 |

---

## 4. Floating Static Routes

By default, a static route has an AD of **1**, making it more "trustworthy" than any dynamic protocol. However, we can manually assign a higher AD to a static route so it only appears in the routing table if the dynamic protocol fails.

* **Concept:** A static route with a manual AD higher than the dynamic protocol (e.g., AD of 130 to backup RIP).
* **Function:** It "floats" above the routing table and only drops into the table if the primary dynamic route is lost.

---

## Verification Commands (Cisco)

```bash
# To view the routing table (look for R, O, D, B codes)
Router# show ip route

# To view specific details about routing protocols running
Router# show ip protocols

# To check the AD and Metric of a specific route
Router# show ip route 192.168.1.0

```
