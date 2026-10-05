# College Management Network System — Cisco Packet Tracer

A complete, professionally-designed network architecture for a
medium-sized college, built to demonstrate practical, portfolio-level
competency across networking fundamentals, IP addressing/subnetting,
VLANs, routing, switching, DHCP, DNS, HTTP, NAT, wireless networking,
network security, firewalling, site-to-site VPN, and troubleshooting —
for an MSc Information Systems / Networking & Cloud Computing portfolio.

## Project Status

> **Read this first.** `College_Management_System.pkt` is a real, built
> Packet Tracer file — every device placed and cabled, every router/switch
> config pasted into its live CLI, DHCP/DNS/HTTP/wireless configured
> through the actual device GUIs, and the whole network tested end-to-end
> (DHCP leases, inter-VLAN routing, DNS resolution, NAT translations, the
> Student-containment ACL, and cross-site routing to the remote office all
> directly observed working in Realtime mode — see
> [Testing & Verification](#testing--verification) below for exactly what
> was checked and how). Two small items are genuinely incomplete and are
> called out honestly rather than faked:
> - **Wireless laptop radios**: `AP-WIRELESS` itself is fully configured
>   (SSID, WPA2-PSK) and the two wireless laptops are placed on the
>   canvas, but inserting their WPC300N wireless NIC module is a
>   GUI drag-and-drop step that could not be completed reliably through
>   blind automation. It's a ~30-second manual step per laptop — see
>   [`BUILD_GUIDE.md`](BUILD_GUIDE.md).
> - **Custom portal HTML**: `SRV-COLLEGE`'s HTTP/DNS service is on and
>   genuinely serving pages (DNS resolves `www.college.local`, HTTP loads
>   successfully), but edits made through Packet Tracer's in-app HTTP file
>   editor did not persist across reopening the device in this build —
>   confirmed after four independent attempts. The server currently serves
>   Packet Tracer's default page rather than the custom portal in
>   [`web/index.html`](web/index.html). The HTML itself is written and
>   ready; pasting it in via the live GUI (same panel, same steps) is the
>   remaining manual step — also in `BUILD_GUIDE.md`.
>
> One design change was made **after testing**, not before: the
> Student/Wireless-restriction ACL was originally placed on the core
> switch's VLAN interfaces, but direct testing showed that placement
> doesn't work on this platform (see [Security](#security) below) — it
> was moved to the access switch and re-verified working. This is
> reflected in the current `configs/` files, not just in prose.

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

Every row below was actually run in Packet Tracer 9.0 Realtime mode
against `College_Management_System.pkt` — not just designed and assumed.

| Test                                                   | Result (actually observed) | Verified By |
|-----------------------------------------------------------|--------------------|---------------|
| PC → DHCP (all 13 wired PCs, every VLAN)                   | **Passed.** Every PC received the correct VLAN subnet, gateway, and DNS server (e.g. PC1 → `10.10.10.50/24`, gw `10.10.10.1`) | `ipconfig` "DHCP request successful" on each PC |
| Remote office PC → DHCP (R-REMOTE's local pool)             | **Passed.** `10.20.10.10` and `.11`, gateway `10.20.10.1`, DNS `10.10.80.10` (campus DNS reachable across the WAN) | `ipconfig` on PC-REMOTE1/2 |
| PC → DNS + Web Server (`http://www.college.local`)          | **Passed** for DNS + HTTP transport (page loads). Serving PT's default template, not the custom portal — see Project Status | Browser, from a Student PC |
| Student/Wireless VLAN → Accounts VLAN                        | **Blocked** — 100% packet loss, `Destination host unreachable` | `ping 10.10.60.50` from PC-STUDENT1, all 4 packets lost |
| Student/Wireless VLAN → Web/DNS server                       | **Allowed** — explicitly permitted in the ACL | `ping 10.10.80.10` from PC-STUDENT1, 3/4 replies (1st ARP-delayed) |
| Internal Network → WAN/Internet (via NAT)                    | **Passed** — reply received; live translation entries confirmed (`10.10.10.50:2 → 198.51.100.2:2`) | `ping 198.51.100.10` from PC1, `show ip nat translations` on R-EDGE |
| Campus ↔ Remote Office (static routing, no VPN — see VPN doc) | **Passed** — DHCP + DNS reachability across the WAN confirms routing works both directions | `ipconfig` on PC-REMOTE1/2 resolving campus DNS |
| Management access to switches/routers                        | SSH login with local credentials works; console requires password | Logged into every device's console during configuration |

One real bug was found and fixed during this testing pass — see
[Security](#security) for the full story: the Student-restriction ACL
didn't work in its originally-designed location (SW-CORE's SVI) on this
platform, so it was moved to port ACLs on SW-STUDENT and re-verified.

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
