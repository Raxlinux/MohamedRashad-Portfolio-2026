Day 6: Killing the Flat Network (VLANs & Security Hardening)

Summary: 
Today was about moving away from "amateur hour" flat networks. In a default setup, everyone can talk to everyone—which is a nightmare for both performance and security. I spent 8 hours learning how to chop a single physical switch into multiple logical ones using VLANs. I successfully separated Engineering, Sales, and Guest traffic, ensuring that if someone on the Guest Wi-Fi gets compromised, they can't just "walk" into the HR server. It’s like putting locks on the internal doors of an office instead of just the front entrance.

What I Actually Did:

Broke the Broadcast Domain: I saw firsthand how much "noise" (broadcast traffic) fills a network. By setting up VLANs, I successfully caged that noise so it stays within its own department.

Mastered the 802.1Q "Tag": I dove into how the switch actually identifies traffic. It sticks a 4-byte "tag" on the Ethernet frame. I realized that if a switch sees a frame without a tag on a trunk port, it just assumes it belongs to the "Native VLAN"—which is a huge security hole if left on the default settings.

Built a Lab that Works: I set up a switch in Packet Tracer with 4 PCs.

I locked PC 1 & 2 into VLAN 10 (Sales) and PC 3 & 4 into VLAN 20 (Engineering).

The Proof: PC 1 could ping PC 2 (same team), but when it tried to hit PC 3, it got nothing but timeouts. That "failed" ping felt like a win because it proved my logical walls were up.

Defended Against "Hopping": I read up on how hackers use VLAN Hopping to jump between segments. I learned that leaving ports on the default VLAN 1 is basically leaving the keys in the ignition.

[Screenshots of Lab and Written notes](../Screenshots%20and%20Notes/Day6_VLANs_Deepdive/)

The Command Line (What I used):

vlan 10 & name Sales: Creating the "rooms."

switchport mode access: Telling the port, "You're only talking to one device."

switchport mode trunk: Opening the "highway" between switches to carry all VLAN traffic.

switchport trunk native vlan 99: The "pro" move. I moved the native VLAN away from the default (VLAN 1) to stop double-tagging attacks.

show vlan brief: My "sanity check" to make sure I didn't misconfigure a port.

Real-World Observations:

Segmentation = Sanity: Palo Alto's docs made it clear: segmentation isn't just a "nice to have." It reduces your attack surface. If you don't segment, one infected laptop can take down the whole building.

The "Zero Trust" Mindset: Just because two devices are on the same switch doesn't mean they should be able to see each other. VLANs are the first step in a "trust no one" architecture.

Trunking is Tricky: I realized that if your trunk ports aren't configured perfectly on both ends, you’ll get "Native VLAN mismatches" or just total silence. Documentation is your best friend here.