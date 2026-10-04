# Traffic Analysis: HTTP and TLS (Wireshark)

## Important and Honest Disclaimer

**Cisco Packet Tracer does not provide real Wireshark packet capture.**
Its "Simulation Mode" shows a simplified, Packet-Tracer-specific
visualization of PDUs (Protocol Data Units) traveling between devices —
it is useful for teaching the OSI model step-by-step, but it is **not**
a `.pcap` capture and does not show real frame/packet byte structure the
way Wireshark does on an actual network interface.

This project does **not** claim that Wireshark was run inside Packet
Tracer, and does **not** include any fabricated "capture" screenshots.
Instead, this document explains — accurately, based on real TCP/IP and
TLS behaviour — exactly what an analyst *would* observe if they ran
Wireshark on a real network while a client accessed this same college
portal design. This is the honest and professionally correct way to
connect a Packet-Tracer-based design project to real traffic-analysis
skills.

## How you would actually capture this traffic in a real network

1. Run Wireshark on the client PC itself, or on a switch configured with
   a SPAN/mirror port (`monitor session 1 source interface Gi0/1` /
   `monitor session 1 destination interface Gi0/2` on a real Cisco
   switch) facing the segment of interest.
2. Start a capture filtered to the relevant host/port, e.g.
   `host 10.10.80.10 and (tcp port 80 or tcp port 443)`.
3. Generate the traffic (browse to `http://www.college.local` or
   `https://www.college.local`).
4. Stop the capture and analyze.

## HTTP Traffic Analysis (what would be observed)

A student PC (e.g. `10.10.20.50`) browsing to `http://www.college.local`
(resolved via DNS to `10.10.80.10`) would show, in order:

1. **DNS query/response** (see below) resolving `www.college.local` to
   `10.10.80.10`.
2. **TCP three-way handshake** to the web server on port 80:
   - `10.10.20.50:<ephemeral-port> → 10.10.80.10:80  [SYN]`
   - `10.10.80.10:80 → 10.10.20.50:<ephemeral-port>  [SYN, ACK]`
   - `10.10.20.50:<ephemeral-port> → 10.10.80.10:80  [ACK]`
3. **HTTP GET request**, sent as cleartext inside the now-established TCP
   stream:
   ```
   GET / HTTP/1.1
   Host: www.college.local
   User-Agent: ...
   Accept: text/html
   ```
4. **HTTP response**, also cleartext:
   ```
   HTTP/1.1 200 OK
   Content-Type: text/html
   Content-Length: ...

   <html>...college portal HTML...</html>
   ```
5. **TCP connection teardown** (`FIN`/`ACK` exchange, or `RST` if closed
   abruptly).

**Key point for the analysis:** because HTTP is cleartext, Wireshark's
"Follow TCP Stream" view would show the *entire* request and response —
including the full HTML of the portal page — in plain readable text. This
is precisely why the college portal should ultimately be served over
HTTPS in any real deployment, and why this document also covers TLS below.

**Fields an analyst would read off each packet:**

| Field | HTTP example |
|---|---|
| Source IP | 10.10.20.50 |
| Destination IP | 10.10.80.10 |
| Source port | Ephemeral, e.g. 49213 |
| Destination port | 80 |
| Protocol | TCP (carrying HTTP) |
| Application data | Fully readable HTTP request/response text |

## TLS Traffic Analysis (what would be observed over HTTPS)

If the same portal were accessed over `https://www.college.local`
(port 443), Wireshark would show:

1. **TCP three-way handshake** to port 443 (identical pattern to HTTP,
   different destination port):
   - `10.10.20.50:<ephemeral> → 10.10.80.10:443 [SYN]`
   - `10.10.80.10:443 → 10.10.20.50:<ephemeral> [SYN, ACK]`
   - `10.10.20.50:<ephemeral> → 10.10.80.10:443 [ACK]`
2. **TLS handshake** (TLS 1.2/1.3), starting immediately after the TCP
   handshake completes:
   - **Client Hello** — client proposes TLS version, cipher suites, and a
     random value; also carries the `SNI` (Server Name Indication,
     `www.college.local`) in cleartext — this one field is the only
     "readable" application-relevant data in the whole exchange.
   - **Server Hello** — server selects the TLS version and cipher suite,
     sends its own random value.
   - **Certificate** — server sends its X.509 certificate (public key +
     identity); the client validates it against a trusted CA chain.
   - **Key Exchange** (e.g. ECDHE) — both sides derive a shared symmetric
     session key without ever transmitting it on the wire.
   - **Finished** messages from both sides, encrypted under the newly
     derived session key, confirming the handshake integrity.
3. **Encrypted Application Data** — every subsequent packet, in both
   directions, is shown by Wireshark only as `Application Data` — the
   actual HTTP request/response (method, headers, HTML body, cookies,
   any login form data) is **not** visible in plaintext. An analyst
   without the private key (or a pre-shared session key log) cannot read
   it.

**Fields an analyst would read off each packet:**

| Field | TLS example |
|---|---|
| Source IP | 10.10.20.50 |
| Destination IP | 10.10.80.10 |
| Source port | Ephemeral, e.g. 51022 |
| Destination port | 443 |
| Visible plaintext | TCP/IP headers, TLS record headers, SNI hostname in Client Hello only |
| Hidden/encrypted | HTTP method, headers, cookies, HTML body, any submitted form data |

## Why this comparison matters for the college network

- Any portal field that could carry sensitive data (student login
  credentials, staff records, accounts/finance information) **must** be
  served over HTTPS/TLS, not HTTP, specifically because — as shown above —
  HTTP is trivially readable by anyone able to capture traffic on the
  path (a compromised switch, a rogue access point, an ARP-spoofing host
  on the same VLAN), whereas TLS reduces that exposure to connection
  metadata only.
- This is also the practical argument for the VLAN segmentation and
  port-security/ACL controls documented in [security.md](security.md):
  reducing *who can even see the traffic in the first place* is a
  meaningful defense-in-depth layer even before encryption is considered.

## DNS Traffic (for completeness)

Before either HTTP or HTTPS begins, a DNS query/response pair would be
visible in Wireshark:
```
10.10.20.50:<ephemeral> → 10.10.80.10:53   [UDP] Standard query  A www.college.local
10.10.80.10:53 → 10.10.20.50:<ephemeral>   [UDP] Standard query response  A 10.10.80.10
```
DNS over UDP port 53 is also cleartext — this is standard behavior for
traditional DNS and is noted here for completeness, not implemented
differently in this project (DNS-over-HTTPS/TLS was out of scope for a
Packet Tracer-based internal DNS server).
