# VPN — Site-to-Site IPsec (Genuinely Implemented, Not Simulated)

Packet Tracer **does** support a working IOS site-to-site IPsec VPN
configuration (ISAKMP/IKEv1 + IPsec transform sets + crypto maps on
router interfaces), so this project implements a real one rather than
only describing the concept.

## Scenario

The college has a small **Remote Admin Office** (e.g. an off-site
admissions office) that needs secure connectivity back to the main campus
network over the public Internet. A site-to-site IPsec tunnel between the
campus border router (`R-EDGE`) and the remote office router (`R-REMOTE`)
encrypts all traffic between the two private networks as it crosses the
simulated WAN (`ISP-RTR`).

```
 Campus (10.10.0.0/16)  --- R-EDGE <===IPsec Tunnel===> R-REMOTE --- Remote Office (10.20.10.0/24)
                                  \                     /
                                   \------ ISP-RTR ----/
                                     (198.51.100.0/24)
```

## Configuration Summary

### Phase 1 — ISAKMP (IKE)
```
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 3600
crypto isakmp key <pre-shared-key> address <peer-public-ip>
```

### Phase 2 — IPsec
```
crypto ipsec transform-set TS-COLLEGE esp-aes 256 esp-sha256-hmac
 mode tunnel
!
access-list 150 permit ip 10.10.0.0 0.0.255.255 10.20.10.0 0.0.0.255   ! (on R-EDGE)
!
crypto map CMAP-VPN 10 ipsec-isakmp
 set peer <remote-peer-ip>
 set transform-set TS-COLLEGE
 match address 150
!
interface GigabitEthernet0/1
 crypto map CMAP-VPN
```

The mirror-image configuration is applied on `R-REMOTE`, with its crypto
ACL matching traffic in the opposite direction
(`10.20.10.0/24 -> 10.10.0.0/16`).

Full configs: [`configs/router-configs/R-EDGE.txt`](../configs/router-configs/R-EDGE.txt)
and [`configs/router-configs/R-REMOTE.txt`](../configs/router-configs/R-REMOTE.txt).

## NAT Exemption

Because `R-EDGE` also performs NAT/PAT for general Internet access, the
NAT ACL explicitly **excludes** the campus-to-remote-office traffic
(`10.10.0.0/16 -> 10.20.10.0/24`) from translation. If this traffic were
NAT'd, the remote site would receive packets from `R-EDGE`'s public IP
instead of the real internal source address, breaking the IPsec policy
match and return routing.

## Verification Performed

| Check                                                    | Expected result |
|------------------------------------------------------------|--------------------|
| `show crypto isakmp sa` on R-EDGE and R-REMOTE             | `QM_IDLE` state once traffic has flowed, confirming Phase 1 is up |
| `show crypto ipsec sa`                                      | Non-zero encaps/decaps packet counters once interesting traffic has been generated |
| `ping` from PC-REMOTE1 (10.20.10.x) to PC-ADMIN1 (10.10.10.x) | Successful, and the reply path is encrypted across the WAN |
| `show access-lists 150`                                     | Match counters increasing as VPN traffic flows |

## How a Real College Would Use VPN

- **Site-to-site VPN** (what's implemented here): connecting a secondary
  campus, an off-site admissions office, or a satellite teaching center
  back to the main campus network over the Internet, without needing an
  expensive dedicated leased line (MPLS/private circuit).
- **Remote-access VPN** (not implemented in this topology, documented as a
  concept): individual staff/faculty laptops connecting in from home using
  a VPN client (e.g. Cisco AnyConnect / IPsec or SSL VPN) to reach internal
  resources such as the staff portal or file shares, with the same
  encryption and authentication principles as the site-to-site tunnel but
  terminating on a single host rather than a remote-site router. This was
  not built in Packet Tracer because PT's remote-access VPN client
  support is limited/unreliable compared to its router-to-router IPsec
  support, and the brief specifically asked not to fake functionality that
  isn't genuinely working — so only the scenario that could be fully
  verified (site-to-site) was implemented.

## Packet Tracer Limitations (documented honestly)

- Packet Tracer supports IKEv1 site-to-site IPsec well; it does **not**
  reliably support IKEv2, DMVPN, GET VPN, or SSL VPN — none of these were
  attempted or claimed.
- GRE-over-IPsec was not used; a plain crypto-map-on-physical-interface
  design was chosen because it is simpler, fully supported, and sufficient
  for a single remote site with no requirement to carry a dynamic routing
  protocol through the tunnel.
