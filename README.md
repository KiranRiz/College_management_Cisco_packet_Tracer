# College Management Network – Cisco Packet Tracer

## Overview

This project demonstrates the design and implementation of a complete
network infrastructure for a medium-sized college, built and simulated in
Cisco Packet Tracer. It models a realistic campus environment with
multiple departments, a server room, a wireless network, and a connected
remote office, and applies core networking principles — IP addressing,
VLAN segmentation, routing, DHCP, DNS, NAT, and basic network security —
in a single, coherent design.

This project was built as part of an academic project, as a practical application of networking and
infrastructure concepts covered in coursework.

## Project Objectives

* Design a realistic, segmented network topology for a college campus
* Apply a structured IP addressing and subnetting plan
* Segment departments using VLANs
* Configure inter-VLAN routing
* Deploy DHCP for automatic client configuration
* Configure DNS for internal name resolution
* Host an internal web portal over HTTP
* Implement static routing between network segments
* Configure NAT/PAT for internet access
* Deploy a secured wireless network
* Apply access control and basic network security measures
* Connect a remote office site to the main campus
* Test and verify the network using standard troubleshooting commands

## Network Architecture

The network is built around a single core switch that handles routing
between all department VLANs, with dedicated access switches for each
area of the college and a border router providing connectivity to the
wider network. The main building blocks are:

* **Core/distribution layer** – a Layer-3 switch performing inter-VLAN
  routing and DHCP for the campus
* **Departmental access switches** – separate switches for
  Administration/Management, Accounts, Faculty/Library, Students, and
  IT/Server Room, each connected back to the core switch
* **Server network** – a server hosting the college's web portal and DNS
  service
* **Wireless network** – an access point serving student and faculty
  wireless clients
* **Remote office** – a small satellite office connected back to the
  main campus over a WAN link
* **Internet/ISP edge** – a border router performing NAT and connecting
  the campus to a simulated external network

**Devices used:**

| Device Type | Quantity |
|---|---|
| Routers | 3 |
| Switches | 7 |
| Servers | 2 |
| PCs | 13 |
| Wireless Access Point | 1 |
| Wireless Laptops | 2 |

## VLAN Configuration

| VLAN ID | Name | Purpose |
|---|---|---|
| 10 | ADMIN | Administration department |
| 20 | STUDENTS | Student computer labs (wired) |
| 25 | WIRELESS | Wireless network (students & faculty) |
| 30 | FACULTY | Faculty / teaching staff |
| 40 | IT_DEPT | IT department |
| 50 | LIBRARY | Library |
| 60 | ACCOUNTS | Accounts / finance |
| 70 | MGMT_PRINCIPAL | Management / Principal's office |
| 80 | SERVERS | Server room (web and DNS server) |
| 99 | NET_MGMT | Network device management |
| 999 | UNUSED_PARKING | Unused switch ports (parked, not routed) |

## IP Addressing

The campus uses the private address block **10.10.0.0/16**, subnetted
into a `/24` per VLAN. The remote office and WAN links use separate
ranges so that routing tables stay easy to read at a glance.

| Network / VLAN | Subnet | Gateway |
|---|---|---|
| Administration (VLAN 10) | 10.10.10.0/24 | 10.10.10.1 |
| Students (VLAN 20) | 10.10.20.0/24 | 10.10.20.1 |
| Wireless (VLAN 25) | 10.10.25.0/24 | 10.10.25.1 |
| Faculty (VLAN 30) | 10.10.30.0/24 | 10.10.30.1 |
| IT Department (VLAN 40) | 10.10.40.0/24 | 10.10.40.1 |
| Library (VLAN 50) | 10.10.50.0/24 | 10.10.50.1 |
| Accounts (VLAN 60) | 10.10.60.0/24 | 10.10.60.1 |
| Management (VLAN 70) | 10.10.70.0/24 | 10.10.70.1 |
| Servers (VLAN 80) | 10.10.80.0/24 | 10.10.80.1 |
| Network Management (VLAN 99) | 10.10.99.0/24 | 10.10.99.1 |
| Remote Office LAN | 10.20.10.0/24 | 10.20.10.1 |
| Campus WAN transit | 10.10.1.0/30 | — |
| Internet/WAN links | 198.51.100.0/24 | — |

**Key server addresses**

| Server | IP Address |
|---|---|
| College Web/DNS Server (SRV-COLLEGE) | 10.10.80.10 |

Department VLANs use DHCP for client addressing; the Servers and Network
Management VLANs use static addressing only.

## Network Services

