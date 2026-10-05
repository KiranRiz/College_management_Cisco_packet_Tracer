# VPN — Verified Platform Limitation, and the Real-World Concept

## What was attempted

A site-to-site IPsec VPN between the campus border router (`R-EDGE`) and
the remote office router (`R-REMOTE`) was designed and directly tested in
the running Packet Tracer 9.0 build used for this project:

```
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 3600
crypto isakmp key <psk> address <peer>
crypto ipsec transform-set TS-COLLEGE esp-aes 256 esp-sha256-hmac
 mode tunnel
crypto map CMAP-VPN 10 ipsec-isakmp
 set peer <peer>
 set transform-set TS-COLLEGE
 match address 150
```

## What actually happened (verified, not assumed)

Every `crypto` command above was rejected on a Cisco 2911 (`R-EDGE`) with:

```
% Invalid input detected at '^' marker.
```

Suspecting the security feature set simply wasn't licensed/activated,
the standard Cisco remedy was tried next, directly on the device:

```
license boot module c2900 technology-package securityk9
write memory
reload
```

After the reload completed and the device was logged back into, the exact
same `crypto isakmp policy 10` command was retried and **still rejected**
with the identical error. This confirms — rather than assumes — that this
specific Packet Tracer 9.0 build's simulated IOS for the 2911/2901 ISR
platform does not implement the `crypto isakmp` / `crypto ipsec` /
`crypto map` command set at all, regardless of licensing state.

**Per this project's own standard (set out in the original brief): this
is not faked.** No `crypto` configuration lines are present in
[`configs/router-configs/R-EDGE.txt`](../configs/router-configs/R-EDGE.txt)
or
[`configs/router-configs/R-REMOTE.txt`](../configs/router-configs/R-REMOTE.txt).

## What is actually implemented instead

The remote site (`R-REMOTE` / `SW-REMOTE` / `10.20.10.0/24`) is still a
fully real, working part of the topology — it is just reached by **plain
static routing across the WAN**, unencrypted, instead of an IPsec tunnel:

- `R-EDGE` has `ip route 10.20.10.0 255.255.255.0 198.51.100.1`
- `R-REMOTE` has a single default route back out via the simulated ISP
- Campus→remote-office traffic is still explicitly **exempted from NAT**
  on `R-EDGE` (it's treated as internal inter-site traffic, not
  internet-bound), which is the one piece of the original NAT design that
  remains meaningful without encryption

This keeps the multi-site WAN/routing demonstration genuine and testable
(ping/traceroute between campus and remote-office hosts works end-to-end)
without overstating what's actually protecting that traffic.

## How a real college would use VPN technology (concept)

Since the live demonstration isn't possible on this platform build, here
is the concept a real deployment would use instead:

- **Site-to-site VPN**: an IPsec tunnel (as attempted above) between the
  main campus edge router/firewall and a secondary campus, satellite
  admissions office, or remote teaching center's router, built using
  either policy-based (crypto-map, as attempted here) or route-based
  (IPsec VTI / GRE-over-IPsec) configuration, protecting all traffic
  between the two private networks as it crosses the public Internet —
  removing the need for an expensive dedicated leased line.
- **Remote-access VPN**: individual staff/faculty laptops connecting from
  home using a VPN client (e.g. Cisco AnyConnect / IKEv2 or SSL VPN)
  terminating on a concentrator or firewall at the campus edge, so a
  single traveling or remote user gets the same encrypted access to
  internal resources (staff portal, file shares) that the site-to-site
  tunnel gives a whole remote office.
- In both cases the core value is the same: **confidentiality and
  integrity of private traffic crossing a network the institution does
  not control** (the public Internet) — exactly the gap that plain
  static/NAT-exempted routing (what's actually running in this project)
  does **not** close, which is worth being explicit about rather than
  leaving implied.

## Why this verification matters for the portfolio

Being able to say "I designed this, tested it directly against the
platform, hit a genuine tool limitation, confirmed it with the standard
licensing remedy, and then made a documented, honest design decision
instead of faking the output" is a stronger, more senior demonstration of
real troubleshooting and professional judgement than a config file that
merely *claims* a VPN is running.
