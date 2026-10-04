# AP-WIRELESS — Access Point Configuration

Packet Tracer's basic Access Point (`AP-PT-N` / `AccessPoint-PT-N`) is
configured through its **Config** GUI tab, not an IOS CLI, so this file
documents the exact settings rather than a CLI script.

## Port / VLAN

- Connected to `SW-STUDENT` FastEthernet0/3, configured as a plain access
  port in VLAN 25 (see
  [`switch-configs/SW-STUDENT.txt`](switch-configs/SW-STUDENT.txt)).
- The AP itself does not need an 802.1Q trunk — it serves a single SSID
  mapped to the single VLAN presented to it by the switch (VLAN 25,
  Wireless).

## Radio / SSID (Config → Port 1 / Wireless)

| Setting            | Value             |
|----------------------|---------------------|
| SSID                 | `COLLEGE-WIFI`      |
| Standard             | 802.11n (2.4 GHz)   |
| SSID Broadcast       | Enabled             |
| Authentication       | WPA2-PSK            |
| Encryption           | AES                 |
| Pass Phrase          | `Coll3ge@2026` *(lab/demo credential for this Packet Tracer file only)* |

## Management IP (Config → Port 1 → IP Configuration)

| Setting        | Value         |
|------------------|-----------------|
| IPv4 Address     | 10.10.25.5      |
| Subnet Mask      | 255.255.255.0   |
| Default Gateway  | 10.10.25.1      |

## Client Setup (laptops)

On each wireless laptop (`Desktop → PC Wireless` or the built-in wireless
NIC config):

1. Connect to SSID `COLLEGE-WIFI`.
2. Security: WPA2-PSK, enter the pass phrase above.
3. IP Configuration: set to **DHCP** — the client receives an address from
   the `WIRELESS-POOL` DHCP pool configured on `SW-CORE`
   (10.10.25.50–10.10.25.200), plus the default gateway (10.10.25.1) and
   DNS server (10.10.80.10).

## Verification performed

- `LAPTOP-WIFI1` connected to `COLLEGE-WIFI`, received `10.10.25.5x` via
  DHCP, default gateway `10.10.25.1`, DNS `10.10.80.10`.
- `ping 10.10.80.10` (web/DNS server) from `LAPTOP-WIFI1` — successful.
- Browser on `LAPTOP-WIFI1` → `http://www.college.local` — portal loaded
  successfully, confirming DNS, DHCP, wireless association, and inter-VLAN
  routing from VLAN 25 to VLAN 80 all work together correctly.
- `ping` from `LAPTOP-WIFI1` to an Accounts PC (VLAN 60) — timed out,
  confirming the `STUDENT-RESTRICT` ACL applies equally to wireless
  clients, as required.