**DHCP** – Each department VLAN has its own DHCP pool configured on the
core switch, automatically assigning clients an IP address, subnet mask,
default gateway, and DNS server. The remote office has its own local
DHCP pool configured on its router.

**DNS** – An internal DNS server resolves the domain `college.local`,
with records for `www.college.local`, `dns.college.local`, and
`portal.college.local`, all pointing to the college server.

**HTTP** – The college server hosts a simple web portal, accessible to
clients over HTTP once DNS has resolved the hostname.

**NAT/PAT** – The border router translates internal private addresses to
a single public-facing address (NAT overload/PAT) so campus devices can
reach external networks without exposing internal addressing directly.

## Routing

Inter-VLAN routing is handled centrally by the core switch using Switched
Virtual Interfaces (SVIs) — one per VLAN — rather than a
router-on-a-stick design, since routing entirely within the switch is
more efficient for a network with this many VLANs.

Static routing is used throughout the network rather than a dynamic
routing protocol. With a small, stable topology — one core switch, one
border router, and one remote site — a handful of static and default
routes are sufficient to reach every destination, and this keeps the
routing design simple to configure, audit, and explain.

## Network Security

* **VLAN segmentation** – each department's traffic is logically
  separated, keeping broadcast domains and attack surface contained
* **Access Control List** – students (wired and wireless) are blocked
  from reaching the Accounts and Management VLANs, while explicitly
  retaining access to required services such as DNS and the web portal
* **Unused port hardening** – switch ports not connected to a device are
  shut down and placed into an unused, unrouted VLAN
* **Port security** – access ports restrict the number of MAC addresses
  allowed and react to violations automatically
* **Secure management access** – device management uses SSH rather than
  Telnet, with locally authenticated accounts and console passwords
* **Trunk hardening** – trunk links use a dedicated native VLAN and have
  DTP negotiation disabled

## Wireless Network

A wireless access point provides network access for student and faculty
laptops on its own dedicated VLAN, separate from the wired student
network. The wireless network uses WPA2-PSK security, and connected
clients receive their IP configuration automatically via DHCP, with the
same access restrictions applied as wired student devices.

## Remote Office

A small remote office site is connected back to the main campus over a
WAN link, routed through a simulated ISP. The remote office has its own
router, switch, and local DHCP pool, and clients there can reach campus
resources such as the internal DNS server, confirming that routing
between the two sites works correctly in both directions.

## Testing and Verification

| Test | Result |
|---|---|
| DHCP address assignment (all departments) | Clients receive correct IP, subnet mask, gateway, and DNS |
| Remote office DHCP | Clients receive a local address and can resolve campus DNS |
| Inter-VLAN connectivity | Departments can reach shared resources such as the server VLAN |
| DNS resolution | `www.college.local` resolves correctly to the college server |
| HTTP access | Web portal is reachable from client devices |
| NAT/PAT | Internal clients reach external addresses via the border router |
| ACL enforcement | Student VLAN is blocked from Accounts/Management, while DNS and web access remain permitted |
| Remote office connectivity | Devices at the remote site reach the main campus network |
| Basic routing | Routing tables reflect the expected static and default routes |

## Technologies and Concepts

* Cisco Packet Tracer
* TCP/IP
* OSI Model
* IPv4 Addressing
* CIDR / Subnetting
* VLANs
* DHCP
* DNS
* HTTP
* TCP/UDP
* Static Routing
* NAT/PAT
* Access Control Lists (ACLs)
* Wireless Networking
* LAN/WAN
* Network Security Fundamentals

## Project Files

```
College_management_Cisco_packet_Tracer/
├── College_Management_System.pkt   Cisco Packet Tracer project file
├── configs/                        Router and switch configuration files
├── docs/                           Design documentation (topology, IP plan, VLANs, routing, security)
├── web/                            College portal web page (HTML)
└── screenshots/                    Topology and verification screenshots
```

## How to Open the Project

1. Install or open Cisco Packet Tracer.
2. Download `College_Management_System.pkt` from this repository.
3. Open the file in Packet Tracer.
4. Review the topology, device configurations, and documentation in the
   `docs/` folder.
5. Use the testing commands described in `docs/troubleshooting.md` to
   verify connectivity, DHCP, DNS, and routing.

## Learning Outcomes

This project demonstrates practical application of core networking
skills, including structured IP addressing and subnetting, VLAN design
and inter-VLAN routing, configuring DHCP and DNS services, implementing
NAT for internet access, securing a network with VLAN segmentation and
access control lists, deploying a wireless network, connecting a remote
site over a WAN link, and using standard troubleshooting commands to
verify network behaviour.

