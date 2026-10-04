# Network Topology

## Logical Diagram (ASCII)

```
                                   ┌───────────────────────┐
                                   │   EXT-SRV (simulated   │
                                   │   internet host)       │
                                   │   198.51.100.10        │
                                   └───────────┬────────────┘
                                               │ 198.51.100.8/29
                                   ┌───────────┴────────────┐
                                   │        ISP-RTR         │
                                   │  (simulated Internet)  │
                                   └──────┬───────────┬─────┘
                        198.51.100.0/30   │           │   198.51.100.4/30
                                   ┌───────┴──────┐ ┌──┴─────────────┐
                                   │    R-EDGE     │ │    R-REMOTE     │
                                   │ (Border / NAT │ │ (Remote Admin   │
                                   │ / Firewall /  │ │  Office branch  │
                                   │ VPN peer)     │ │  router)        │
                                   └───────┬───────┘ └────────┬────────┘
                               10.10.1.0/30│                   │ 10.20.10.0/24
                                   ┌───────┴───────┐   ┌────────┴────────┐
                                   │    SW-CORE     │   │   SW-REMOTE     │
                                   │ (L3 — inter-   │   │  (L2 access)    │
                                   │  VLAN routing, │   └───┬────────┬───┘
                                   │  DHCP, ACLs)   │     PC-REMOTE1 PC-REMOTE2
                                   └───┬───┬───┬───┬┘
              Trunk (VLAN 99 native)  │   │   │   │
            ┌───────────────────────┐│   │   │   │┌───────────────────────┐
            │                        │   │   │   ││                        │
      ┌─────┴─────┐            ┌─────┴┐  │  ┌┴────┴┐              ┌────────┴──┐
      │ SW-ADMIN  │            │SW-   │  │  │SW-   │              │ SW-SERVER │
      │ VLAN10,70 │            │ACCOUNTS│ │ │ACADEMIC│            │ VLAN40,80 │
      └─┬───────┬─┘            │VLAN60│  │  │VLAN30,50│            └───┬───┬───┘
        │       │              └──┬───┘  │  └──┬───┬──┘                │   │
   PC-ADMIN1 PC-MGMT1         PC-ACCOUNTS1│  PC-FACULTY1 PC-LIBRARY1  PC-IT1 SRV-COLLEGE
   PC-ADMIN2 (Principal)                  │  PC-FACULTY2              (10.10.80.10
                                     ┌─────┴──────┐                    Web+DNS)
                                     │ SW-STUDENT │
                                     │ VLAN20, 25 │
                                     └─┬────────┬─┘
                                  PC-STUDENT1  AP-WIRELESS (VLAN 25)
                                  PC-STUDENT2    │        │
                                           LAPTOP-WIFI1 LAPTOP-WIFI2
```

## Sites

### 1. Main Campus
The primary site. Contains the border router, the Layer-3 core switch, five
department access switches, the server room, and a wireless access point
serving student/faculty Wi-Fi clients.

### 2. Remote Admin Office (satellite site)
A small branch office (e.g. an off-site admissions/admin office) connected
back to the main campus over a site-to-site IPsec VPN tunnel across the
simulated WAN — see [vpn.md](vpn.md). Demonstrates that this design is not
just a single-building LAN exercise but a genuine multi-site WAN/VPN
scenario.

### 3. Simulated Internet (ISP-RTR + EXT-SRV)
`ISP-RTR` is a plain router used purely to simulate the Internet core — it
has no special college-specific configuration. `EXT-SRV` is a server
representing a random host out on the Internet (e.g. a public website),
used to prove that NAT/PAT on `R-EDGE` is correctly translating campus
private addresses before they reach the "outside world".

## Device Role Summary

| Device         | Role                                                                 |
|-----------------|-----------------------------------------------------------------------|
| ISP-RTR         | Simulated Internet / WAN core                                        |
| EXT-SRV         | Simulated external internet host (for NAT verification)              |
| R-EDGE          | Campus border router — NAT/PAT, IOS firewall (CBAC), VPN peer, default routing |
| SW-CORE         | Layer-3 multilayer switch — inter-VLAN routing (SVIs), DHCP server, ACL enforcement |
| SW-ADMIN        | Access switch — Administration (VLAN 10) + Management/Principal (VLAN 70) |
| SW-ACCOUNTS     | Access switch — Accounts (VLAN 60), isolated on its own switch given data sensitivity |
| SW-ACADEMIC     | Access switch — Faculty (VLAN 30) + Library (VLAN 50)                |
| SW-STUDENT      | Access switch — Students (VLAN 20) + uplink to wireless AP (VLAN 25) |
| SW-SERVER       | Access switch — IT department (VLAN 40) + Server room (VLAN 80)      |
| AP-WIRELESS     | Wireless access point — SSID `COLLEGE-WIFI`, WPA2-PSK, VLAN 25       |
| SRV-COLLEGE     | Server hosting the HTTP college portal and the DNS zone `college.local` |
| R-REMOTE        | Remote office branch router — VPN peer, default route                |
| SW-REMOTE       | Remote office access switch                                          |

## Why this topology

- **Collapsed core, single building** — a medium college does not need a
  separate core/distribution/access hierarchy with redundant links; one
  Layer-3 core switch doing inter-VLAN routing for five access switches is
  the right amount of complexity, matches real small-campus designs, and is
  easy to defend in an interview.
- **Edge router separate from the core switch** — NAT, the IOS firewall
  feature set (CBAC), and IPsec VPN termination are kept on a dedicated
  border router (`R-EDGE`) rather than loaded onto the core switch, mirroring
  real designs where the "WAN edge" function is a distinct device/policy
  boundary from "LAN distribution".
- **Accounts isolated on its own access switch** — even though it would fit
  on `SW-ADMIN` physically, Accounts holds the most sensitive data in the
  college, so it is kept on a separate switch and is the specific target of
  the Student-VLAN ACL restriction (see [security.md](security.md)).
- **Remote site + VPN** — included specifically to demonstrate genuine
  site-to-site IPsec VPN functionality that Packet Tracer actually supports,
  rather than only describing VPN theory.
