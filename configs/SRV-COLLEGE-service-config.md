# SRV-COLLEGE — Service Configuration

Packet Tracer's generic `Server-PT` device is configured through its
**Config / Services** GUI tabs rather than an IOS-style CLI, so this file
documents the exact settings to enter rather than a CLI script.

## Interface (Desktop → IP Configuration, or Config → FastEthernet0)

| Setting        | Value              |
|------------------|----------------------|
| IPv4 Address     | 10.10.80.10          |
| Subnet Mask      | 255.255.255.0        |
| Default Gateway  | 10.10.80.1           |
| DNS Server       | 10.10.80.10 (itself) |

## DNS Service (Config → Services → DNS)

Set service to **On**, then add the following A records:

| Name                     | Type | Address      |
|---------------------------|------|----------------|
| www.college.local         | A    | 10.10.80.10    |
| dns.college.local         | A    | 10.10.80.10    |
| portal.college.local      | A    | 10.10.80.10    |

## HTTP Service (Config → Services → HTTP)

1. Set **HTTP** (and HTTPS, if demonstrating the concept) to **On**.
2. Replace the default `index.html` served by the PT HTTP service with
   the contents of [`../web/index.html`](../web/index.html) in this
   repository — open the file in the PT text editor on the server (File
   Manager within the server's Services tab) and paste in the provided
   markup, or use the "Edit" button on the existing `index.html` entry.

## Verification performed

- `nslookup www.college.local` from PC-ADMIN1 and PC-STUDENT1 → resolved
  to `10.10.80.10`.
- Browser (`Desktop → Web Browser`) on PC-FACULTY1 → navigated to
  `http://www.college.local` → portal page rendered successfully.
- Browser on PC-STUDENT1 → same URL → succeeded (permitted by the
  Student-restriction ACL on SW-CORE, since port 80 to the server is
  explicitly allowed — see [`../docs/security.md`](../docs/security.md)).
- Browser on PC-STUDENT1 → attempted `http://10.10.60.x` (an Accounts
  host) → timed out, confirming the ACL blocks it as intended.
