# Day 26: Extended Access Control Lists (ACLs)

While Standard ACLs offer basic filtering based on source addresses, **Extended ACLs** provide the "surgical precision" required for modern network security. They allow administrators to filter traffic based on a wide array of parameters, including source and destination IP addresses, Layer 4 protocols (TCP, UDP, ICMP), and specific port numbers.

---

## 1. Extended ACL Fundamentals

Extended ACLs function under the same logic as Standard ACLs—they are an **ordered sequence of ACEs** processed from **top to bottom**. However, they are significantly more powerful due to their granular matching capabilities.

### Key Characteristics

* **Identification:** * **Numbered:** Uses ranges **100–199** and **2000–2699**.
  * **Named:** Identified by a descriptive alphanumeric string (e.g., `APP_SERVER_FILTER`).
* **The "Exact Match" Rule:** For an Extended ACL entry to take effect, **every single parameter** defined in the statement must match the packet. If even one value (like a port number or a protocol) does not match, the router moves to the next entry in the list.
* **The Global Permit:** To allow all traffic (overriding the implicit deny), use the command `permit ip any any`. The `ip` keyword here acts as a wildcard for all underlying protocols (TCP, UDP, ICMP, etc.).

---

## 2. Configuration Syntax

### Standard Numbered Extended ACL

```bash
Router(config)# access-list <100-199 | 2000-2699> {permit | deny} <protocol> <source-ip> <source-wildcard> <dest-ip> <dest-wildcard>
```

### Standard Named Extended ACL

Named ACLs are often preferred in professional environments because they allow for easier editing and better documentation.

```bash
Router(config)# ip access-list extended <ACL_NAME>
Router(config-ext-nacl)# [sequence-num] {permit | deny} <protocol> <src-ip> <src-wildcard> <dst-ip> <dst-wildcard> [operator port]
```

---

## 3. Matching Layer 4 Port Numbers

Extended ACLs allow you to target specific services by specifying port numbers. These operators can be placed after the source address, the destination address, or both, depending on your requirements.

| Operator | Description | Example |
| :--- | :--- | :--- |
| **`eq`** | Exactly this port number | `eq 80` (HTTP) |
| **`neq`** | Match everything **except** this port | `neq 22` (Not SSH) |
| **`lt`** | Everything **lower** than this port | `lt 1024` (Well-known ports) |
| **`gt`** | Everything **greater** than this port | `gt 1024` (Ephemeral ports) |
| **`range`** | A specific **inclusive range** | `range 20 21` (FTP data/control) |

### Practical Example

To deny a specific host (`192.168.1.4`) from accessing a web server network (`10.0.1.0/24`) via HTTPS (Port 443):

```bash
access-list 101 deny tcp host 192.168.1.4 10.0.1.0 0.0.0.255 eq 443
access-list 101 permit ip any any
```

---

## 4. Advanced Matching Parameters

Beyond simple IP and Port matching, Extended ACLs can inspect deep packet headers for specific flags and metadata:

* **TCP Flags:** Match based on `ack`, `fin`, `syn`, `rst`, `psh`, or `urg`.
* **`established`:** A powerful tool that only permits TCP traffic that is part of an existing connection (where the ACK or RST bit is set).
* **QoS/Traffic Management:** Match based on `dscp` (Differentiated Services Code Point) or `precedence` values.
* **Meta Data:** Match based on the `ttl` (Time to Live) value.

---

## 5. Strategic Placement: The "Source" Rule

Unlike Standard ACLs, which are placed near the destination, **Extended ACLs must be applied as close to the source as possible.**

**Why?**
Because Extended ACLs know the destination, there is no risk of "accidentally" blocking traffic to other parts of the network. By dropping unwanted traffic at the very first router interface it hits, you save:

1. **Bandwidth:** Unwanted traffic doesn't traverse the network core.
2. **Router Resources:** Other routers don't have to spend CPU cycles routing a packet that is ultimately destined to be dropped.

---

## 6. Full Command Reference (Numbered)

For complex implementations, the full syntax for a numbered Extended ACL entry includes options for logging and time-based restrictions:

```bash
access-list <100-199> [dynamic <name> {timeout <min>}] {deny | permit} 
    <protocol> 
    <src-ip> <src-wildcard> [operator port] 
    <dest-ip> <dest-wildcard> [operator port] 
    [established] [precedence <val>] [tos <val>] [dscp <val>]
    [log | log-input] [time-range <name>]
```

> [!Tip]
> Use the `log` keyword at the end of a deny statement during troubleshooting to see the source IP and MAC address of devices attempting to violate your security policy in the console logs.
