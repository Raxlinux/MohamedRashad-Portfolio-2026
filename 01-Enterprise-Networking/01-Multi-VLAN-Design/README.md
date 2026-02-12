# Enterprise Network Segmentation & Security Hardening

### Project Overview: Transitioning from a "Flat" Network to a Secure VLAN Architecture

## 📌 Executive Summary

In a default "flat" network, all devices occupy the same broadcast domain, creating significant performance bottlenecks and a massive attack surface. Leveraging my background in industrial systems (Siemens), I designed and implemented a segmented network for a simulated enterprise environment.

This project demonstrates **VLSM (Variable Length Subnet Masking)** for address optimization and **Router-on-a-Stick (ROAS)** for secure inter-departmental communication, while implementing security hardening to prevent common Layer 2 attacks.

## 🏗 Phase 1: Network Architecture & Design

### 1. Address Optimization (VLSM)

I moved away from wasteful classful addressing by taking a 172.16.10.0/23 block and carving it specifically for department requirements. I prioritized the largest subnets first to ensure contiguous allocation:

| Department | Requirement | Subnet Assigned | CIDR |
|-------------|-------------|-------------|-------------|
| **Engineering** | 120 Hosts | 172.16.10.0 | /25 |
| **Sales** | 50 Hosts | 172.16.10.128 | /26 |
| **Guests** | 30 Hosts | 172.16.10.192 | /27 |
| **HR** | 25 Hosts | 172.16.10.224 | /27 |

### 2. Logical Segmentation (VLANs)

To "kill the flat network," I partitioned the switch into 5 distinct VLANs. This ensures Layer 2 isolation; if a Guest Wi-Fi device is compromised, the attacker cannot move laterally into the HR or Engineering segments at the data link layer.

## 🛠 Phase 2: Implementation & Security Hardening

### 1. Inter-VLAN Routing (ROAS)

I utilized **802.1Q encapsulation** on a single physical link to a Core Router. By creating logical sub-interfaces (Gig0/0/0.X), I enabled controlled communication between departments without the expense of multiple physical router ports.

### 2. Security Configuration (CLI Highlights)

I implemented several "Pro-Level" security measures to defend against **VLAN Hopping** and double-tagging attacks:

**- Native VLAN Hardening:** Moved the native VLAN from the default (VLAN 1) to an unused VLAN 99.

    switchport trunk native vlan 99

**- Static Port Definitions:** Manually set ports to access or trunk to disable DTP (Dynamic Trunking Protocol), a common entry point for attackers.

    switchport mode access

**- Traffic Control:** Configured sub-interfaces to ensure only tagged frames are processed by the gateway.

## ✅ Phase 3: Verification & "Proof of Work"\

**Isolation Test:** PC 1 (Engineering) was unable to ping PC 3 (Sales) initially, proving that logical walls were functional before routing was applied.

**Connectivity Test:** Verified inter-VLAN routing via show ip route to ensure all VLSM subnets were identified.

**Trunk Verification:** Confirmed 802.1Q tags were preserved using show interfaces trunk.

[Screenshot: topology-diagram.png](../01-Multi-VLAN-Design/assets/topology-diagram.png)
[Screenshot: ping-core-router.png](../01-Multi-VLAN-Design/assets/ip-route-core-router.png)
[Screenshot: trace-route.png](../01-Multi-VLAN-Design/assets/trace-route.png)

## 📝 Engineering Reflections

**> Industrial Context:** This methodology mirrors the logical isolation required in Siemens Control and Relay Panels (CRP). Just as high-voltage monitoring traffic must be isolated from management traffic, enterprise data requires strict segmentation to ensure system uptime and security.

**> The Zero-Trust Mindset:** This lab was built on the principle that "just because you are on the same switch doesn't mean you are trusted." VLANs are the first step in a Zero-Trust architecture.

📁 Project Artifacts

[- Handwritten Subnetting Calculations](../01-Multi-VLAN-Design/assets/IPv4-subnetting/)
[- Cisco Packet Tracer Lab File (.pkt)](../01-Multi-VLAN-Design/IPv4_VLSM_Network_Architecture.pkt)