# VLAN Design

## Design Philosophy

The college network is segmented using VLANs so that each department's broadcast
traffic, security policy, and troubleshooting scope stay isolated from the
others, while still allowing controlled inter-VLAN routing for the services
every department legitimately needs (DNS, the web portal, internet access).

The brief's suggested VLAN plan (10/20/30/40/50/60/70/80/99) is used as the
base, with two professional additions:

- **VLAN 25 — Wireless Network**: wireless clients (student and faculty
  laptops/phones) are placed in their own VLAN rather than bridged into the
  wired Student or Faculty VLANs. This is standard enterprise practice —
  wireless media is inherently less trusted than a wall-jack in a locked
  office, so it gets its own policy and its own troubleshooting domain.
- **VLAN 999 — Unused/Parking**: every switchport that is not actively
  patched to a device is administratively shut down **and** moved into this
  VLAN. If a port is ever accidentally re-enabled, it lands in a VLAN with no
  routing and no useful access — not back on a live department VLAN.

## VLAN Table

| VLAN ID | Name              | Department / Purpose                  | Subnet           | Switch(es)                  |
|---------|-------------------|----------------------------------------|-------------------|------------------------------|
| 10      | VLAN10_ADMIN      | Administration office                  | 10.10.10.0/24     | SW-ADMIN                    |
| 20      | VLAN20_STUDENTS   | Student computer labs (wired)          | 10.10.20.0/24     | SW-STUDENT                  |
| 25      | VLAN25_WIRELESS   | Wireless network (students & faculty)  | 10.10.25.0/24     | SW-STUDENT (AP uplink)      |
| 30      | VLAN30_FACULTY    | Faculty / teaching staff               | 10.10.30.0/24     | SW-ACADEMIC                 |
| 40      | VLAN40_IT         | IT department / helpdesk               | 10.10.40.0/24     | SW-SERVER                   |
| 50      | VLAN50_LIBRARY    | Library                                | 10.10.50.0/24     | SW-ACADEMIC                 |
| 60      | VLAN60_ACCOUNTS   | Accounts / finance (sensitive)         | 10.10.60.0/24     | SW-ACCOUNTS                 |
| 70      | VLAN70_MGMT       | Management / Principal's office        | 10.10.70.0/24     | SW-ADMIN                    |
| 80      | VLAN80_SERVERS    | Server room (Web, DNS)                 | 10.10.80.0/24     | SW-SERVER                   |
| 99      | VLAN99_NETMGMT    | Network device management (SVIs, SSH)  | 10.10.99.0/24     | All switches (native VLAN)  |
| 999     | VLAN999_UNUSED    | Parking lot for disabled/unused ports  | Not routed        | All switches                |

## Why departments are grouped onto shared access switches

A real medium-sized college does not put one switch per department — it
shares switches between departments that sit physically close together
(same building/floor), while keeping their traffic logically separated by
VLAN:

| Switch       | VLANs carried      | Reasoning                                                        |
|--------------|---------------------|-------------------------------------------------------------------|
| SW-ADMIN     | 10, 70              | Admin office and Principal/Management office share the same admin block |
| SW-ACCOUNTS  | 60                  | Kept on its own switch because Accounts holds the most sensitive data — smaller blast radius if the switch is ever compromised |
| SW-ACADEMIC  | 30, 50              | Faculty staff room and the Library are in the same academic block |
| SW-STUDENT   | 20, 25 (AP uplink)  | Student labs, plus the wireless AP that serves student/faculty Wi-Fi |
| SW-SERVER    | 40, 80              | IT department sits next to the server room it administers        |

Each access switch trunks a **single uplink** to the Layer-3 core switch
(SW-CORE), which performs all inter-VLAN routing. This keeps the design a
clean, easily-explained collapsed-core/distribution-free topology —
appropriate for a single-building, medium-sized college and easy to justify
in an interview without over-engineering a design that would normally only
suit a much larger multi-building campus.

## Trunking

- All switch-to-switch and switch-to-router links are configured as **802.1Q
  trunks**, carrying only the VLANs actually present on that switch (pruned
  with `switchport trunk allowed vlan`) plus VLAN 99 for management.
- The trunk **native VLAN is changed to 99** (not the default VLAN 1) on every
  trunk, and `switchport nonegotiate` disables DTP, which closes off the
  classic VLAN-hopping attack vectors (double-tagging via VLAN 1, and rogue
  DTP negotiation).

## Inter-VLAN routing

Inter-VLAN routing is performed centrally on **SW-CORE**, a Layer-3
multilayer switch, using one SVI (switched virtual interface) per VLAN. See
[routing.md](routing.md) for full details and justification of this design
choice over router-on-a-stick.
