# Day 27: Layer 2 Discovery Protocols (CDP & LLDP)

In a complex network environment, knowing "who is connected to what" is half the battle. **Layer 2 Discovery Protocols** act as the network's internal directory, allowing devices to automatically discover and share information with their directly connected neighbors.

While incredibly useful for troubleshooting and mapping, these protocols are a "double-edged sword" because they share sensitive details—like software versions and IP addresses—with any connected device. Consequently, in high-security production environments, they are often disabled or strictly controlled.

---

## 1. Core Concepts

Layer 2 discovery protocols function by sending periodic advertisements out of active interfaces. This information includes:

* **Device Identifiers:** Hostname and IP address.
* **Hardware Info:** Device type/model and capabilities (Router, Switch, etc.).
* **Connection Info:** Local and remote port IDs.
* **Software Info:** IOS version and compatibility.

> [!WARNING]  
> **Security Risk:** Because discovery protocols broadcast internal topology details in plain text, an attacker who plugs into a port can easily map your network. Standard practice is to disable these on "untrusted" or user-facing ports.

---

## 2. Cisco Discovery Protocol (CDP)

CDP is a **Cisco-proprietary** protocol used to share information between Cisco-branded devices. It is the veteran of discovery protocols and is usually enabled by default on most Cisco hardware.

### Key Characteristics

* **Propagation:** Messages are sent to the multicast address `01:00:0c:cc:cc:cc`.
* **Non-Forwarding:** When a device receives a CDP message, it processes the data and discards it. It does **not** forward the message to other devices.
* **Default Timers:** * **Update Timer:** 60 seconds (how often it "says hello").
  * **Hold Time:** 180 seconds (how long it waits before dropping a neighbor from the table).
* **CDPv2:** The modern version (enabled by default) includes enhanced features like **Native VLAN mismatch detection**.

### CDP Configuration & Show Commands

| Task | Command |
| :--- | :--- |
| **Global Enable/Disable** | `[no] cdp run` |
| **Interface Enable/Disable** | `[no] cdp enable` |
| **Change Hello Timer** | `cdp timer <seconds>` |
| **Change Hold Time** | `cdp holdtime <seconds>` |
| **View Neighbor Summary** | `show cdp neighbor` |
| **View Detailed Neighbor Info** | `show cdp neighbor detail` or `show cdp entry <name>` |
| **Verify Traffic/Status** | `show cdp` or `show cdp traffic` |

---

## 3. Link Layer Discovery Protocol (LLDP)

LLDP is the **industry-standard** (IEEE 802.1AB) alternative to CDP. It provides the same basic functionality but allows for vendor interoperability (e.g., a Cisco switch talking to a Juniper router).

### LLDP Key Characteristics

* **Status:** Usually disabled by default; must be enabled manually.
* **Propagation:** Sent to the multicast address `01:80:c2:00:00:0e`.
* **Default Timers:** * **Update Timer:** 30 seconds.
  * **Hold Time:** 120 seconds.
* **Initialization Delay:** LLDP features a mandatory "Reinitialization Delay" (default **2 seconds**). The device waits for this timer to expire after LLDP is enabled before it starts sending/receiving updates.
* **Granular Control:** Unlike CDP, LLDP allows you to toggle transmission (`transmit`) and reception (`receive`) independently on an interface.

### Configuration & Show Commands

| Task | Command |
| :--- | :--- |
| **Global Enable** | `lldp run` |
| **Enable TX on Interface** | `lldp transmit` |
| **Enable RX on Interface** | `lldp receive` |
| **Change Hello Timer** | `lldp timer <seconds>` |
| **Change Hold Time** | `lldp holdtime <seconds>` |
| **Change Init Delay** | `lldp init <seconds>` |
| **View Neighbor Summary** | `show lldp neighbors` |
| **View Detailed Info** | `show lldp neighbors detail` or `show lldp entry <name>` |

---

## 4. Comparison Summary

| Feature | CDP | LLDP |
| :--- | :--- | :--- |
| **Ownership** | Cisco Proprietary | Industry Standard (IEEE 802.1AB) |
| **Default State** | Enabled | Disabled |
| **Update Timer** | 60 seconds | 30 seconds |
| **Hold Time** | 180 seconds | 120 seconds |
| **Multicast MAC** | `0100.0ccc.cccc` | `0180.c200.000e` |
| **Interface Control** | On/Off | Independent TX and RX |

---

### Practical Lab Tip

If you are working in a mixed-vendor environment, you can run both CDP and LLDP simultaneously on a Cisco device. This ensures you can "see" both your Cisco neighbors and your third-party appliances without choosing sides.
