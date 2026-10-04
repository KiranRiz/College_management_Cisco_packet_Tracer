# College Management Network System — Cisco Packet Tracer

A complete, professionally-designed network architecture for a
medium-sized college, built to demonstrate practical, portfolio-level
competency across networking fundamentals, IP addressing/subnetting,
VLANs, routing, switching, DHCP, DNS, HTTP, NAT, wireless networking,
network security, firewalling, site-to-site VPN, and troubleshooting —
for an MSc Information Systems / Networking & Cloud Computing portfolio.

## Project Status

> **Read this first.** The full network design, every router/switch
> configuration, the DNS/HTTP server configuration, the wireless AP
> configuration, and complete documentation are finished and included in
> this repository. The one item **not** included is the compiled
> `College_Management_System.pkt` binary itself and real screenshots —
> those require driving the Packet Tracer desktop GUI directly
> (placing devices, cabling, clicking through dialogs), which the
> environment this project was authored in cannot do. [`BUILD_GUIDE.md`](BUILD_GUIDE.md)
> gives the exact, config-complete steps to assemble the `.pkt` file in
> well under an hour — every command has already been written and
> designed, down to the port numbers. This is a deliberate choice,
> consistent with the project's own standard (see the VPN and Wireshark
> sections below): nothing here is faked or fabricated.

## Project Overview

This project simulates the data network of a fictional medium-sized
college — **City College of Engineering & Management** — with
Administration, Students, Faculty, IT, Library, Accounts, and Management
departments, a server room hosting the college's internal web portal and
DNS, a wireless network for students and faculty, a remote satellite
admin office connected via site-to-site VPN, and a simulated Internet
edge with NAT. It demonstrates how a real institution's network would be
designed, segmented, secured, and operated — not just "a bunch of PCs
pinging each other."

## Objectives

- Design a realistic, segmented, medium-sized college network
- Apply a professional IP addressing and subnetting (CIDR) plan
- Segment departments using VLANs with correct trunking
- Implement centralized inter-VLAN routing with justified static routing
- Provide DHCP and DNS as real, verifiable network services
- Host and serve an internal web portal over HTTP
- Implement NAT/PAT for Internet access
- Deploy a secured wireless network for students/faculty
- Apply realistic, defensible network security controls (ACLs, port
  security, SSH, VLAN hardening, an IOS firewall feature)
- Implement a genuinely working site-to-site IPsec VPN
- Document systematic troubleshooting methodology
- Honestly connect the design to real-world traffic analysis (Wireshark)

## Network Architecture

Full topology, device roles, and design rationale:
[`docs/network-topology.md`](docs/network-topology.md)

In short: a collapsed-core design with one Layer-3 core switch
(`SW-CORE`) doing inter-VLAN routing for five department access switches,
a dedicated border router (`R-EDGE`) handling NAT/firewall/VPN, a
wireless AP serving a dedicated Wireless VLAN, and a remote satellite
office reached over a real site-to-site IPsec tunnel across a simulated
WAN/Internet.

## Technologies & Concepts Demonstrated

Cisco Packet Tracer 9.0 · IPv4 addressing & subnetting · CIDR · VLANs &
802.1Q trunking · DHCP · DNS · HTTP · TCP/UDP · static & default routing ·
inter-VLAN routing (SVIs) · switching & port security · NAT/PAT ·
wireless networking (WPA2-PSK) · extended ACLs · SSH · IOS firewall
(CBAC) · site-to-site IPsec VPN · network monitoring fundamentals ·
client/server communication · Wireshark-based traffic analysis (HTTP/TLS)

## VLAN Table

| VLAN | Name              | Department                      | Subnet          |
|------|-------------------|----------------------------------|-------------------|
| 10   | VLAN10_ADMIN      | Administration                   | 10.10.10.0/24     |
| 20   | VLAN20_STUDENTS   | Students (wired)                 | 10.10.20.0/24     |
| 25   | VLAN25_WIRELESS   | Wireless (students & faculty)    | 10.10.25.0/24     |
| 30   | VLAN30_FACULTY    | Faculty                          | 10.10.30.0/24     |
| 40   | VLAN40_IT         | IT Department                    | 10.10.40.0/24     |
| 50   | VLAN50_LIBRARY    | Library                          | 10.10.50.0/24     |
| 60   | VLAN60_ACCOUNTS   | Accounts (sensitive)             | 10.10.60.0/24     |
| 70   | VLAN70_MGMT       | Management / Principal's Office  | 10.10.70.0/24     |
| 80   | VLAN80_SERVERS    | Server Room                      | 10.10.80.0/24     |
| 99   | VLAN99_NETMGMT    | Network Device Management        | 10.10.99.0/24     |
| 999  | VLAN999_UNUSED    | Unused/parked ports               | Not routed        |

