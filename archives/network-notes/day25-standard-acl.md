# Day 25: Standard Access Control Lists (ACLs)

Access Control Lists (ACLs) serve as the primary "security guards" of a network. They are essentially a set of rules used by routers to filter traffic based on specific criteria, allowing or denying packets as they attempt to pass through an interface. Understanding how to precisely control this flow is fundamental to network security and traffic management.

---

## 1. ACL Fundamentals and Operation

An ACL is an **ordered sequence** of permit or deny statements, known as **Access Control Entries (ACEs)**. When a packet reaches an interface where an ACL is applied, the router examines the packet against these entries sequentially, from top to bottom.

### Key Operational Rules

* **Top-Down Processing:** The router stops at the first match it finds. Once a match is made, no further entries are processed.
* **Implicit Deny:** Every ACL ends with an invisible "deny all" statement. If a packet does not match any of your defined rules, it is dropped by default.
* **Application:** ACLs are defined globally on the router but remain inactive until they are applied to a specific interface in a specific direction (**inbound** or **outbound**).
* **Capacity Limits:** A maximum of **one ACL** can be applied per interface, per direction. This means a single interface can have at most two ACLs: one for incoming traffic and one for outgoing traffic.
* **The Replacement Rule:** If you apply a new ACL to an interface that already has one enabled, the old ACL is immediately replaced by the new one.

---

## 2. Types of ACLs

While we are focusing on Standard ACLs today, it is important to distinguish them from Extended ACLs to understand their specific use cases.

| Feature | Standard ACLs | Extended ACLs |
| :--- | :--- | :--- |
| **Matching Criteria** | **Source IP Address** only. | Source/Destination IP, Protocol (TCP/UDP), and Port Numbers. |
| **Identification** | Numbered (1-99, 1300-1999) or Named. | Numbered (100-199, 2000-2699) or Named. |
| **Best Practice Placement** | As close to the **destination** as possible. | As close to the **source** as possible. |

> [!Important]
> In real-world scenarios, Extended ACLs are typically applied **inbound** on the interface closest to the source to "kill" unwanted traffic before it consumes any unnecessary bandwidth across the network.

---

## 3. Standard Numbered ACLs

Standard Numbered ACLs are the simplest form of filtering, identifying the list by a unique ID number.

### Configuration Commands

**1. Create the ACL (Global Configuration):**

```bash
Router(config)# access-list <number> {deny | permit} <ip-address> <wildcard-mask>
```

**2. Apply to an Interface:**

```bash
Router(config-if)# ip access-group <number> {in | out}
```

### Configuration Examples

* **Deny a specific network:** `access-list 10 deny 192.168.1.0 0.0.0.255`
* **Deny a specific host:** `access-list 10 deny 2.2.2.2 0.0.0.0` (or simply `access-list 10 deny host 2.2.2.2`)
* **Permit all other traffic:** `access-list 10 permit any` (essential to counteract the implicit deny).

---

## 4. Standard Named ACLs

Named ACLs offer more flexibility, allowing you to give descriptive names to your filters and edit individual lines more easily.

### Configuration Commands for Named ACLs

**1. Enter Named ACL Mode:**

```bash
Router(config)# ip access-list standard <ACL_NAME>
```

**2. Define the ACEs:**

```bash
Router(config-std-nacl)# <sequence-number> {permit | deny} <ip-address> <wildcard-mask>
```

**3. Apply to an Interface:**

```bash
Router(config-if)# ip access-group <ACL_NAME> {in | out}
```

---

## 5. Verification and Troubleshooting

Properly verifying your ACLs is critical, as a small logic error can accidentally block legitimate business traffic.

| Command | Purpose |
| :--- | :--- |
| `show access-lists` | Displays all ACLs configured on the router and their hit counts. |
| `show ip access-list` | Specifically shows IP ACL details. |
| `show ip interface <int>` | Shows which ACL is applied to a specific interface and in which direction. |
| `show running-config \| section access-list` | Shows the literal configuration syntax currently in use. |

---

> [!Tip]
> **Placement Strategy:** Always place Standard ACLs as close to the **destination** as possible. Because they only filter based on the Source IP, placing them too close to the source might accidentally block that user from reaching other legitimate destinations.
