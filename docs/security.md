# Security Design

This document covers every security control implemented in the project:
what it is, what it protects, where it's configured, and — honestly —
what its limitations are inside Packet Tracer.

## 1. Device Access Hardening (all routers and switches)

| Control                          | Purpose                                                            |
|------------------------------------|----------------------------------------------------------------------|
| `enable secret` (hashed, not `enable password`) | Privileged-mode password is stored as a hash, not plaintext, in the config |
| `service password-encryption`     | Obfuscates remaining plaintext passwords (line/VTY) in `show running-config` |
| Local user database + SSH only    | `line vty` accepts `transport input ssh` only — Telnet is explicitly disabled |
| `ip domain-name` + `crypto key generate rsa` | Required to enable SSH on IOS devices |
| Console password + `login`        | Physical/console access still requires authentication              |
| `banner motd`                     | Legal warning banner displayed before login ("Unauthorized access is prohibited...") |
| Management VLAN 99                | All switch/SVI management interfaces live on a VLAN carrying no end-user traffic |

**Why SSH instead of Telnet:** Telnet sends credentials and all session
traffic in clear text. Any device on a shared segment (or anyone who later
captures traffic, see [wireshark-analysis.md](wireshark-analysis.md)) could
read admin passwords in plaintext. SSH encrypts the session.

## 2. Switch Port Security

Applied to every access port facing an end-user device (PCs, laptops, the
AP uplink):

```
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict
switchport port-security mac-address sticky
```

- **Maximum 2** allows a PC plus, e.g., a VoIP phone or a docked laptop,
  without allowing someone to plug an unauthorized switch/hub into a wall
  jack and attach many devices.
- **Violation restrict** (rather than `shutdown`) drops offending traffic
  and logs it, without taking the whole port down — a deliberate choice so
  a single misbehaving device doesn't cause a full helpdesk outage for a
  shared lab port, while the violation is still visible in `show port-security`.
- **Sticky MAC** learns the first connected MAC(s) automatically rather than
  requiring every device's MAC to be hand-entered — practical for a
  101+-PC college deployment.

## 3. Unused Port Hardening

Every switchport not patched to a live device is:

```
switchport access vlan 999
switchport mode access
shutdown
```

VLAN 999 is a dead-end VLAN with no SVI and no routing anywhere in the
network — even if a port is physically patched in by mistake or by a bad
actor and accidentally re-enabled, it lands in a VLAN that goes nowhere.

## 4. VLAN / Trunk Hardening

- **Native VLAN changed to 99** on every trunk (`switchport trunk native
  vlan 99`), instead of the default VLAN 1 — defeats the classic "double
  tagging" VLAN-hopping attack, which relies on the native VLAN matching
  VLAN 1 on both ends of a trunk.
- **`switchport nonegotiate`** on every trunk disables DTP (Dynamic
  Trunking Protocol) negotiation — an attacker's PC can no longer ask a
  switchport to "become" a trunk port and gain access to every VLAN.
- **`switchport trunk allowed vlan`** explicitly prunes each trunk to only
  the VLANs actually needed on that link, reducing unnecessary broadcast
  propagation and attack surface.

## 5. Spanning-Tree Hardening

- `spanning-tree portfast` on all end-device access ports — PCs/servers
  come straight up without waiting through STP's listening/learning delay.
- `spanning-tree bpduguard enable` on the same ports — if a port that
  should only ever see a PC suddenly receives a BPDU (i.e. someone plugged
  in a rogue switch), the port is immediately error-disabled instead of
  being allowed to participate in the spanning tree.
- `SW-CORE` is set as the root bridge (lowest priority) for all VLANs,
  since it is the one device that should always be the center of the
  topology.

## 6. Extended ACL — Student/Wireless Containment (verified working, with a platform correction)

**Requirement:** Students (wired, VLAN 20, and wireless, VLAN 25) must be
able to reach the DNS server and the college web portal, and the Internet
(via NAT), but must **not** be able to reach the Accounts (VLAN 60) or
Management/Principal (VLAN 70) subnets.

