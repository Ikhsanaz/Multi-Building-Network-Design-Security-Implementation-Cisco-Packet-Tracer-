# Multi-Building-Network-Design-Security-Implementation-Cisco-Packet-Tracer-

A hierarchical 3-building LAN architecture designed, configured, and validated in **Cisco Packet Tracer**. The implementation integrates **VLAN segmentation**, **Multi-Area OSPF routing** with MD5 authentication and ABR route summarization, centralized **DHCP Relay services**, and **Layer 2/3 security hardening** (Extended ACLs, DHCP Snooping, and Port Security).
---

## 1. Project Overview & Requirements

* **Architecture Scope:** 3 buildings across 10 floors supporting **709 active hosts** (scaled to **~923 hosts** with a **30% growth headroom**).
* **Core Technologies:** IEEE 802.1Q Trunking, Inter-VLAN Routing (SVI), Multi-Area OSPF (`Area 0`, `10`, `20`, `30`), OSPF MD5 Authentication, ABR Route Summarization, DHCP Relay (`ip helper-address`), Extended IP ACLs, DHCP Snooping, and Sticky Port Security.
* **Simulation Platform:** Cisco Packet Tracer (ISR 4331/2911 Routers, Catalyst 3560 Multilayer Switches, Catalyst 2960 Access Switches).

---

## 2. IPv4 Addressing & VLAN Allocation Plan

Subnetting was engineered using **VLSM** based on per-floor device counts plus **~30% expansion headroom**:

| Building | Floor | VLAN ID | VLAN Name | Role | Base Devices | +30% Headroom | Subnet / CIDR | SVI Gateway | Host Usable Range |
| :--- | :---: | :---: | :--- | :--- | :---: | :---: | :--- | :--- | :--- |
| **Building A** | Fl. 1 | `10` | `A-FL1` | User | 60 | 78 | `192.168.0.0/25` | `192.168.0.1` | `192.168.0.2 – 192.168.0.126` |
| **Building A** | Fl. 2 | `20` | `A-FL2` | User | 80 | 104 | `192.168.0.128/25` | `192.168.0.129` | `192.168.0.130 – 192.168.0.254` |
| **Building A** | Fl. 3 | `30` | `A-FL3` | User | 30 | 39 | `192.168.1.0/26` | `192.168.1.1` | `192.168.1.2 – 192.168.1.62` |
| **Building A** | Fl. 4 | `40` | `A-FL4` | User | 10 | 13 | `192.168.1.64/28` | `192.168.1.65` | `192.168.1.66 – 192.168.1.78` |
| **Building B** | Fl. 1 | `110` | `B-FL1` | User | 120 | 156 | `192.168.2.0/24` | `192.168.2.1` | `192.168.2.2 – 192.168.2.254` |
| **Building B** | Fl. 2 | `120` | `B-FL2` | User | 90 | 117 | `192.168.3.0/25` | `192.168.3.1` | `192.168.3.2 – 192.168.3.126` |
| **Building B** | Fl. 3 | `130` | `B-FL3` | User | 12 | 16 | `192.168.3.128/27` | `192.168.3.129` | `192.168.3.130 – 192.168.3.158` |
| **Building B** | Fl. 4 | `140` | `ADMIN` | Admin/Server | 7 | 10 | `192.168.3.160/28` | `192.168.3.161` | `192.168.3.162 – 192.168.3.174` |
| **Building C** | Fl. 1 | `210` | `C-FL1` | User | 200 | 260 | `172.16.0.0/23` | `172.16.0.1` | `172.16.0.2 – 172.16.1.254` |
| **Building C** | Fl. 2 | `220` | `C-FL2` | User | 100 | 130 | `192.168.4.0/24` | `192.168.4.1` | `192.168.4.2 – 192.168.4.254` |

### Point-to-Point Backbone Links (`/30`)

| Link Segment | Subnet | Endpoint 1 | Endpoint 2 | OSPF Area |
| :--- | :--- | :--- | :--- | :---: |
| `MLS-A` ↔ `Router-A` | `10.1.0.0/30` | `MLS-A Fa0/1` (`10.1.0.1`) | `Router-A Gi0/0/1` (`10.1.0.2`) | `Area 0` |
| `Router-A` ↔ `Router-B` | `10.1.0.4/30` | `Router-A Gi0/0/0` (`10.1.0.5`) | `Router-B Gi0/1` (`10.1.0.6`) | `Area 0` |
| `MLS-B` ↔ `Router-B` | `10.1.1.0/30` | `MLS-B Fa0/1` (`10.1.1.1`) | `Router-B Gi0/0` (`10.1.1.2`) | `Area 0` |
| `Router-B` ↔ `Router-C` | `10.1.2.0/30` | `Router-B Gi0/2` (`10.1.2.1`) | `Router-C Gi0/0/0` (`10.1.2.2`) | `Area 0` |
| `MLS-C` ↔ `Router-C` | `10.1.3.0/30` | `MLS-C Fa0/1` (`10.1.3.1`) | `Router-C Gi0/0/1` (`10.1.3.2`) | `Area 0` |

