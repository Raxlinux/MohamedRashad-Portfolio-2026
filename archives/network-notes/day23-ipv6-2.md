# Day 23: IPv6 Address Types & EUI-64

In IPv4, we relied heavily on DHCP or manual entry for every single device. IPv6 introduces a more "plug-and-play" approach. By understanding the different address types and the **EUI-64** process, we can see how devices can intelligently generate their own identities without human intervention.

---

## 1. Modified EUI-64 (Extended Unique Identifier)

**EUI-64** is a method used to automatically generate the 64-bit **Interface Identifier** (the second half of an IPv6 address) using a device's 48-bit MAC address. Because MAC addresses are globally unique, this ensures the IPv6 address is also unique.

### The Conversion Process ("Flip and Stuff")

To convert a MAC address into an EUI-64 interface ID, follow these three steps:

1. **Split:** Take the 48-bit MAC address and split it exactly in the middle.
2. **Stuff:** Insert the hex digits **`FFFE`** between the two halves.
3. **Flip:** Convert the first byte to binary and flip the **7th bit** (the Universal/Local bit).

#### Example Conversion

* **Original MAC:** `0011:2233:4455`
* **Step 1 & 2 (Split & Stuff):** `0011:22` + `FFFE` + `33:4455` $\rightarrow$ `0011:22FF:FE33:4455`
* **Step 3 (Flip the 7th bit):**
* The first byte is `00` (Hex) $\rightarrow$ `00000000` (Binary).
* Flip the 7th bit $\rightarrow$ `00000010` (Binary).
* New Hex value $\rightarrow$ `02`.

* **Resulting EUI-64 ID:** `0211:22FF:FE33:4455`

---

## 2. Configuration: Auto-Generating Addresses

Instead of typing out the full 128-bit address, you can tell the router to use the EUI-64 process to complete the address based on its own hardware.

```bash
Router(config)# interface gigabitEthernet 0/0

# Provide the 64-bit prefix and let the router do the rest
Router(config-if)# ipv6 address 2001:db8::/64 eui-64

```

---

## 3. Types of IPv6 Addresses

IPv6 categorizes traffic differently than IPv4. Notably, **Broadcast is gone**; it has been replaced by more efficient Multicast groups.

| Address Type | Purpose | Range/Address |
| --- | --- | --- |
| **Global Unicast** | Publicly routable on the internet (Like IPv4 Public IPs). | `2000::/3` |
| **Link-Local** | Used for communication on the local wire only; non-routable. | `fe80::/10` |
| **Unique Local** | Private addresses used inside an organization (Like 192.168.x.x). | `fc00::/7` |
| **Multicast** | One-to-many communication. | `ff00::/8` |
| **Anycast** | One-to-nearest communication (shared by multiple devices). | *Uses GUA range* |
| **Unspecified** | Used when a device doesn't have an IP yet. | `::` |
| **Loopback** | Used for testing the local software stack. | `::1` |

---

## 4. IPv6 Multicast Scopes

Multicast in IPv6 is highly organized using "Scopes." The scope defines how far a multicast packet is allowed to travel before a router drops it. This is determined by the **4th hex digit** of the address.

* **FF01 (Interface-local):** The packet never leaves the local node (used for internal testing).
* **FF02 (Link-local):** The packet stays on the local physical wire/VLAN. Routers will **not** forward this.
* **FF05 (Site-local):** The packet can travel throughout the entire local site or office but not beyond.
* **FF08 (Organization-local):** Larger than a site; covers all locations owned by one company.
* **FF0E (Global):** Can travel across the entire public internet.

---

> [!Note]
> Link-Local addresses (`fe80::`) are mandatory. Even if you don't configure a Global IP, your router will generate a Link-Local address the moment you run `ipv6 enable` or assign a Global address. This allows the router to start "talking" to neighbors immediately.