**Original design, and what was actually found when testing it:** the ACL
was first applied inbound on `SW-CORE`'s `Vlan20`/`Vlan25` SVIs (the
textbook-correct location for this policy on a Layer-3 switch). Tested
directly in Packet Tracer 9.0: the command
(`ip access-group STUDENT-RESTRICT in` under `interface Vlan20`) is
accepted with no error — but never actually takes effect. Confirmed three
independent ways before concluding this was a real platform limitation and
not a mistake:

1. `show ip interface vlan20` continued to report `Inbound access list is
   not set` immediately after applying it (and after `write memory`).
2. `show running-config | section Vlan20` confirmed the interface's saved
   config genuinely has no `ip access-group` line.
3. A live `ping` from a Student PC to an Accounts PC still succeeded
   (3 of 4 replies) — the deny rule was not being enforced in practice.

This is a limitation of the simulated Catalyst 3560 SVI implementation in
this Packet Tracer build, not a configuration error — the identical
`ip access-group` syntax is standard, correct IOS.

**What actually works, and is what's deployed:** the same ACL logic
applied as a **port ACL (PACL)** on the Layer-2 access ports of
`SW-STUDENT` (a 2960) — `FastEthernet0/1` and `FastEthernet0/2` (the wired
student PCs) and `FastEthernet0/3` (the AP-WIRELESS uplink, covering
wireless clients). This was verified the same rigorous way:

1. `show running-config | begin FastEthernet0/1` confirmed
   `ip access-group STUDENT-RESTRICT in` genuinely saved under the port.
2. A live `ping` from PC-STUDENT1 to an Accounts PC afterward returned
   `Destination host unreachable` for all 4 packets (100% loss) — blocked.
3. A live `ping` and browser test from PC-STUDENT1 to the web/DNS server
   (`10.10.80.10`) succeeded — explicitly permitted traffic still works.

```
ip access-list extended STUDENT-RESTRICT
 remark Deny Students/Wireless -> sensitive Accounts & Management subnets
 deny   ip any 10.10.60.0 0.0.0.255
 deny   ip any 10.10.70.0 0.0.0.255
 remark Explicitly permit required services to the server VLAN
 permit udp any host 10.10.80.10 eq domain
 permit tcp any host 10.10.80.10 eq www
 permit icmp any host 10.10.80.10
 remark Permit everything else (e.g. Internet access via NAT at R-EDGE)
 permit ip any any
```

Applied under `interface FastEthernet0/1`, `0/2`, and `0/3` on
`SW-STUDENT`. Full configuration:
[`configs/switch-configs/SW-STUDENT.txt`](../configs/switch-configs/SW-STUDENT.txt).
A reference-only copy of the original SVI-targeted ACL (not applied to any
interface) is kept in
[`configs/switch-configs/SW-CORE.txt`](../configs/switch-configs/SW-CORE.txt)
with a note explaining why it was moved.

**Why this order:** ACLs are processed top-down, first match wins. The
specific `deny` statements for Accounts/Management must come *before* the
general `permit ip any any`, otherwise the explicit denies would never be
reached.

**Trade-off of the port-ACL approach:** since the rule now lives per-port
on the access switch instead of centrally on the core switch, every new
student-facing access port added in the future needs the same
`ip access-group STUDENT-RESTRICT in` line applied to it explicitly — it
is not automatically inherited the way a single SVI-level policy would
have been. For a network this size (one student access switch), that's a
minor, clearly-documented maintenance note rather than a real limitation.

## 7. NAT / PAT (see also [routing.md](routing.md))

`R-EDGE` performs NAT overload (PAT), translating the entire
`10.10.0.0/16` campus range to its single WAN IP when talking to the
Internet (`EXT-SRV`), hiding internal addressing from the outside world.
Traffic destined for the Remote Office over the VPN is explicitly
**excluded** from NAT (NAT exemption), since IPsec must encrypt the
original private addresses end-to-end for routing to work correctly at the
remote site.

