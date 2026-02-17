## Day 9: Pathfinding (Routing Fundamentals & Static Logic)

### Summary:

Yesterday was about Layer 2 forwarding on switches and understanding the difference between CAM (how switches store MAC addresses) and TCAM (how they handle more complex lookups like ACLs and QoS).

Today, I moved up the stack to see how routers handle the "big picture" logic of moving data between entirely different networks. If the switch is responsible for getting a letter to the right room in an office building, the router is the logistics center deciding which highway the truck needs to take to get to another city.

I focused on the logic of the Routing Table—the brain of the router—and practiced the manual art of Static Routing. Understanding exactly how a router chooses an exit path is critical before I let dynamic protocols like OSPF do the heavy lifting for me later.

### What I Covered:

**1. What is Routing?**
At its core, routing is the process of forwarding a packet from one network to another. While a switch cares about MAC addresses (Layer 2), a router only cares about the IP destination (Layer 3).

**Types of Routing:**
* **Static Routing:** Manually configured paths. I tell the router exactly which way to go.
* **Dynamic Routing:** Routers use protocols (like OSPF or EIGRP) to "talk" to each other and learn the best paths automatically.
* **Default Routing:** A special type of static route used when the router doesn't have a specific match for a destination.


**2. Connected vs. Local Routes**
When I look at a Cisco routing table (show ip route), the first things I see are the routes the router "just knows" because cables are plugged into it.
* **Connected (C):** This represents the entire subnet attached to an interface (e.g., 192.168.1.0/24).
* **Local (L):** This is a relatively modern addition to IOS. It represents the specific, individual IP address assigned to the router's own interface (a /32 host route).

**3. Static Routing: The Manual Map**
I practiced telling the router: *"If you want to reach Network X, send the data to Point Y."* It’s stable and uses zero CPU overhead, but it doesn't scale well—if a link goes down, a static route stays there until I manually delete it.

**4. The Gateway of Last Resort (Default Route)**
This is the "I don't know, just send it here" route. 
* **Logic:** Defined as **0.0.0.0 0.0.0.0**. 
* **Usage:** Usually points toward the ISP. If a packet's destination isn't in the routing table, the router sends it to the Default Route instead of dropping it.

---

### Hands-on: Configuring Routes in Packet Tracer

I worked through the syntax for the ip route command, which follows this pattern:  
Router(config)# ip route [destination_network] [subnet mask] [next_hop]

I experimented with the three different ways to define that "next step":

* **Next-Hop IP Address:** I tell the router the IP of the next router's interface. This requires a "recursive lookup" (the router has to look up how to reach that IP first).
* **Exit Interface:** I tell the router to just spit the packet out of a specific port (e.g., Gig0/0). This is best for point-to-point links.
* **Fully Specified (Both):** Providing both the Exit Interface and the Next-Hop IP. This is the most efficient and precise method, especially in multi-access networks, as it removes any ambiguity for the router.

---

**Analogy: "The Postal Service"**
* **The Connected Route:** Mail being delivered within the same building.
* **The Static Route:** A specific instruction saying, "All mail for New York must go into the Blue Bin."
* **The Default Route:** The "Dead Letter Office" or the "General Outbox." If the clerk doesn't recognize the city on the envelope, they just throw it in the truck headed to the main regional hub and hope they can figure it out.

---

**Up Next: The Triple-Router Lab**
Now that I understand the theory, I’m going to build a more complex topology tomorrow: 

**2 PCs connected via 3 Routers.** 
The goal is to manually configure every hop so that the PCs can ping each other across the entire chain. This will test my ability to configure "return paths"—the #1 reason pings fail in real-world static setups.
