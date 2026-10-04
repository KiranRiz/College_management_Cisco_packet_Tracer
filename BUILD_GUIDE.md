# Build Guide — Assembling `College_Management_System.pkt`

## Why this guide exists

This repository was built in an automated environment that can write
files (configs, documentation, HTML) but **cannot drive the Cisco Packet
Tracer GUI** (there is no mouse/keyboard/drag-and-drop automation for
desktop applications available in that environment). Packet Tracer's
`.pkt` project file is also a proprietary, versioned binary format — not
something safe to hand-author outside the application, as a hand-built
file would risk being corrupt or simply refusing to open.

Rather than fake a `.pkt` file or fabricate screenshots of a topology that
was never actually built and tested, every piece of **real, verifiable**
work is included instead: the full IP/VLAN design, every router and
switch CLI configuration (copy-paste ready), the DNS/HTTP server
settings, the AP settings, and this exact build sequence. Following the
steps below in Packet Tracer 9.0 (already confirmed installed) produces
the finished, working `.pkt` file in well under an hour, after which you
can save it into this repository as `College_Management_System.pkt` and
take real screenshots for the `screenshots/` folder.

## Step 1 — Place the devices

Open Packet Tracer 9.0 and place the following, matching
[`docs/network-topology.md`](docs/network-topology.md):

| Device name     | Packet Tracer model                         | Category |
|-------------------|------------------------------------------------|------------|
| ISP-RTR           | Generic Router (e.g. 2811)                     | Network Devices → Routers |
| EXT-SRV           | Server-PT                                      | End Devices |
| R-EDGE            | Router 2911                                    | Network Devices → Routers |
| R-REMOTE          | Router 2901                                    | Network Devices → Routers |
| SW-CORE           | Multilayer Switch 3560-24PS                    | Network Devices → Switches |
| SW-ADMIN          | Switch 2960-24TT                               | Network Devices → Switches |
| SW-ACCOUNTS       | Switch 2960-24TT                               | Network Devices → Switches |
| SW-ACADEMIC       | Switch 2960-24TT                               | Network Devices → Switches |
| SW-STUDENT        | Switch 2960-24TT                               | Network Devices → Switches |
| SW-SERVER         | Switch 2960-24TT                               | Network Devices → Switches |
| SW-REMOTE         | Switch 2960-24TT                               | Network Devices → Switches |
| AP-WIRELESS       | AccessPoint-PT-N                               | Network Devices → Wireless Devices |
| SRV-COLLEGE       | Server-PT                                      | End Devices |
| PC-ADMIN1/2, PC-STUDENT1/2, PC-FACULTY1/2, PC-IT1, PC-LIBRARY1, PC-ACCOUNTS1, PC-MGMT1, PC-REMOTE1/2 | PC-PT | End Devices |
| LAPTOP-WIFI1/2    | Laptop-PT (with wireless NIC)                   | End Devices |

Rename every device to match the table (double-click the device → change
the display name) — this is what makes the topology self-explanatory in
screenshots and in an interview walkthrough.

## Step 2 — Cable everything

Use **copper straight-through** for PC/server/AP-to-switch links, and
**copper straight-through** for switch-to-switch/router Gigabit links
(Packet Tracer auto-detects crossover on modern IOS ports, so
straight-through cables work throughout). Follow the port numbers already
specified inside each config file's interface descriptions — e.g.
`SW-CORE.txt` expects its uplink to `R-EDGE` on `Gi0/1`, and its trunk to
`SW-ADMIN` on `Gi0/2`, etc. Cabling exactly to those port numbers means the
configs can be pasted in with zero editing.

For the WAN side: cable `R-EDGE ↔ ISP-RTR`, `ISP-RTR ↔ R-REMOTE`, and
`ISP-RTR ↔ EXT-SRV` — all copper straight-through (PT routers auto-detect).

## Step 3 — Apply the configurations

For each router/switch, open its **CLI** tab and either:
- type the commands manually from the matching file in
  [`configs/router-configs/`](configs/router-configs/) or
  [`configs/switch-configs/`](configs/switch-configs/), starting from
  global configuration mode (`enable` → `configure terminal`), **or**
- paste the whole file at once using the CLI tab's paste function
  (right-click → Paste in the terminal, after entering config mode).

After pasting each device's config, run `write memory` (or `copy
running-config startup-config`) to persist it.

> Enter commands exactly in the order given — some (like `crypto map`
> applied under an interface) depend on the crypto map already being
> defined further down the file; Packet Tracer will still accept this
> because the whole file is applied as one block, but if typing manually,
> define the `crypto isakmp`/`crypto ipsec`/`crypto map` blocks **before**
> applying `crypto map CMAP-VPN` under the interface.

## Step 4 — Configure the servers and AP (GUI-based, not CLI)

- `SRV-COLLEGE`: follow
  [`configs/SRV-COLLEGE-service-config.md`](configs/SRV-COLLEGE-service-config.md)
  exactly (IP config, DNS records, and uploading `web/index.html` as the
  HTTP root page).
- `EXT-SRV`: set IP `198.51.100.10 / 255.255.255.248`, gateway
  `198.51.100.9`; enable HTTP with the default PT page (or any simple
  "Internet Test Page" HTML) — it only needs to respond to prove NAT is
  working.
- `AP-WIRELESS`: follow
  [`configs/AP-WIRELESS-config.md`](configs/AP-WIRELESS-config.md) exactly
  (SSID, WPA2-PSK, VLAN 25 management IP).

## Step 5 — Configure PC/laptop IP settings

All wired PCs: `Desktop → IP Configuration → DHCP`. All wireless laptops:
connect to `COLLEGE-WIFI` with the pass phrase from the AP config doc,
then `DHCP`. No PC should need a manually-typed static IP.

## Step 6 — Validate (this is the part that matters most)

Work through [`docs/troubleshooting.md`](docs/troubleshooting.md) and the
**Testing & Verification** table in [`README.md`](README.md) one row at a
time. Do not consider the build finished until every row genuinely passes
in Simulation or Realtime mode — if something fails, the relevant
scenario in `troubleshooting.md` almost certainly covers the exact mistake
(VLAN not assigned, ACL not applied to an interface, NAT inside/outside
not marked, crypto map not applied, etc.).

## Step 7 — Save and capture screenshots

1. `File → Save As… → College_Management_System.pkt`, placed at the root
   of this repository.
2. Take screenshots for `screenshots/`:
   - Full logical topology view
   - `show vlan brief` on SW-CORE
   - `show ip route` on SW-CORE and R-EDGE
   - `show crypto isakmp sa` on R-EDGE (tunnel up)
   - The rendered `www.college.local` portal page in a PC's browser
   - A failed `ping` from a Student PC to an Accounts PC (ACL proof)
3. Commit and push both the `.pkt` file and the screenshots.

## Step 8 — Final sanity check before calling it "done"

Re-read the **VALIDATION** checklist in the original project brief (DHCP,
DNS, web server, VLANs, inter-VLAN routing, routing tables, NAT, wireless,
ACL/security, management access, no leftover config errors) and confirm
each item against what you actually observed in Packet Tracer — not
against what the documentation merely claims should happen.
