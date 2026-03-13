# Day 22: The Evolution of IPv6 Addressing

While IPv4 served us well for decades, its limitations eventually caught up with the scale of the modern internet. IPv6 isn't just "IPv4 with more numbers"; it is a complete redesign of how we handle logical addressing, built for a world where even your toaster might need an IP.

---

## 1. The History of the "Version"

You might wonder why we jumped straight from Version 4 to Version 6.

* **The "Lost" Version:** In the late 1970s, the **Internet Stream Protocol (ST)** was developed for experimental transmission of voice and video. It was internally referred to as **IPv5**.
* **The Decision:** Since IPv5 was already used in research but never released for public use, the IETF named the successor to IPv4 "**IPv6**" to avoid any technical confusion or overlapping protocols.

---

## 2. The Language of IPv6: Hexadecimal

To handle the massive scale of IPv6, we moved away from the 10-digit decimal system and embraced **Hexadecimal (Base 16)**. This allows us to represent large binary strings in a much shorter format.

| System | Base | Prefix | Digits Used |
| --- | --- | --- | --- |
| **Binary** | Base 2 | `0b` | 0, 1 |
| **Decimal** | Base 10 | `0d` | 0 - 9 |
| **Hexadecimal** | Base 16 | `0x` | 0 - 9 and A - F |

### Conversion Logic

In IPv6, every **4 bits** (a "nibble") of a binary address is represented by **1 hexadecimal character**.

* **Binary to Hex:** `1111` in binary becomes `F` in hex.
* **Hex to Binary:** `A` in hex becomes `1010` in binary.

---

## 3. Why the Change?

The primary driver for IPv6 is **address exhaustion**.

* **The IPv4 Limit:** IPv4 uses a 32-bit address, providing roughly **$2^{32}$** addresses (approx. 4.3 billion). In a world with billions of smartphones and IoT devices, this is nowhere near enough.
* **The IPv6 Solution:** IPv6 uses a 128-bit address, providing **$2^{128}$** addresses.

> [!Note]
> To put $2^{128}$ in perspective, it is roughly 340 undecillion addresses—enough to give every grain of sand on Earth its own dedicated IP range.

### Governance and Distribution

IP space is not a free-for-all. It is strictly managed by **IANA** (Internet Assigned Numbers Authority), which distributes blocks to five **Regional Internet Registries (RIRs)**:

1. **AFRINIC:** Africa
2. **APNIC:** Asia-Pacific
3. **ARIN:** Canada, USA, and some Caribbean islands
4. **LACNIC:** Latin America and the Caribbean
5. **RIPE NCC:** Europe, the Middle East, and Central Asia

---

## 4. IPv6 Shortening Rules

A full IPv6 address is long and cumbersome (e.g., `2001:0db8:0000:0000:0000:ff00:0042:8329`). To make them readable, we use two primary rules:

1. **Remove Leading Zeros:** In any 16-bit section (quartet), you can drop the zeros at the start. `0db8` becomes `db8`.
2. **The Double Colon (`::`):** You can replace a contiguous string of all-zero quartets with a single `::`.

> [!Caution]
> **The One-Shot Rule:** The "::" can **only be used once** in an address. If you use it twice, the computer won't know how many zeros to put in each spot, making the address ambiguous.

---

## 5. IPv6 Address Structure

Unlike the flexible subnetting of IPv4, a standard IPv6 Unicast address follows a predictable 48-16-64 split:

* **Global Routing Prefix (48 bits):** Assigned by the ISP; this is how the internet knows which provider owns the address.
* **Subnet Identifier (16 bits):** Used by the internal organization to define their different VLANs or subnets.
* **Interface Identifier (64 bits):** The unique "host" portion of the address (often derived from the MAC address).

---

## 6. Basic Configuration Commands

Before a Cisco router can even think about moving IPv6 traffic, you must wake up its IPv6 engine.

### Enabling the Protocol

```bash
# Globally enable IPv6 routing (Disabled by default!)
Router(config)# ipv6 unicast-routing

# Enter the interface
Router(config)# interface gigabitEthernet 0/0

# Assign a static Global Unicast Address
Router(config-if)# ipv6 address 2002:db8:0:0::1/64

# Activate the interface
Router(config-if)# no shutdown

# Optional: Enable IPv6 on an interface without a Global IP
# This generates a Link-Local address automatically
Router(config-if)# ipv6 enable

```

---

> [!Tip]
> Always use `ipv6 unicast-routing` first! If you forget this command, your router will let you configure IPv6 addresses on interfaces, but it will refuse to actually forward packets between them.