---

## 3. Routing & Infrastructure Services

* **Multi-Area OSPF Hierarchy:**
  * **Backbone (`Area 0`):** Interconnects `Router-A`, `Router-B`, `Router-C`, and the routed uplinks of `CORE-A`, `CORE-B`, and `CORE-C`.
  * **Building Areas:** Building A (`Area 10`), Building B (`Area 20`), and Building C (`Area 30`).
  * **ABR Route Summarization:** Configured on `CORE-A` (`area 10 range 192.168.0.0 255.255.254.0`), `CORE-B` (`area 20 range 192.168.2.0 255.255.254.0`), and `CORE-C` (`area 30 range 172.16.0.0 255.255.254.0`) to minimize LSA flooding and shrink backbone routing tables.
  * **Routing Authentication:** Enabled **OSPF MD5 authentication** (`ip ospf message-digest-key 1 md5`) across all `Area 0` interfaces.
* **Centralized DHCP & Relay:**
  * Dedicated DHCP Server located in `VLAN 140` (`192.168.3.162`) serving address pools for all user VLANs (`VLAN 10–40`, `110–130`, `210–220`) via `ip helper-address 192.168.3.162` on each SVI.
  * Local DHCP pool configured on `CORE-B` for `VLAN 140` (Admin).

---

## 4. Network Security Hardening (My Primary Contribution)

To protect administrative infrastructure and mitigate Layer 2/Layer 3 internal threats, three defensive layers were implemented:

1. **Extended IP Access Control Lists (`ACL 100`) on Multilayer Switches:**
   * Applied inbound (`ip access-group 100 in`) across all user SVIs (`VLAN 10, 20, 30, 40, 110, 120, 130, 210, 220`).
   * **Policy Logic:** Permits `icmp echo-reply` to `192.168.3.160/28` (allowing admins in `VLAN 140` to ping and troubleshoot user hosts), explicitly denies `icmp echo` requests from user VLANs toward `VLAN 140`, and permits legitimate service traffic (`DHCP relay`, `OSPF`, inter-VLAN data).
2. **DHCP Snooping on Access Switches:**
   * Enabled globally and per active user VLAN (`ip dhcp snooping vlan <id>`).
   * **Trust Boundary:** Configured uplink ports toward Multilayer Switches (`Fa0/1`) as `ip dhcp snooping trust`, while enforcing `ip dhcp snooping limit rate 10` (10 pps) on untrusted host access ports to prevent rogue DHCP servers and DHCP starvation attacks.
3. **Port Security on Edge Access Ports:**
   * Enabled `switchport port-security` with `maximum 2`, `mac-address sticky`, and `violation restrict` across user-facing ports—dropping unauthorized frames and logging security violations without shutting down legitimate shared ports.

---

## 5. Verification & Proof-of-Concept (PoC) Summary

| Test Case | Verification Command / Method | Result |
| :--- | :--- | :---: |
| Cross-Building Inter-VLAN Connectivity | `ping 192.168.4.3` from `VLAN 10` (`Building A`) to `VLAN 220` (`Building C`) | **Pass** (0% loss, avg 5ms) |
| Admin-to-User Reachability | `ping 192.168.0.3` from `VLAN 140` (`Admin`) to `VLAN 10` | **Pass** (ICMP Echo Reply permitted) |
| Unauthorized Ping Block to Admin VLAN | `ping 192.168.3.162` from `VLAN 210` (`Building C`) | **Pass** (`Destination unreachable` via ACL 100) |
| OSPF Neighbor & Summarized Routes | `show ip ospf neighbor` & `show ip route ospf` (`O IA` entries verified) | **Pass** (`FULL/DR` & `FULL/BDR` states) |
| Dynamic DHCP Lease via Relay | `ipconfig` on client PCs across all VLANs | **Pass** (Valid IP, mask, gateway, DNS) |
| Rogue DHCP & MAC Spoofing Mitigation | `show ip dhcp snooping` & `show port-security interface fa0/2` | **Pass** (Trusted uplink & `Restrict` active) |

---

## 6. Team & Credits

* **Ikhsan Aydin Zahid** — *Network Security* (Extended ACLs, DHCP Snooping, Port Security)
* **Muhammad Ramadhan** — *Routing & Network Design* (Topology, IP Addressing, Multi-Area OSPF)
* **Henry Ersinalsal Ginting** — *VLAN & Switching* (VLANs, 802.1Q Trunking, Switch Segmentation)
* **Rifki Ramadhan** — *Infrastructure & Services* (DHCP Server, SVI Gateways, Service Integration)
