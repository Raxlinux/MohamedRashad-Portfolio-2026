# Day 28: Network Time Protocol (NTP) & Clock Management

In the world of networking, time isn't just a suggestion—it’s the "source of truth." Every router, switch, and firewall has an internal clock, but left to their own devices, these clocks behave like cheap wristwatches: they **drift**.

If one router thinks it's 2:00 PM and another thinks it's 2:05 PM, your logs become a jigsaw puzzle with missing pieces. When something breaks, you need to know exactly what happened and in what order. Without synchronized time, troubleshooting isn't just hard; it's impossible.

---

## 1. Why Accuracy Matters

Every event on a network device is recorded via **Syslog**. If your timestamps are off, you can't correlate logs across different devices.

> [!IMPORTANT]  
> **Real-World Impact:** In industries like **Substation Control** or **SCADA integration**, time synchronization is the most critical part of commissioning. Even if the logic and wiring are perfect, if the clocks aren't synced, you won't get accurate event logs during a power fault. In these environments, "close enough" isn't good enough.

---

## 2. Manual Clock Management

Before we automate with NTP, we need to understand how Cisco handles time internally. Most devices have two clocks:

1. **Software Clock:** Used by the IOS operating system.
2. **Hardware Clock (Calendar):** A battery-backed clock on the motherboard that keeps time even when the power is off.

### Manual Setup Commands

To set the software clock (Privileged Exec Mode):
`clock set 17:39:00 28 Mar 2026`

To set the hardware clock:
`calendar set 17:39:00 28 Mar 2026`

### Keeping Them in Sync

Typically, you want these two to match. You can "push" the time from one to the other:

* **Update Hardware from Software:** `clock update-calendar`
* **Update Software from Hardware:** `clock read-calendar`

### Timezones and Daylight Savings

By default, devices think in UTC. To make logs readable for humans, we set the offset:

* **Timezone:** `clock timezone IST 5 30` (Sets IST as UTC +5:30).
* **Daylight Savings:** `clock summer-time BST recurring last Sunday March 01:00 last Sunday October 01:00`

---

## 3. Network Time Protocol (NTP)

Manually setting the clock on 500 routers is a nightmare. **NTP (UDP Port 123)** allows devices to automatically sync their time over the network.

### The Stratum Hierarchy

NTP uses a "Stratum" system to describe the distance from a high-precision reference clock.

* **Stratum 0:** The source (Atomic clocks, GPS satellites). These aren't on the network; they are the "Reference Clocks."
* **Stratum 1:** Servers directly connected to a Stratum 0 source.
* **Stratum 15:** The maximum hop count.
* **Stratum 16:** Considered "Unsynchronized" and unreliable.

### Accuracy Expectations

* **LAN:** ~1 millisecond accuracy.
* **WAN/Internet:** ~50 milliseconds accuracy.

---

## 4. Configuration & NTP Roles

### Basic Client Setup

To sync with a public server (like Google), first find the IP: `nslookup time.google.com`.

```bash
# Global Configuration Mode
ntp server 216.239.35.4 prefer  # 'prefer' makes this the primary choice
```

### Advanced Roles

* **Master Clock:** `ntp master 8` (Forces the device to act as an authoritative source).
* **Peering:** `ntp peer <IP>` (Syncs with a device of the same stratum level).
* **NTP Source:** `ntp source loopback0` (Ensures the device uses a stable IP for its time requests).

---

## 5. Verification: The "Show" Commands

Once you've configured your clocks, you need to verify they are actually working. Here are the essential commands to run in **Privileged Exec Mode**:

### Clock & Calendar Status

| Command | What it tells you |
| :--- | :--- |
| `show clock` | Displays the current software time and date. |
| `show clock detail` | Shows the time AND the source (e.g., if it was synced via NTP). |
| `show calendar` | Displays the current time on the hardware clock hardware. |

### NTP Troubleshooting

| Command | What it tells you |
| :--- | :--- |
| `show ntp status` | The big picture. Tells you if the clock is "synchronized," its stratum, and the reference ID. |
| `show ntp associations` | Lists all servers this device is talking to. Look for the asterisk (`*`) next to the server—that’s the one currently being used as the "Master." |

### The "Payoff" Command

| Command | Why it matters |
| :--- | :--- |
| `show logging` | Displays the device logs. If NTP is working, you will see precise, human-readable timestamps for every event. |

> [!TIP]
> If `show logging` is showing numbers like `00:04:22` (uptime) instead of dates, make sure you have enabled timestamps in global config:  
> `service timestamps log datetime msec`

---

## 6. NTP Authentication (Security)

Authentication ensures your router only trusts authorized NTP servers.

```bash
ntp authenticate
ntp authentication-key 1 md5 <your_password>
ntp trusted-key 1
ntp server <IP> key 1
```

---
