# IP Addressing Plan

## Summary

The entire main campus is carved out of the private address block
**10.10.0.0/16**, subnetted into /24s — one per VLAN/department. This gives
254 usable hosts per department (far more than needed today) with huge room
to grow, while keeping the addressing scheme trivial to read and explain:
the third octet of every address *is* the VLAN's department.

The remote satellite office and the simulated WAN/Internet use separate,
clearly-documented ranges so the three "zones" of the network (campus,
remote site, WAN) are never ambiguous when reading a `show ip route` table.

## Campus Department Subnets (from 10.10.0.0/16)

| VLAN | Department          | Network           | CIDR | Subnet Mask      | Usable Hosts | Default Gateway | Broadcast     | DHCP Range                  |
|------|----------------------|--------------------|------|-------------------|--------------|-------------------|---------------|-------------------------------|
| 10   | Administration       | 10.10.10.0         | /24  | 255.255.255.0     | 254          | 10.10.10.1        | 10.10.10.255  | 10.10.10.50 – 10.10.10.200    |
| 20   | Students             | 10.10.20.0         | /24  | 255.255.255.0     | 254          | 10.10.20.1        | 10.10.20.255  | 10.10.20.50 – 10.10.20.200    |
| 25   | Wireless Network     | 10.10.25.0         | /24  | 255.255.255.0     | 254          | 10.10.25.1        | 10.10.25.255  | 10.10.25.50 – 10.10.25.200    |
| 30   | Faculty              | 10.10.30.0         | /24  | 255.255.255.0     | 254          | 10.10.30.1        | 10.10.30.255  | 10.10.30.50 – 10.10.30.200    |
| 40   | IT Department        | 10.10.40.0         | /24  | 255.255.255.0     | 254          | 10.10.40.1        | 10.10.40.255  | 10.10.40.50 – 10.10.40.200    |
| 50   | Library              | 10.10.50.0         | /24  | 255.255.255.0     | 254          | 10.10.50.1        | 10.10.50.255  | 10.10.50.50 – 10.10.50.200    |
| 60   | Accounts             | 10.10.60.0         | /24  | 255.255.255.0     | 254          | 10.10.60.1        | 10.10.60.255  | 10.10.60.50 – 10.10.60.200    |
| 70   | Management/Principal | 10.10.70.0         | /24  | 255.255.255.0     | 254          | 10.10.70.1        | 10.10.70.255  | 10.10.70.50 – 10.10.70.200    |
| 80   | Servers              | 10.10.80.0         | /24  | 255.255.255.0     | 254          | 10.10.80.1        | 10.10.80.255  | **Static only — no DHCP**    |
| 99   | Network Management   | 10.10.99.0         | /24  | 255.255.255.0     | 254          | 10.10.99.1        | 10.10.99.255  | **Static only — no DHCP**    |

Addressing convention inside every department subnet:

- `.1` — default gateway (SVI on SW-CORE)
- `.2 – .9` — reserved for static infrastructure (printers, AP management IP, etc.)
- `.10 – .49` — reserved for statically-addressed devices (servers in VLAN 80)
- `.50 – .200` — DHCP pool for end-user devices
- `.201 – .254` — reserved for future static assignments

## Static Infrastructure Addresses

| Device                     | Interface        | IP Address      | VLAN/Network         |
|-----------------------------|-------------------|-------------------|------------------------|
| SRV-COLLEGE (Web + DNS)     | FastEthernet0     | 10.10.80.10/24    | VLAN 80 Servers        |
| SW-CORE                     | Vlan99 (mgmt SVI) | 10.10.99.2/24     | VLAN 99 Net Mgmt       |
| SW-ADMIN                    | Vlan99 (mgmt SVI) | 10.10.99.11/24    | VLAN 99 Net Mgmt       |
| SW-ACCOUNTS                 | Vlan99 (mgmt SVI) | 10.10.99.12/24    | VLAN 99 Net Mgmt       |
| SW-ACADEMIC                 | Vlan99 (mgmt SVI) | 10.10.99.13/24    | VLAN 99 Net Mgmt       |
| SW-STUDENT                  | Vlan99 (mgmt SVI) | 10.10.99.14/24    | VLAN 99 Net Mgmt       |
| SW-SERVER                   | Vlan99 (mgmt SVI) | 10.10.99.15/24    | VLAN 99 Net Mgmt       |
| AP-WIRELESS                 | Management IP     | 10.10.25.5/24     | VLAN 25 Wireless       |

## Transit / Point-to-Point Links (not VLANs — routed links)

| Link                              | Network            | CIDR | Device A              | Device B                  |
|------------------------------------|----------------------|------|------------------------|-----------------------------|
| SW-CORE ↔ R-EDGE                  | 10.10.1.0/30         | /30  | SW-CORE .1             | R-EDGE Gi0/0 .2            |
| R-EDGE ↔ ISP-RTR (campus WAN)     | 198.51.100.0/30      | /30  | R-EDGE Gi0/1 .2        | ISP-RTR Gi0/0 .1           |
| ISP-RTR ↔ R-REMOTE (remote WAN)   | 198.51.100.4/30      | /30  | ISP-RTR Gi0/1 .5       | R-REMOTE Gi0/0 .6          |
| ISP-RTR ↔ EXT-SRV (simulated internet host) | 198.51.100.8/29 | /29 | ISP-RTR Gi0/2 .9       | EXT-SRV .10                |

> `198.51.100.0/24` is the IANA **TEST-NET-2** block reserved by RFC 5737
> specifically for documentation and lab examples. It is used here instead
> of a real public IP range so the WAN side of this project can never be
> confused with — or accidentally routed toward — a real internet address.

## Remote / Satellite Office (separate site, reached via site-to-site VPN)

| VLAN/Network | Purpose                 | Network        | CIDR | Default Gateway |
|---------------|--------------------------|------------------|------|-------------------|
| N/A (flat)    | Remote Admin Office LAN | 10.20.10.0/24    | /24  | 10.20.10.1        |

The remote site intentionally uses a **10.20.x.x** range, clearly outside the
10.10.x.x campus block, so that a glance at any routing table immediately
tells you whether a destination is on the main campus or at the remote site.

## Subnetting Rationale (CIDR explanation)

- The campus could have been issued a single flat /16, but a **flat network
  with 9+ departments would mean one giant broadcast domain**, no
  containment of faults or broadcast storms, and no way to apply
  department-specific security policy. Subnetting into /24s per VLAN solves
  all three.
- A /24 per department (254 usable hosts) is deliberately generous for a
  "medium-sized college" department (typically 20–100 devices per
  department) — this leaves headroom for growth without having to
  re-subnet later, while still being small enough that `/24` summarizes
  cleanly in every ACL and route statement in this project.
- `/30` is used for every router-to-router point-to-point link, since a
  point-to-point link only ever needs 2 usable host addresses — using
  anything larger would waste address space for no benefit.