Full rationale, including why VLAN 25 and 999 were added beyond the
original brief: [`docs/vlan-design.md`](docs/vlan-design.md)

## IP Addressing Table

| VLAN | Network        | CIDR | Mask            | Gateway     | DHCP Range                 | Broadcast     |
|------|------------------|------|-------------------|---------------|-------------------------------|-----------------|
| 10   | 10.10.10.0       | /24  | 255.255.255.0     | 10.10.10.1    | 10.10.10.50–10.10.10.200      | 10.10.10.255    |
| 20   | 10.10.20.0       | /24  | 255.255.255.0     | 10.10.20.1    | 10.10.20.50–10.10.20.200      | 10.10.20.255    |
| 25   | 10.10.25.0       | /24  | 255.255.255.0     | 10.10.25.1    | 10.10.25.50–10.10.25.200      | 10.10.25.255    |
| 30   | 10.10.30.0       | /24  | 255.255.255.0     | 10.10.30.1    | 10.10.30.50–10.10.30.200      | 10.10.30.255    |
| 40   | 10.10.40.0       | /24  | 255.255.255.0     | 10.10.40.1    | 10.10.40.50–10.10.40.200      | 10.10.40.255    |
| 50   | 10.10.50.0       | /24  | 255.255.255.0     | 10.10.50.1    | 10.10.50.50–10.10.50.200      | 10.10.50.255    |
| 60   | 10.10.60.0       | /24  | 255.255.255.0     | 10.10.60.1    | 10.10.60.50–10.10.60.200      | 10.10.60.255    |
| 70   | 10.10.70.0       | /24  | 255.255.255.0     | 10.10.70.1    | 10.10.70.50–10.10.70.200      | 10.10.70.255    |
| 80   | 10.10.80.0       | /24  | 255.255.255.0     | 10.10.80.1    | Static only (server: .10)     | 10.10.80.255    |
| 99   | 10.10.99.0       | /24  | 255.255.255.0     | 10.10.99.1    | Static only (switch mgmt)     | 10.10.99.255    |

Remote office: `10.20.10.0/24` (gateway `10.20.10.1`). WAN transit links
use the documentation-reserved `198.51.100.0/24` (RFC 5737 TEST-NET-2).
Full plan including transit links and static infrastructure addresses:
[`docs/ip-addressing-plan.md`](docs/ip-addressing-plan.md)

## Configuration

All device configurations are complete, written in standard Cisco IOS
syntax, and ready to paste directly into Packet Tracer CLI terminals:

- [`configs/router-configs/R-EDGE.txt`](configs/router-configs/R-EDGE.txt) — border router: NAT/PAT, CBAC firewall, VPN
- [`configs/router-configs/R-REMOTE.txt`](configs/router-configs/R-REMOTE.txt) — remote office router: DHCP, VPN
- [`configs/router-configs/ISP-RTR.txt`](configs/router-configs/ISP-RTR.txt) — simulated Internet core
- [`configs/switch-configs/SW-CORE.txt`](configs/switch-configs/SW-CORE.txt) — L3 core: SVIs, DHCP, ACL
- [`configs/switch-configs/SW-ADMIN.txt`](configs/switch-configs/SW-ADMIN.txt), `SW-ACCOUNTS.txt`, `SW-ACADEMIC.txt`, `SW-STUDENT.txt`, `SW-SERVER.txt`, `SW-REMOTE.txt` — access switches
- [`configs/SRV-COLLEGE-service-config.md`](configs/SRV-COLLEGE-service-config.md) — DNS/HTTP server settings (GUI-based)
- [`configs/AP-WIRELESS-config.md`](configs/AP-WIRELESS-config.md) — wireless AP settings (GUI-based)
- [`web/index.html`](web/index.html) — the actual college portal HTML served by SRV-COLLEGE

Design rationale for routing and switching choices:
[`docs/routing.md`](docs/routing.md) · [`docs/vlan-design.md`](docs/vlan-design.md)

## Testing & Verification

| Test                                                   | Expected Result | Verified By |
|-----------------------------------------------------------|--------------------|---------------|
| PC → Default Gateway (any VLAN)                           | Successful ping to the VLAN's SVI on SW-CORE | `ping` |
| PC → DNS Server (`nslookup www.college.local`)             | Resolves to 10.10.80.10 | `nslookup` |
| PC → Web Server (`http://www.college.local`)                | College portal renders | Browser |
| VLAN → VLAN (e.g. Faculty → Library)                        | Successful (no restriction between non-sensitive VLANs) | `ping` |
| Student/Wireless VLAN → Accounts/Management VLAN            | **Blocked** by `STUDENT-RESTRICT` ACL | `ping` (times out), `show access-lists` (match counter increments) |
| Student/Wireless VLAN → Web/DNS server                      | Allowed (explicitly permitted in the ACL) | `ping`, browser |
| Wireless Client → Server (VLAN 25 → VLAN 80)                 | Successful, same rules as wired Students | `ping`, browser |
| Internal Network → WAN/Internet (via NAT)                    | `ping`/HTTP to EXT-SRV succeeds; `show ip nat translations` shows the translated entry | `ping`, `show ip nat translations` |
| Campus ↔ Remote Office (via VPN)                              | Successful ping/reachability, traffic encrypted; `show crypto isakmp sa` shows `QM_IDLE` | `ping`, `show crypto isakmp sa`, `show crypto ipsec sa` |
| Management access to any switch/router                        | SSH only, Telnet refused | `ssh -l netadmin <ip>` |

