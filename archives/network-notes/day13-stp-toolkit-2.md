# Day 13: STP Hardening & Guard Features

## Summary

While STP prevents loops, it is inherently "trusting." If a rogue switch is plugged into an access port, it could hijack the Root Bridge position or create a loop. Today's focus is on **STP Features**—the toolkit used to lock down the topology, protect "Edge" ports, and ensure that a hardware failure doesn't turn a redundant link into a network-killing loop.

---

## 1. Protecting the Edge: BPDU Guard & Filter

These features are almost always paired with **PortFast** to ensure that ports meant for end-devices (PCs, printers) stay that way.

### BPDU Guard

If a port with BPDU Guard receives a BPDU (meaning someone plugged in a switch), the port is immediately shut down and placed into an **err-disabled** state. It’s the "shoot first, ask questions later" approach to security.

### BPDU Filter

This is the more "silent" version. It prevents the port from sending or receiving BPDUs.

* **Global:** If enabled globally, it waits for a BPDU. If one is received, the port loses its PortFast status and becomes a normal STP port.
* **Interface:** It effectively disables STP on that port. **Caution:** This can easily cause a loop if you aren't careful.

---

## 2. Protecting the Path: Root Guard & Loop Guard

These features protect the core of your network topology.

### Root Guard

Prevents a downstream switch from becoming the Root Bridge. If a port receives a superior BPDU (one with a lower Bridge ID), Root Guard puts the port into a **root-inconsistent** state. Traffic stops until the offending BPDUs cease.

### Loop Guard

STP relies on receiving BPDUs to keep a port in a "Blocking" state. If a link becomes unidirectional (can send but not receive), the switch might stop receiving BPDUs and mistakenly transition the port to "Forwarding," creating a loop. Loop Guard detects this lack of BPDUs and moves the port to a **loop-inconsistent** state.

> [!IMPORTANT]
> **The Mutual Exclusion Rule:** Loop Guard and Root Guard are mutually exclusive. You cannot run both on the same port simultaneously. If you attempt to configure one after the other, the most recent command will override the previous one.

---

## 3. ErrDisable Recovery

When BPDU Guard kills a port, it stays dead until an admin manually does a `shutdown` / `no shutdown`. **ErrDisable Recovery** allows the switch to automatically attempt to bring the port back online after a specified timeout period.

---

## Configuration & Commands

### Interface Configuration Mode

Used for surgical precision on specific ports.

```bash
# BPDU Protection
Switch(config-if)# spanning-tree bpduguard enable
Switch(config-if)# spanning-tree bpduguard disable

# BPDU Filtering
Switch(config-if)# spanning-tree bpdufilter enable
Switch(config-if)# spanning-tree bpdufilter disable

# Port Guards (Mutually Exclusive)
Switch(config-if)# spanning-tree guard root
Switch(config-if)# spanning-tree guard loop
Switch(config-if)# spanning-tree guard none

```

### Global Configuration Mode

Used to apply "blanket" policies across the entire switch.

```bash
# Apply to all PortFast-enabled ports
Switch(config)# spanning-tree portfast bpduguard default
Switch(config)# spanning-tree portfast bpdufilter default

# Apply guards globally
Switch(config)# spanning-tree rootguard default
Switch(config)# spanning-tree loopguard default

# Automatic recovery from BPDU Guard shutdown
Switch(config)# errdisable recovery cause bpduguard
Switch(config)# errdisable recovery interval 300  # Seconds until retry

```

---

## Feature Comparison Table

| Feature | Primary Goal | Recommended Location | Action on Violation |
| --- | --- | --- | --- |
| **BPDU Guard** | Stop unauthorized switches | Access/Edge Ports | Err-Disable |
| **BPDU Filter** | Stop STP traffic | Lab environments | Drops BPDUs |
| **Root Guard** | Enforce Root Bridge location | Distribution Ports (Downlink) | Root-Inconsistent |
| **Loop Guard** | Prevent unidirectional loops | Uplink/Trunk Ports | Loop-Inconsistent |

---

## Real-World Insight: The "Mesh" Perspective

In the Siemens panels i tested for client witnessing, the "Grid" or "Mesh" connectivity provides immense reliability, but it increases the number of redundant paths exponentially. Without features like **Loop Guard**, a single fiber strand failure in a high-EMI (Electromagnetic Interference) factory environment could cause a silent STP failure. Using these guards ensures the network fails **safely** (by blocking) rather than failing **destructively** (by looping).
