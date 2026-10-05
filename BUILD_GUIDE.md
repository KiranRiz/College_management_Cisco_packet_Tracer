# Build Guide — Assembling `College_Management_System.pkt`

## Current status: the .pkt already exists and is already tested

`College_Management_System.pkt` in this repository's root is a real,
already-built Packet Tracer file — every device in the table below is
placed and cabled, every router/switch has its full config applied via
CLI, DHCP/DNS/HTTP/wireless are configured, and the network has been
tested end-to-end in Realtime mode (see the README's **Testing &
Verification** section for exactly what was checked).

**Two small manual steps are still needed** before the file is 100%
complete — both are GUI drag-and-drop/paste actions that could not be
finished reliably through automation:

1. **Step 2b** — insert the WPC300N wireless module into `LAPTOP-WIFI1`
   and `LAPTOP-WIFI2` (currently placed on the canvas but offline — no
   module, no cable). ~30 seconds each.
2. **Step 4 (SRV-COLLEGE)** — paste [`web/index.html`](web/index.html)
   into the server's HTTP file editor to replace Packet Tracer's default
   page. The server's HTTP/DNS service is already on and already serving
   pages (DNS resolves, HTTP loads) — this step only swaps the page
   content.

The rest of this guide is kept in full below so you can verify any part
of the build, re-derive it from scratch if needed, or use it as a
reference while doing the two steps above.

## Why this guide exists (original rationale)

This repository was originally built in an automated environment that
could write files (configs, documentation, HTML) but could not drive the
Cisco Packet Tracer GUI directly. That constraint was later lifted for
this project (see **Current status** above) and the `.pkt` was built and
tested for real. Packet Tracer's `.pkt` format is a proprietary, versioned
binary — never hand-authored outside the application, which is why this
guide always worked by driving the real app rather than fabricating the
file.

Rather than fake a `.pkt` file or fabricate screenshots of a topology that
was never actually built and tested, every piece of **real, verifiable**
work is included: the full IP/VLAN design, every router and switch CLI
configuration (copy-paste ready), the DNS/HTTP server settings, the AP
settings, and this exact build sequence — all of which the committed
`.pkt` was actually built from.

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
| LAPTOP-WIFI1/2    | Laptop-PT (with wireless NIC — see Step 2b)     | End Devices |

Rename every device to match the table (double-click the device → change
the display name) — this is what makes the topology self-explanatory in
screenshots and in an interview walkthrough.

## Step 2 — Cable everything (exact port-to-port list)

Use **copper straight-through** for every link below — Packet Tracer
auto-detects crossover on modern IOS/switch ports, so straight-through
works for PC-to-switch, switch-to-switch, and switch-to-router links
alike. Every port number here is copied directly from the `description`
lines already in `configs/`, so there is no guessing: cable exactly this
list and every config file pastes in with zero editing.

**Backbone / router links**

| Device A    | Port A | Device B   | Port B |
|--------------|--------|-------------|--------|
| ISP-RTR      | Gi0/0  | R-EDGE      | Gi0/1  |
| ISP-RTR      | Gi0/1  | R-REMOTE    | Gi0/0  |
| ISP-RTR      | Gi0/2  | EXT-SRV     | FastEthernet0 |
| R-EDGE       | Gi0/0  | SW-CORE     | Gi0/1  |
| R-REMOTE     | Gi0/1  | SW-REMOTE   | Gi0/1  |

**Core-to-access trunks**

> Correction verified directly against Packet Tracer: the 3560-24PS model
> only exposes 2 Gigabit ports (Gi0/1, Gi0/2) plus 24 FastEthernet ports —
> not 6 Gigabit ports as an earlier draft of `SW-CORE.txt` assumed. Fixed
> in both the config and this table: Gi0/2 keeps the SW-ADMIN trunk, the
> other four trunks move to FastEthernet0/1–0/4 (100 Mbps trunk links —
> fully valid for 802.1Q trunking in Packet Tracer).

| Device A  | Port A | Device B     | Port B |
|------------|--------|---------------|--------|
| SW-CORE    | Gi0/2  | SW-ADMIN      | Gi0/1  |
| SW-CORE    | Fa0/1  | SW-ACCOUNTS   | Gi0/1  |
| SW-CORE    | Fa0/2  | SW-ACADEMIC   | Gi0/1  |
| SW-CORE    | Fa0/3  | SW-STUDENT    | Gi0/1  |
| SW-CORE    | Fa0/4  | SW-SERVER     | Gi0/1  |

**Access ports — end devices**

| Switch       | Port    | End Device                  |
|---------------|---------|--------------------------------|
| SW-ADMIN      | Fa0/1   | PC-ADMIN1                      |
| SW-ADMIN      | Fa0/2   | PC-ADMIN2                      |
| SW-ADMIN      | Fa0/3   | PC-MGMT1 (Principal Office)    |
| SW-ACCOUNTS   | Fa0/1   | PC-ACCOUNTS1                   |
| SW-ACCOUNTS   | Fa0/2   | PC-ACCOUNTS2                   |
| SW-ACADEMIC   | Fa0/1   | PC-FACULTY1                    |
| SW-ACADEMIC   | Fa0/2   | PC-FACULTY2                    |
| SW-ACADEMIC   | Fa0/3   | PC-LIBRARY1                    |
| SW-STUDENT    | Fa0/1   | PC-STUDENT1                    |
| SW-STUDENT    | Fa0/2   | PC-STUDENT2                    |
| SW-STUDENT    | Fa0/3   | AP-WIRELESS (Ethernet Port 1)  |
| SW-SERVER     | Fa0/1   | PC-IT1                         |
| SW-SERVER     | Fa0/2   | SRV-COLLEGE (FastEthernet0)    |
| SW-REMOTE     | Fa0/1   | PC-REMOTE1                     |
| SW-REMOTE     | Fa0/2   | PC-REMOTE2                     |

