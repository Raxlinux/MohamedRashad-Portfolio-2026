Day 5: VLSM Design & Inter-VLAN Routing

Summary: 
Today’s focus shifted from basic connectivity to address space optimization and segmenting broadcast domains. I moved away from flat network topologies by taking a 172.16.10.0/23 block and applying Variable Length Subnet Masking (VLSM). By calculating specific requirements for Engineering, Sales, HR, and Guest segments, I eliminated the "address wastage" common in standard Classful or fixed-length subnetting. It’s the difference between a disorganized warehouse and a custom-engineered office floor plan.

Technical Execution:
Host Requirement Analysis: Performed manual calculations to accommodate varying department sizes: 120 hosts for Engineering, 50 for Sales, 30 for Guests, and 25 for HR. I prioritized the largest subnets first to ensure contiguous block allocation.

VLAN Segmentation: 
Logically partitioned the switch into 5 distinct VLANs. This ensures layer-2 isolation, keeping high-risk traffic (Guest WiFi) physically and logically separated from the Server Farm.

Router-on-a-Stick (ROAS) Implementation: 
Configured a Core Router to handle Inter-VLAN routing. By utilizing sub-interfaces on a single physical link, I enabled communication between segmented departments without the need for multiple physical router ports.

Edge Integration: 
Routed the internal topology through an Edge Router to a simulated ISP Cloud to test NAT boundaries and external reachability.

Key Commands:
- interface Gig0/0/0.X: Initialized logical sub-interfaces for the ROAS gateway.

- encapsulation dot1Q X: Applied the IEEE 802.1Q standard to tag frames, ensuring the router can distinguish between VLAN traffic.

- show ip route: Verified the routing table to ensure all VLSM-calculated subnets were correctly identified as directly connected.

- show interfaces trunk: Confirmed the trunk link status between the switch and router to ensure VLAN tags were being preserved across the link.

Lessons & Architecture Observations:

Sequential Allocation: In VLSM, the order of operations is critical. Starting with the largest host requirement isn't just a "tip"—it’s a requirement to prevent overlapping and wasted address space.

Throughput Constraints: While Router-on-a-Stick is a cost-effective solution for small-to-mid-sized environments, it creates a single point of failure and a bandwidth bottleneck. In a production environment with high inter-departmental traffic, I’d migrate this logic to a Layer 3 Multi-Layer Switch (SVI) for wire-speed routing.

Trunk Discipline: Misconfigured native VLANs or pruned trunks are the primary causes of silent failures. Maintaining strict documentation of which VLANs are permitted on a trunk is non-negotiable for troubleshooting.

The Zero-Trust Transition: Establishing connectivity is only half the battle. The next step is implementing Access Control Lists (ACLs) to ensure that "can communicate" doesn't become "is allowed to communicate."