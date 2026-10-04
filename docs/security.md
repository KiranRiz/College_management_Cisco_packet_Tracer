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

## 6. Extended ACL — Student/Wireless Containment

**Requirement:** Students (wired, VLAN 20, and wireless, VLAN 25) must be
able to reach the DNS server and the college web portal, and the Internet
(via NAT), but must **not** be able to reach the Accounts (VLAN 60) or
Management/Principal (VLAN 70) subnets.

Implemented as extended ACL `110` on `SW-CORE`, applied inbound on the
`Vlan20` and `Vlan25` SVIs:

```
ip access-list extended STUDENT-RESTRICT
 remark Deny Students/Wireless -> sensitive Accounts & Management subnets
 deny   ip 10.10.20.0 0.0.0.255 10.10.60.0 0.0.0.255
 deny   ip 10.10.20.0 0.0.0.255 10.10.70.0 0.0.0.255
 deny   ip 10.10.25.0 0.0.0.255 10.10.60.0 0.0.0.255
 deny   ip 10.10.25.0 0.0.0.255 10.10.70.0 0.0.0.255
 remark Explicitly permit required services to the server VLAN
 permit udp 10.10.20.0 0.0.0.255 host 10.10.80.10 eq 53
 permit udp 10.10.25.0 0.0.0.255 host 10.10.80.10 eq 53
 permit tcp 10.10.20.0 0.0.0.255 host 10.10.80.10 eq 80
 permit tcp 10.10.25.0 0.0.0.255 host 10.10.80.10 eq 80
 permit icmp 10.10.20.0 0.0.0.255 host 10.10.80.10
 permit icmp 10.10.25.0 0.0.0.255 host 10.10.80.10
 remark Permit everything else (e.g. Internet access via NAT at R-EDGE)
 permit ip any any
```

Full configuration with interface application is in
[`configs/switch-configs/SW-CORE.txt`](../configs/switch-configs/SW-CORE.txt).

**Why this order:** ACLs are processed top-down, first match wins. The
specific `deny` statements for Accounts/Management must come *before* the
general `permit ip any any`, otherwise the explicit denies would never be
reached.

## 7. NAT / PAT (see also [routing.md](routing.md))

`R-EDGE` performs NAT overload (PAT), translating the entire
`10.10.0.0/16` campus range to its single WAN IP when talking to the
Internet (`EXT-SRV`), hiding internal addressing from the outside world.
Traffic destined for the Remote Office over the VPN is explicitly
**excluded** from NAT (NAT exemption), since IPsec must encrypt the
original private addresses end-to-end for routing to work correctly at the
remote site.

## 8. Firewall — IOS CBAC (Context-Based Access Control)

**What's implemented:** On `R-EDGE`'s WAN-facing interface, an inbound
extended ACL denies all unsolicited inbound connections from the Internet
by default, while `ip inspect` (CBAC) is applied outbound so that
connections *initiated from inside* the campus (web browsing, DNS queries)
are dynamically permitted back in for their return traffic only.

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

**What it protects:** The campus LAN from unsolicited inbound connections
originating on the Internet — i.e., nothing outside can initiate a new
session into any campus host, but anything a campus host starts (browsing,
DNS lookups) works normally.

**Packet Tracer / CBAC limitations (documented honestly):**
- CBAC is a legacy IOS feature; modern real-world deployments use
  **Zone-Based Policy Firewall (ZFW)** or a dedicated firewall appliance
  (e.g. Cisco ASA — Packet Tracer does ship an ASA5505 model, but it was
  not used here to keep the topology focused and because CBAC already
  demonstrates the stateful-inspection *concept* clearly on hardware
  already present in the design).
- CBAC/PT does not provide deep packet inspection, application-layer
  filtering, or intrusion prevention — it only tracks TCP/UDP/ICMP session
  state to decide whether return traffic should be allowed back in.
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