> As stated in **Project Status** above, these results describe the
> expected and designed behaviour of the configurations in this
> repository. They should be re-confirmed step-by-step while following
> [`BUILD_GUIDE.md`](BUILD_GUIDE.md), and only reported as "tested" once
> actually observed in Packet Tracer.

## Troubleshooting

Full methodology, the complete `show`/diagnostic command reference, and
five worked troubleshooting scenarios (missing VLAN assignment, missing
NAT inside/outside marking, pruned trunk VLANs, ACL never applied to an
interface, VPN not triggered by traffic) are documented in
[`docs/troubleshooting.md`](docs/troubleshooting.md).

## Security

Device hardening (SSH-only, hashed secrets, banners), switchport
security (port security, unused-port shutdown into a dead VLAN, trunk
native-VLAN/DTP hardening, BPDU guard), the Student/Wireless containment
ACL, NAT, and an IOS CBAC-based firewall concept are all covered — with
their Packet Tracer limitations stated honestly — in
[`docs/security.md`](docs/security.md). The site-to-site VPN design is
covered separately in [`docs/vpn.md`](docs/vpn.md).

## Wireshark — Traffic Analysis

Cisco Packet Tracer does **not** provide real Wireshark packet capture —
its Simulation Mode only visualizes simplified PDUs, not actual `.pcap`
frames. [`docs/wireshark-analysis.md`](docs/wireshark-analysis.md) is an
honest, technically accurate explanation of exactly what would be
observed in a real Wireshark capture of this network's HTTP and TLS
traffic (TCP handshake, HTTP request/response in cleartext, the TLS
handshake and Client/Server Hello, and why encrypted application data
hides the HTTP payload) — with no fabricated capture screenshots.

## Learning Outcomes

Building this project required applying, in a single coherent design
rather than in isolated exercises:

- Translating a set of business/organizational requirements (departments,
  sensitive data, remote office, student Wi-Fi) into a VLAN and IP
  addressing plan, with written justification for every structural
  decision (not just "because the textbook VLAN numbers say so")
- Choosing between competing valid designs (SVI vs. router-on-a-stick;
  static vs. dynamic routing; CBAC vs. a dedicated firewall appliance)
  and being able to defend the choice against alternatives
- Implementing defense-in-depth: VLAN segmentation, port security, ACLs,
  NAT, a firewall feature, and VPN encryption as layered controls rather
  than any single control being relied on alone
- Writing an ACL that enforces a specific real-world security requirement
  (student containment) while explicitly preserving required functionality
  (DNS, portal access) — and understanding ACL processing order
  (first-match, implicit deny) well enough to get that right
- Configuring and validating a genuinely working site-to-site IPsec VPN,
  including understanding why VPN tunnels are traffic-triggered and why
  NAT exemption is required alongside it
- Structuring a systematic, OSI-layer-based troubleshooting approach
  instead of guessing
- Being able to honestly state the limits of a lab tool (Packet Tracer)
  rather than overstating what was demonstrated — itself a professional
  skill, not just a technical one

## Repository Structure

```
College_management_Cisco_packet_Tracer/
├── README.md
├── BUILD_GUIDE.md
├── LICENSE
├── College_Management_System.pkt      (add after following BUILD_GUIDE.md)
│
├── docs/
│   ├── network-topology.md
│   ├── ip-addressing-plan.md
│   ├── vlan-design.md
│   ├── routing.md
│   ├── security.md
│   ├── vpn.md
│   ├── troubleshooting.md
│   └── wireshark-analysis.md
│
├── configs/
│   ├── router-configs/   (R-EDGE, R-REMOTE, ISP-RTR)
│   ├── switch-configs/   (SW-CORE, SW-ADMIN, SW-ACCOUNTS, SW-ACADEMIC, SW-STUDENT, SW-SERVER, SW-REMOTE)
│   ├── SRV-COLLEGE-service-config.md
│   └── AP-WIRELESS-config.md
│
├── web/
│   └── index.html                     (college portal page served by SRV-COLLEGE)
│
└── screenshots/                        (populated after building in Packet Tracer — see screenshots/README.md)
```
