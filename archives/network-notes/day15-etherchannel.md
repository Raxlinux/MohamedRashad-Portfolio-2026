# Day 15: EtherChannel (Port Channel / LAG)

## Summary

In a standard STP environment, redundant links between switches are blocked to prevent loops. **EtherChannel** (also known as Link Aggregation or LAG) allows us to bundle up to eight physical Ethernet links into a single **logical** link. This tricks STP into seeing one big pipe instead of multiple individual paths, providing both increased bandwidth and hardware redundancy.

---

## 1. Purpose & Benefits

Why bother bundling links? If one cable fails, the traffic simply redistributes across the remaining links in the bundle without the spanning tree needing to re-converge.

* **Increased Bandwidth:** Combines the speed of multiple interfaces (e.g., 4 x 1Gbps = 4Gbps logical link).
* **Redundancy:** If a single link in the bundle fails, traffic stays up.
* **Load Balancing:** Traffic is distributed across all active physical links based on a hashing algorithm.
* **STP Efficiency:** STP treats the bundle as a single interface, so no ports are "Blocked" unless the entire bundle is redundant to another path.

---

## 2. The "Golden Rules" of Configuration

For a Port Channel to form, the member interfaces on **both** switches must be identical twins. If there is a mismatch, the channel will either fail to form or put the ports into an `err-disabled` state.

### Required Matching Parameters

* **Speed:** All ports must be the same (e.g., all 1Gbps).
* **Duplex:** All ports must be the same (typically Full-Duplex).
* **Switchport Mode:** All must be Access or all must be Trunk.
* **VLAN Consistency:**
* **Access Mode:** Must belong to the same VLAN.
* **Trunk Mode:** Must have the same **Allowed VLANs** and the same **Native VLAN**.

---

## 3. Load Balancing Inputs

EtherChannel does not use "Round Robin" (sending packet 1 to link A, packet 2 to link B). Instead, it uses a **hash algorithm** based on specific inputs to ensure that a single "flow" stays on the same physical wire to prevent out-of-order packets.

You can configure the switch to use one of the following 6 inputs for that hash:

| Input Type | Description |
| --- | --- |
| **src-mac** | Based on the Source MAC address. |
| **dst-mac** | Based on the Destination MAC address. |
| **src-dst-mac** | Combination of Source and Destination MAC. |
| **src-ip** | Based on the Source IP address. |
| **dst-ip** | Based on the Destination IP address. |
| **src-dst-ip** | Combination of Source and Destination IP (Most common for balanced traffic). |

---

## 4. Negotiation Protocols

There are two main protocols used to negotiate the formation of an EtherChannel, plus a manual "static" method.

| Protocol | Origin | Modes |
| --- | --- | --- |
| **LACP** (802.3ad) | IEEE Standard | **Active** (Initiates) / **Passive** (Waits) |
| **PAgP** | Cisco Proprietary | **Desirable** (Initiates) / **Auto** (Waits) |
| **Static** | N/A | **On** (No negotiation; forces the channel) |

> [!IMPORTANT]
> Never use the `on` mode unless you are certain the other side is also set to `on`. Since there is no negotiation protocol (no "handshake"), you run a high risk of creating a switching loop if one side thinks it's a channel and the other doesn't.

---

## Configuration & Commands

### Load Balancing (Global)

This command is applied once per switch and affects all EtherChannels.

```bash
# Change the load-balancing method
Switch(config)# port-channel load-balance src-dst-ip

```

### Creating the Bundle (Interface)

You should always apply these commands to a **range** of interfaces.

```bash
Switch(config)# interface range g0/1 - 2

# Optional: Define the protocol first (LACP is standard)
Switch(config-if-range)# channel-protocol lacp

# Assign to a group and set the mode
# Modes: active, passive (LACP) | desirable, auto (PAgP) | on (Static)
Switch(config-if-range)# channel-group 1 mode active

```

### Verification

```bash
# The Status of EtherChannel commands
# Look for status "P" (Bundled in port-channel)
Switch# show etherchannel summary

# Check the load-balancing hash currently in use
Switch# show etherchannel load-balance

```

---

## Real-World Insight: The Logical Interface

Once you create a `channel-group 1`, the switch creates a new virtual interface called `interface Port-channel 1`. From that point forward, **any configuration** (like `switchport mode trunk` or `spanning-tree cost`) should be applied to the **Port-channel interface**, not the physical member ports. The switch will automatically "push" the config down to the physical links.