## 8. Firewall — Perimeter ACL (verified platform limitation on CBAC)

**What was attempted:** An IOS CBAC (Context-Based Access Control)
stateful firewall was originally designed for `R-EDGE`'s WAN-facing
interface:

```
ip inspect name FW-INSPECT tcp
ip inspect name FW-INSPECT udp
ip inspect name FW-INSPECT icmp
!
interface GigabitEthernet0/1
 description WAN link to ISP
 ip access-group WAN-IN in
 ip inspect FW-INSPECT out
```

**What actually happened (verified directly on the device, not assumed):**
every `ip inspect` command was rejected on `R-EDGE` with
`% Unrecognized command` / `% Invalid input detected`. Packet Tracer
9.0's simulated IOS for the ISR platform used here does not implement
CBAC at all. Per this project's standard, that's not faked — no
`ip inspect` lines appear in
[`configs/router-configs/R-EDGE.txt`](../configs/router-configs/R-EDGE.txt).

**What's actually implemented instead:** the extended ACL `WAN-IN`,
applied inbound on `R-EDGE`'s WAN interface, genuinely is in the running
config and genuinely works — it default-denies all unsolicited inbound
traffic from the Internet while explicitly permitting the ICMP replies
needed for outbound troubleshooting (ping/traceroute from inside still
work normally, since TCP/UDP sessions *initiated from inside* get their
replies back regardless of this ACL — only *inbound-initiated* connections
are blocked). This is a real, verified, stateless packet filter — the
firewall "default-deny inbound" concept is demonstrated, just without
CBAC's stateful session tracking on top of it.

**What it protects:** The campus LAN from unsolicited inbound connections
originating on the Internet.

**Limitations (documented honestly):**
- This is **stateless** filtering (fixed rules only) — not the stateful,
  connection-aware inspection CBAC or a real firewall appliance would
  provide. It cannot tell "this inbound TCP packet is part of a session I
  already allowed out" from "this is a brand-new inbound TCP SYN" the way
  a stateful firewall can; it only matches on the fields explicitly
  written into the ACL.
- Deep packet inspection, application-layer filtering, and intrusion
  prevention are out of scope for both the attempted CBAC approach and
  this ACL-based fallback.
- A real deployment would use **Zone-Based Policy Firewall (ZFW)** or a
  dedicated appliance (e.g. Cisco ASA — Packet Tracer does ship an
  ASA5505 model; it wasn't substituted in here, to keep the verification
  findings in this section specific to the originally-designed platform
  rather than changing the design to dodge the limitation).
- This is a **concept demonstration appropriate for a student portfolio**,
  not a production-grade firewall design.

## 9. Wireless Security

- SSID: `COLLEGE-WIFI`, broadcast disabled is **not** used (a hidden SSID
  provides negligible real security and breaks normal client roaming — a
  conscious decision, not an oversight).
- Authentication: **WPA2-PSK (AES)**. WEP and open authentication were
  deliberately avoided as they are not real security.
- Pre-shared key used in this lab: `Coll3ge@2026` — clearly a lab/demo
  credential for this Packet Tracer file only, and called out here so it
  is never mistaken for a production secret.

## 10. VPN

See [vpn.md](vpn.md) for the full site-to-site IPsec design between
`R-EDGE` and `R-REMOTE`.

## 11. Monitoring

- `logging buffered 16384 informational` on all routers/switches — keeps a
  local log buffer of security and operational events (e.g. port-security
  violations, interface up/down, ACL deny logging) viewable with `show
  logging`.
- `snmp-server community COLLEGE-RO RO` configured as a read-only
  monitoring community — representing how a real NMS (e.g. SolarWinds,
  LibreNMS, PRTG) would poll device health in production. No SNMP
  manager/server is included in the topology itself — this is
  configuration-level readiness for monitoring, documented honestly as
  such rather than claiming a full NMS was deployed.
