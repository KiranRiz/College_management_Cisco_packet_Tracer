# Routing Design

## Inter-VLAN Routing: Multilayer Switch (SVI) vs Router-on-a-Stick

Two options were considered for inter-VLAN routing:

1. **Router-on-a-stick** — a single router interface trunked to the switch,
   using sub-interfaces (one per VLAN) to route between VLANs.
2. **Multilayer (Layer-3) switch with SVIs** — each VLAN gets a switched
   virtual interface directly on the core switch, and the switch's routing
   engine handles inter-VLAN traffic in hardware.

**This project uses option 2 (SW-CORE as a Layer-3 switch).**

### Why

- A router-on-a-stick forces *all* inter-VLAN traffic through one physical
  trunk link and one router CPU — with 10 VLANs on one college network,
  that link would be a throughput bottleneck and a single point of failure
  for every department talking to every other department.
- A Layer-3 switch routes between VLANs internally at wire speed (ASIC-based
  switching), and only traffic that actually needs to leave the campus (the
  default route) is sent up to the dedicated border router, `R-EDGE`.
- This is also how real enterprise/campus networks are built today —
  router-on-a-stick is typically only seen in small labs or branch offices
  with 2–3 VLANs, not in a 10-VLAN, medium-sized campus.

## Routing Protocol: Static Routing (not a dynamic IGP)

**Static and default routing is used throughout this project — no OSPF/EIGRP.**

### Why static routing is the right choice here

- The topology is small and stable: one core switch, one border router, one
  remote site. There are only 3 non-directly-connected routing decisions in
  the entire design (campus → Internet, campus ↔ remote site, remote site →
  everything). Running a dynamic routing protocol to manage 3 routes would
  be complexity for its own sake.
- Static/default routes are also easier to secure and audit — there's no
  routing protocol adjacency to spoof or inject malicious routes into on
  this network's edge.
- In an interview, "I used static routing because the topology is small,
  stable, and the routing table only needed 3 entries — a dynamic routing
  protocol would have been complexity without benefit" is a stronger, more
  senior answer than "I ran OSPF because that's what's covered in the
  textbook."

If the college were to grow to multiple buildings/multiple core switches
with many redundant paths, OSPF would become the better choice — this
trade-off is called out explicitly here because recognizing *when* static
routing stops being appropriate is itself part of the skill being
demonstrated.

## Routing Tables (as configured)

### SW-CORE (Layer-3 core switch)
```
! Directly connected: all 10 VLAN SVIs (10.10.10.0/24 ... 10.10.99.0/24)
! and the transit link to R-EDGE (10.10.1.0/30)
ip route 0.0.0.0 0.0.0.0 10.10.1.2        ! default route -> R-EDGE
```
Everything not on a locally-connected VLAN (i.e. the Internet, and the
remote office) is sent to `R-EDGE` via the default route. `R-EDGE` is the
only exit point from the campus LAN.

### R-EDGE (Border router)
```
ip route 10.10.0.0 255.255.0.0 10.10.1.1      ! campus VLANs -> SW-CORE
ip route 10.20.10.0 255.255.255.0 198.51.100.1 ! remote office -> via ISP, VPN-protected
ip route 0.0.0.0 0.0.0.0 198.51.100.1          ! default route -> ISP (Internet)
```
`R-EDGE` has three possible destinations: back into the campus LAN, out to
the remote office (automatically encrypted by the IPsec crypto map — see
[vpn.md](vpn.md)), or out to the Internet by default.

### R-REMOTE (Remote office router)
```
ip route 0.0.0.0 0.0.0.0 198.51.100.5          ! default route -> ISP
```
The remote office is a simple branch — everything it doesn't know about
locally goes out its single default route. Traffic destined for the campus
10.10.0.0/16 range is automatically picked up and encrypted by its own
crypto map before it reaches the ISP.

### ISP-RTR (simulated Internet core)
No static routes needed — `ISP-RTR` has three directly-connected interfaces
(to `R-EDGE`, to `R-REMOTE`, and to `EXT-SRV`) and simply forwards between
them, exactly like a real ISP router forwarding between its customers.

## Verifying Routing

See [troubleshooting.md](troubleshooting.md) for the full command list, but
the key routing verification commands used on this project are:

```
show ip route
show ip interface brief
show ip protocols      ! confirms no dynamic routing protocol is running
traceroute <destination>
```