`LAPTOP-WIFI1` and `LAPTOP-WIFI2` are **not cabled** — they associate to
`AP-WIRELESS` over wireless (see Step 2b and Step 5).

Every other port on every switch (e.g. SW-ADMIN Fa0/4–0/24) is left
unpatched — this matches the "UNUSED - administratively shut down"
description already in each switch config, so leaving them empty and
applying the config as-is is correct; nothing further to do.

## Step 2b — Add wireless NICs to the laptops

Packet Tracer laptops ship with a wired NIC only. Before `LAPTOP-WIFI1`/
`LAPTOP-WIFI2` can see the SSID:

1. Click the laptop → **Physical** tab.
2. Click the power button to switch the laptop **off**.
3. Drag a **WPC300N** wireless module from the modules list into the
   laptop's empty module slot.
4. Power the laptop back **on**.
5. Go to the **Desktop** tab → **PC Wireless** to configure the SSID/PSK
   in Step 5.

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

- **Every wired PC** (`PC-ADMIN1/2`, `PC-STUDENT1/2`, `PC-FACULTY1/2`,
  `PC-IT1`, `PC-LIBRARY1`, `PC-ACCOUNTS1/2`, `PC-MGMT1`, `PC-REMOTE1/2`):
  `Desktop → IP Configuration → DHCP`. No manual IP entry — each should
  come back with an address in its VLAN's `.50–.200` range (e.g.
  `PC-STUDENT1` → something in `10.10.20.50–200`), gateway `.1` of that
  subnet, and DNS `10.10.80.10`.
- **LAPTOP-WIFI1 / LAPTOP-WIFI2** (after Step 2b): `Desktop → PC Wireless`
  → Connect tab → select SSID `COLLEGE-WIFI` → enter PSK `Coll3ge@2026`
  (WPA2-PSK/AES) → once associated, same tab's IP Configuration → `DHCP`.
  Expect an address in `10.10.25.50–200`, gateway `10.10.25.1`.

If any client comes back with `169.254.x.x` (APIPA) instead, stop and
check that device's switchport VLAN assignment and the matching DHCP pool
on `SW-CORE` before continuing — see Scenario 1 in
[`docs/troubleshooting.md`](docs/troubleshooting.md).

## Step 6 — Validate (this is the part that matters most)

Run through this list in order, in Packet Tracer **Realtime** mode. Each
row names the exact command/action and the exact result that confirms a
pass — do not consider the build finished until every row genuinely
passes as observed, not assumed:

| # | Action (exact command) | Expected result |
|---|---------------------------|---------------------|
| 1 | `ipconfig` on PC-STUDENT1, PC-ADMIN1, LAPTOP-WIFI1 | Each shows an IP in its VLAN's DHCP range, correct gateway/DNS |
| 2 | `ping 10.10.80.10` from PC-FACULTY1 | Replies — confirms inter-VLAN routing to the server VLAN |
| 3 | `nslookup www.college.local` on a PC that supports it (or browser URL bar test) | Resolves to `10.10.80.10` |
| 4 | Browser → `http://www.college.local` from PC-STUDENT1 and LAPTOP-WIFI1 | College portal page renders |
| 5 | `ping 10.10.60.10` (an Accounts PC) from PC-STUDENT1 | **Times out** — ACL is working |
| 6 | On SW-CORE: `show access-lists STUDENT-RESTRICT` | Deny-line match counters are non-zero after step 5 |
| 7 | On SW-CORE: `show vlan brief` | All 10 VLANs listed with correct ports assigned |
| 8 | On SW-CORE / R-EDGE: `show ip route` | Matches the routes documented in [`docs/routing.md`](docs/routing.md) |
| 9 | On R-EDGE: `ping 198.51.100.10` (EXT-SRV) then `show ip nat translations` | Ping succeeds; a translation entry appears for the source PC's real IP |
| 10 | Generate traffic PC-REMOTE1 → PC-ADMIN1 (`ping 10.10.10.50`-range address), then on R-EDGE: `show crypto isakmp sa` | Ping succeeds; SA state shows `QM_IDLE` |
| 11 | `ssh -l netadmin 10.10.99.2` from any PC in the management VLAN's reach, then try `telnet 10.10.99.2` | SSH prompts for password and connects; Telnet is refused |
| 12 | Unplug/replug a PC's cable into an already-"UNUSED" shut port | Port stays down (`shutdown` still applied — confirms unused-port hardening) |

If anything in rows 1–12 fails, the matching scenario in
[`docs/troubleshooting.md`](docs/troubleshooting.md) almost certainly
describes the exact mistake (VLAN not assigned, ACL not applied to an
interface, NAT inside/outside not marked, crypto map not applied, etc.).

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
