# Day 14: Rapid Spanning Tree Protocol (RSTP)

## Summary

Standard STP (802.1D) is slow, often taking 30–50 seconds to converge. In modern networking, that's an eternity. **Rapid Spanning Tree Protocol (RSTP / 802.1w)** was designed to reduce convergence time to sub-second levels by introducing new port roles, link types, and a proactive "handshake" mechanism between switches.

---

## 1. Evolution of Spanning Tree Versions

The networking world is split between official IEEE standards and Cisco's enhanced proprietary versions.

| Feature | IEEE Standard | Cisco Version | Description |
| --- | --- | --- | --- |
| **Classic STP** | 802.1D | **PVST+** | One STP instance per VLAN. Slow convergence. |
| **Rapid STP** | 802.1w | **Rapid PVST+** | Fast convergence. Cisco's version still runs per-VLAN. |
| **Multiple STP** | 802.1s | **MST** | Maps multiple VLANs into a single STP instance to save CPU. |

> [!NOTE]
> On Cisco switches, the default is often PVST+. To get the benefits of RSTP, you must explicitly change the mode to `rapid-pvst`.

---

## 2. RSTP Port Roles & Functions

RSTP keeps the **Root Port** and **Designated Port** roles but replaces the generic "Blocking" state with two specific roles that define *why* a port is waiting.

### Alternate Port Role

The "Backup Root Port." This port has an alternative path to the Root Bridge but is currently discarded. If the primary Root Port fails, the Alternate port can transition to forwarding almost instantly.

### Backup Port Role

The "Backup Designated Port." This occurs when a switch has two links to the same collision domain (usually via a hub). It serves as a redundant path for the segment.

---

## 3. Link Types & Convergence

RSTP convergence speed is heavily dependent on the **Link Type**. The switch automatically detects these based on duplex settings, but they can be hardcoded.

* **Edge Port (PortFast):** Connected to end-devices. Transitions to forwarding immediately because it assumes no loops can form.
* **Point-to-Point (P2P):** A direct link between two switches (running in Full-Duplex). These use a "Proposal/Agreement" handshake to transition states in milliseconds.
* **Shared:** Ports operating in Half-Duplex (connected to a Hub). These fall back to slow 802.1D timers because they cannot safely use the handshake mechanism.

---

## 4. Key Operational Change: BPDU Handling

In classic 802.1D, only the Root Bridge generated BPDUs, which were then "relayed" by other switches.

In **RSTP**, every switch sends its own BPDUs containing its current information every "Hello" interval (default 2s). This acts as a keep-alive; if a switch misses three BPDUs in a row (6 seconds), it immediately assumes the neighbor is gone and starts re-calculating.

---

## Configuration & Commands

### Global Configuration

Switch to the faster protocol and define edge ports.

```bash
# Change the STP mode to RSTP (Cisco's version)
Switch(config)# spanning-tree mode rapid-pvst

# Global PortFast (Sets all access ports as Edge Ports)
Switch(config)# spanning-tree portfast default

```

### Interface Configuration

Manually overriding the link-type detection.

```bash
# Force a port to be an Edge Port
Switch(config-if)# spanning-tree portfast

# Force a port to behave as a Point-to-Point link (Handshake enabled)
Switch(config-if)# spanning-tree link-type point-to-point

# Force a port to behave as a Shared link (Timers enabled)
Switch(config-if)# spanning-tree link-type shared

```

### Verification

```bash
# Check the STP mode and see Port Roles (Altn, Back, Desg, Root)
Switch# show spanning-tree

```

---

## Real-World Insight: The Duplex Trap

RSTP's "Rapid" feature only works on **Point-to-Point** links. Because the switch detects this based on duplex, if a cable is bad or a port is hardcoded to Half-Duplex, the switch will downgrade the link to **Shared**. This reverts the port to the old 30-second delay. Always ensure your switch-to-switch trunks are healthy and running in Full-Duplex to keep the "Rapid" in RSTP.
