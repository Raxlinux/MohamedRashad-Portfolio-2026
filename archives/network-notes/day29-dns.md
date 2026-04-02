# Day 29: Domain Name System (DNS)

If IP addresses are the "phone numbers" of the internet, then **DNS** is the ultimate contacts list. While machines thrive on the logic of 32-bit or 128-bit addresses, humans are notoriously bad at remembering strings of numbers. DNS bridges this gap, ensuring we can navigate the web using names like `google.com` instead of `142.250.190.46`.

---

## 1. The Core Concept: Names vs. Numbers

In a network, devices communicate using **IP Addresses**. However, asking a user to remember the specific IP for every website they visit is impractical.

* **Human Preference:** We use meaningful, alphanumeric names (Domain Names).
* **Machine Preference:** Routers and PCs use numerical identifiers (IP Addresses).

Every time you type a URL into your browser, a background process occurs where your device "resolves" that name into an IP address.

### How your device gets DNS information

Your device needs to know which DNS server to "ask." This is handled in two ways:

1. **Manually:** You hardcode the IP of a DNS server (like Google's `8.8.8.8`) into your network settings.
2. **Automatically:** Your device receives DNS server information from a **DHCP (Dynamic Host Configuration Protocol)** server when it joins the network.

---

## 2. DNS Protocol & Port Mechanics

DNS doesn't just use one transport method; it adapts based on the size of the data being sent.

* **UDP Port 53:** The standard for most queries. It is fast and has low overhead, which is perfect for small request/response packets.
* **TCP Port 53:** Used when the DNS message exceeds **512 bytes** or during "Zone Transfers" between DNS servers. TCP ensures the larger data set arrives without errors.

---

## 3. Windows CMD Troubleshooting

When troubleshooting connectivity, the command prompt is your best friend. Here are the essential tools for managing DNS on a Windows machine:

| Command | Purpose |
| :--- | :--- |
| `ipconfig` | Shows basic IP configuration. |
| `ipconfig /all` | Displays detailed info, including which **DNS Servers** you are using. |
| `ipconfig /displaydns` | Shows the local DNS cache (names your PC already resolved). |
| `ipconfig /flushdns` | Clears the local cache (useful if a site moved to a new IP). |
| `nslookup <domain-name>` | Directly queries a DNS server to find the IP for a specific name. |
| `ping <ip> -n <number>` | Tests connectivity to an IP for a specific number of attempts. |

---

## 4. DNS in Cisco IOS

By default, a router acts as a simple "forwarder"—it moves packets toward their destination without caring about the DNS names inside them. However, a Cisco router can be configured to take a more active role.

### The Router as a DNS Server

You can turn your router into a local "phonebook" for your office or lab.

```bash
# Enable the DNS server function
ip dns server

# Create manual entries (Static Mappings)
ip host PC-01 192.168.1.10
ip host Printer-01 192.168.1.50
```

### The Router as a DNS Client

If you want the router itself to be able to resolve names (e.g., so you can `ping google.com` from the CLI), you must configure it as a client.

```bash
# Enable the router to look up names
ip domain lookup

# Specify which external DNS servers to ask
ip name-server 8.8.8.8
ip name-server 1.1.1.1

# Optional: Set a local domain suffix
ip domain name company.com
```

> [!TIP]
> If `ip domain lookup` is enabled but no DNS server is reachable, the router may "hang" for several seconds trying to resolve a mistyped command. You can disable this behavior with `no ip domain-lookup`.

---

## 5. Verification & Monitoring

To see what the router knows about the hosts on your network, use the following command in **Privileged Exec Mode**:

### `show hosts`

This command displays the temporary and static name-to-IP bindings currently in the router's cache.

```text
Router# show hosts
Default domain is company.com
Name servers are 8.8.8.8, 1.1.1.1

Host                      Flags      Age Type   Address(es)
PC-01                    (perm, OK)  0    IP    192.168.1.10
wikipedia.org            (temp, OK)  1    IP    103.102.166.224
```

* **perm:** A static entry created with the `ip host` command.
* **temp:** A dynamic entry learned via a DNS query.

---

## 6. The Default Gateway Role

In a standard DNS process, the router's main job is simply to provide the **Default Gateway**. It ensures the DNS request packet (UDP 53) leaves the local network and reaches the internet. Unless specifically configured as a server or client, the router does not change or "inspect" the DNS data. All that it does is removes and replace the layer 2 header - it just moves the mail.
