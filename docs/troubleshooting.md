# Network Troubleshooting

## Methodology

Troubleshooting on this network follows the standard bottom-up OSI
approach: verify Layer 1/2 (cable, port status, VLAN membership) before
Layer 3 (IP addressing, routing) before Layer 4+ (ports, services like DNS
and HTTP). Jumping straight to "the website doesn't load" and assuming a
web server problem is how real troubleshooting time gets wasted — most
"the portal is down" tickets on this network turn out to be Layer 1-3
problems.

## Command Reference by Layer

| Layer / Area        | Command                                   | What it confirms |
|-----------------------|---------------------------------------------|----------------------|
| Physical / cabling    | `show interfaces status` / `show ip interface brief` | Port is up/up, not administratively down or err-disabled |
| VLAN membership       | `show vlan brief`                           | Port is assigned to the correct VLAN |
| Trunking               | `show interfaces trunk`                     | Trunk is up, correct native VLAN, correct allowed-VLAN list |
| MAC learning           | `show mac address-table`                    | Switch has learned the expected MAC on the expected port/VLAN |
| IP addressing (client) | `ipconfig /all` (Windows) or `ipconfig` (PT PC) | Client received correct IP/mask/gateway/DNS from DHCP |
| Reachability           | `ping <target>`                             | Layer 3 connectivity (ICMP) to gateway, server, or remote host |
| Path                   | `tracert` (PC) / `traceroute` (IOS)         | Identifies exactly which hop is dropping traffic |
| ARP                    | `show arp` / `arp -a`                       | Layer 2-to-3 mapping is correct; detects duplicate IP/ARP issues |
| Routing                | `show ip route`                             | Expected route (static/default) is present and points to the correct next hop |
| DNS                    | `nslookup www.college.local` (on PCs that support it) | Confirms name resolution is working and returns the correct IP |
| ACL behaviour          | `show access-lists`                         | Match counters show whether traffic is being permitted/denied as expected |
| Running config sanity  | `show running-config`                       | Confirms the intended configuration was actually applied/saved |
| Port security          | `show port-security interface <if>`         | Confirms a port hasn't gone into `err-disabled`/`restrict` due to a MAC violation |

## Example Troubleshooting Scenarios (diagnosed on this network)

### Scenario 1 — "Student PC has no IP address"
1. `ipconfig` on the PC shows `169.254.x.x` (APIPA) → DHCP failed.
2. Checked the access port: `show vlan brief` — port was still in the
   default VLAN 1, not VLAN 20. **Root cause:** the access port had not
   been explicitly assigned `switchport access vlan 20`.
3. Fix: applied the correct `switchport access vlan 20` command, port
   re-initialized, PC received a lease from the SW-CORE DHCP pool.

### Scenario 2 — "Faculty PC can reach the Library PC but not the Internet"
1. `ping 10.10.80.10` (server) succeeds → intra-campus routing is fine.
2. `ping 198.51.100.10` (EXT-SRV, "the Internet") fails.
3. `show ip route` on R-EDGE — default route to ISP was present.
4. `show ip nat translations` showed no translation entries being created.
   **Root cause:** the NAT inside/outside interface marking
   (`ip nat inside` / `ip nat outside`) was missing on one interface.
5. Fix: applied `ip nat inside` on the SW-CORE-facing interface of R-EDGE;
   translations began appearing immediately and Internet ping succeeded.

### Scenario 3 — "Two switches on the same trunk can't see each other's VLANs"
1. `show interfaces trunk` on both switches showed the trunk up, but the
   **allowed VLAN list** on one side had been pruned too aggressively
   during initial configuration (a VLAN was accidentally left out of
   `switchport trunk allowed vlan`).
2. Fix: corrected the allowed-VLAN list to include every VLAN that
   actually needs to cross that trunk.

### Scenario 4 — "Student VLAN can reach Accounts VLAN when it shouldn't"
1. `show access-lists 110` showed zero matches on the deny lines — meaning
   traffic wasn't hitting the ACL at all.
2. **Root cause:** the ACL was built but never applied to the `Vlan20`
   SVI with `ip access-group 110 in`.
3. Fix: applied the access-group statement to both the `Vlan20` and
   `Vlan25` SVIs; subsequent `ping` from a Student PC to an Accounts PC
   timed out as intended, while DNS/web traffic to the server continued
   to work.

### Scenario 5 — "Remote office VPN tunnel never comes up"
1. `show crypto isakmp sa` showed no SA at all (not even negotiating).
2. Generated interesting traffic (`ping` from a remote PC to a campus PC)
   — IPsec is **traffic-triggered**, it will not establish just because
   the config is present.
3. Still nothing — checked `show crypto map`: the crypto map was
   configured but **not applied** to the WAN interface.
4. Fix: applied `crypto map CMAP-VPN` under the WAN-facing interface on
   both routers; the next ping triggered IKE Phase 1/2 negotiation and the
   tunnel came up (`QM_IDLE`).

## Common Failure Patterns (quick checklist)

- "No IP address" → check switchport VLAN assignment and DHCP pool/exclusions first, not the server.
- "Everything on my VLAN works, nothing else does" → check the default route and NAT inside/outside markings on R-EDGE.
- "Works on wired, not on Wi-Fi" → check AP SSID-to-VLAN mapping and WPA2 PSK, then DHCP for VLAN 25.
- "ACL doesn't seem to do anything" → 99% of the time, the ACL was never actually applied to an interface.
- "VPN never comes up" → remember it's traffic-triggered; generate real traffic before assuming the config is broken.
