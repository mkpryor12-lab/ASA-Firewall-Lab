# ASA-Firewall-Lab
Cisco ASA 5505 firewall lab — inside network protected by security levels, NAT/PAT, and ICMP inspection, with outside-to-inside traffic blocked.

ASA Firewall Lab — Cisco ASA 5505 in Packet Tracer
Based on ITTechStudies, "Cisco ASA Firewall Configuration in Packet Tracer" (39:15, Sep 14, 2026): https://youtu.be/u3O__d31fD8
Built from scratch, not the video's downloadable topology.
Devices — 6 total
Network gear:
Switch0 — 2960-24TT
Router0 — 2911
ASA 5505
Endpoints:
PC0, PC1 — inside
Server0 — outside / internet stand-in
File: ASA-Firewall-Lab.pkt
IP plan
Inside 192.168.10.0/24, Security Level 100:
PC0 192.168.10.10/24, gateway 192.168.10.1
PC1 192.168.10.11/24, gateway 192.168.10.1
ASA VLAN 1 (inside) 192.168.10.1/24
Transit 192.168.20.0/30:
ASA VLAN 2 (outside) 192.168.20.1/30, Security Level 0
Router0 Gig0/0 192.168.20.2/30
Outside 203.0.113.0/30:
Router0 Gig0/1 203.0.113.1/30
Server0 203.0.113.2/30, gateway 203.0.113.1
Cabling
PC0 FastEthernet0 → Switch0 Fa0/1
PC1 FastEthernet0 → Switch0 Fa0/2
Switch0 Fa0/24 → ASA Ethernet0/1 (inside)
ASA Ethernet0/0 → Router0 Gig0/0 (outside)
Router0 Gig0/1 → Server0 FastEthernet0
Step 1 — Baseline office network (verified)
No ASA yet. Router0 Gig0/0 was 192.168.10.1, Gig0/1 was 203.0.113.1.
All pings worked: PC0→PC1, PC0→gateway, PC0→Server0, Server0→PC0.
Step 2 — Insert ASA (verified)
ASA:
interface vlan 1 — nameif inside, security-level 100, ip address 192.168.10.1 255.255.255.0, no shutdown
interface vlan 2 — nameif outside, security-level 0, ip address 192.168.20.1 255.255.255.252, no shutdown
route outside 0.0.0.0 0.0.0.0 192.168.20.2
Router0:
Gig0/0 moved to 192.168.20.2/30
ip route 192.168.10.0 255.255.255.0 192.168.20.1
ICMP inspection (needed for ping replies through the ASA):
class-map inspection_default / match default-inspection-traffic
policy-map global_policy / class inspection_default / inspect icmp
service-policy global_policy global
Tests — all verified 2026-11-18:
PC0→PC1 192.168.10.11 — works
PC0→ASA inside 192.168.10.1 — works
PC0→Router0 192.168.20.2 — works
PC0→Server0 203.0.113.2 — works
Server0→PC0 192.168.10.10 — fails, as designed (outside Level 0 cannot start traffic to inside Level 100)
Step 3 — NAT/PAT (verified)
ASA only:
object network INSIDE-NET
 subnet 192.168.10.0 255.255.255.0
 nat (inside,outside) dynamic interface
Proof — show xlate:
ICMP PAT from inside:192.168.10.10 to outside:192.168.20.1/39844
Inside PCs are hidden behind the ASA outside address 192.168.20.1.
Troubleshooting notes
ASA Vlan1 was initially unassigned — it had no IP, so PC0→192.168.10.1 failed. Fixed by re-entering the Vlan1 IP.
Router0 Gig0/0 was initially typed as 192.168.2.2 (missing a zero). The static route would not install because next-hop 192.168.20.1 was unreachable. Corrected to 192.168.20.2, then show ip route showed S 192.168.10.0/24 via 192.168.20.1.
Packet Tracer's ASA has no inspection_default class-map by default — create it first. The policy-map name is global_policy (underscore, not hyphen).
Ping to the ASA's own outside address 192.168.20.1 is dropped by the ASA by default. Test through-traffic (192.168.20.2, 203.0.113.2) instead.
Next
Static NAT
ACLs
DMZ 10.0.0.0/24, Security Level 50 — Web / DNS / FTP servers
