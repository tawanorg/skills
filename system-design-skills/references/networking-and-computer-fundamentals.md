# Networking and Computer Fundamentals

A working reference for the network and operating-system mechanics that show up in
system-design interviews and in production debugging. Each section is built to be
skimmed under pressure: tables for lookup, diagrams for the protocols that have moving
parts.

## OSI Model vs TCP/IP Model

The OSI model is a 7-layer teaching abstraction; the TCP/IP model is the 4-layer
implementation the real Internet runs on. The mapping is not perfectly clean — OSI's
session and presentation concerns are mostly folded into the application layer in
practice.

| OSI Layer | # | TCP/IP Layer | Purpose | Example Protocols / Units |
|-----------|---|--------------|---------|---------------------------|
| Application | 7 | Application | App-facing semantics, requests/responses | HTTP, DNS, SMTP, SSH, gRPC |
| Presentation | 6 | Application | Encoding, serialization, encryption | TLS, JPEG, ASCII, Protobuf |
| Session | 5 | Application | Establish/maintain/close dialogues | TLS sessions, RPC, NetBIOS |
| Transport | 4 | Transport | End-to-end delivery, ports, reliability | TCP, UDP, QUIC (segments/datagrams) |
| Network | 3 | Internet | Logical addressing, routing between networks | IP, ICMP, BGP, OSPF (packets) |
| Data Link | 2 | Link | Framing, MAC addressing, local delivery | Ethernet, Wi-Fi (802.11), ARP (frames) |
| Physical | 1 | Link | Bits over a medium | copper, fiber, radio (bits) |

Mental model: data is **encapsulated** going down the stack (each layer prepends its
header) and **de-encapsulated** going up. A "packet" at L3 becomes the payload of a
"frame" at L2.

## TCP vs UDP

| Property | TCP | UDP |
|----------|-----|-----|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Guaranteed delivery, retransmission | Best-effort, no retransmit |
| Ordering | In-order via sequence numbers | No ordering guarantee |
| Flow/congestion control | Yes (windowing, AIMD) | No (app must handle it) |
| Header size | 20+ bytes | 8 bytes |
| Speed/overhead | Higher latency, more overhead | Low latency, minimal overhead |
| Use cases | HTTP, email, file transfer, DB | DNS, VoIP, live video, gaming, QUIC |

### TCP 3-Way Handshake

```
Client                         Server
  |  ---- SYN (seq=x) ------->  |   client opens, picks ISN x
  |  <-- SYN-ACK (seq=y,       |   server acks x+1, picks ISN y
  |        ack=x+1) -----------|
  |  ---- ACK (ack=y+1) ----->  |   connection ESTABLISHED
  |                             |
  |  ===== data flows =====     |
```

Teardown is a 4-way exchange (FIN / ACK in each direction); the active closer sits in
`TIME_WAIT` (~2×MSL) to absorb late-arriving segments before fully releasing the port.

### Why gaming/streaming often choose UDP

Real-time media values **timeliness over completeness**. If a video frame or a player
position packet is lost, retransmitting it is useless — by the time it arrives, the
moment has passed; the app would rather drop it and move on. UDP gives the application
full control to do interpolation, forward error correction, or simply skip, instead of
TCP stalling the whole stream to redeliver one old packet.

### Head-of-line (HOL) blocking

TCP delivers bytes strictly in order. If segment #5 is lost, segments #6–#10 may have
already arrived but the kernel **cannot hand them to the application** until #5 is
retransmitted and received — everything queues behind the missing segment. This hurts
multiplexed protocols most: HTTP/2 puts many streams on one TCP connection, so one lost
packet blocks all streams. QUIC (HTTP/3) fixes this by running independent streams over
UDP, where loss in one stream doesn't stall the others.

## IP Addressing

| | IPv4 | IPv6 |
|---|------|------|
| Size | 32-bit (~4.3 billion) | 128-bit (~3.4×10³⁸) |
| Notation | dotted decimal `192.0.2.1` | hex groups `2001:db8::1` |
| Header | variable, includes checksum | fixed 40 bytes, no checksum, simpler |
| Config | manual / DHCP | SLAAC (autoconfig) or DHCPv6 |
| NAT | ubiquitous | largely unnecessary |

**Why IPv6:** IPv4 address exhaustion. The free pool was depleted (IANA ran out in
2011), and IPv6's vast space removes the need for address scarcity workarounds, plus it
streamlines routing and supports built-in autoconfiguration.

**Private vs public.** Public addresses are globally routable. Private ranges (RFC 1918)
are reusable inside any network and never routed on the public Internet:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`
- (plus `127.0.0.0/8` loopback, `169.254.0.0/16` link-local)

**CIDR basics.** `a.b.c.d/n` means the first `n` bits are the network prefix; the
remaining `32−n` bits address hosts. `/24` = 256 addresses (254 usable: network +
broadcast reserved). Smaller `n` = bigger block. `/16` = 65,536 addresses. CIDR
replaced rigid Class A/B/C boundaries, enabling flexible allocation and route
aggregation (supernetting).

## NAT (Network Address Translation)

A router with NAT rewrites the source IP (and port) of outbound packets from private
hosts to its single public IP, and reverses the rewrite for the return traffic. It
tracks each flow in a translation table keyed by (internal IP:port → external port).

```
192.168.1.5:51000  --->  [NAT]  --->  203.0.113.9:40001  ---> server:443
   (private)                            (public)
```

**Why it extended IPv4's life:** an entire household or office of devices can share one
public address. This let the world keep growing long after IPv4 should have run out.
The port-multiplexing form is **PAT / NAT overload** — many internal hosts mapped to one
public IP, disambiguated entirely by port number. The downside is that inbound
connections need explicit port forwarding or hole-punching, since there's no permanent
public address per host.

## DNS

DNS translates human names into IP addresses (and other records). It is a distributed,
hierarchical, heavily cached key-value system.

### Record Types

| Type | Maps | Notes |
|------|------|-------|
| A | name → IPv4 | the classic forward lookup |
| AAAA | name → IPv6 | "quad-A" |
| CNAME | alias → canonical name | can't coexist with other records at same name |
| MX | domain → mail server | has a priority field; lower = preferred |
| TXT | name → free text | SPF, DKIM, domain verification |
| NS | zone → authoritative nameservers | delegation |
| SOA | zone metadata | primary NS, serial, refresh/retry/expire/min-TTL |
| PTR | IP → name | reverse DNS, lives under `in-addr.arpa` / `ip6.arpa` |

### Recursive Lookup Flow

The **resolver** (usually your ISP's or `8.8.8.8`) does the legwork on the client's
behalf, walking down the hierarchy:

```
Client          Recursive Resolver        Root        TLD (.com)     Authoritative
  |  who is        |                        |              |              |
  | www.ex.com? -->|                        |              |              |
  |                | -- ? --------------->   |              |              |
  |                | <- ask .com TLD -----   |              |              |
  |                | -- ? -------------------------->       |              |
  |                | <- ask ns.ex.com ------------------    |              |
  |                | -- ? ------------------------------------------->     |
  |                | <- 93.184.216.34 ------------------------------------ |
  | <- 93.184...   |                                                       |
```

1. Resolver checks its cache; on miss, queries a **root** server.
2. Root replies with the **TLD** nameservers for `.com`.
3. TLD replies with the **authoritative** nameservers for `example.com`.
4. Authoritative server returns the actual A/AAAA record.
5. Resolver caches the answer and returns it to the client.

**TTL / caching.** Every record carries a TTL (seconds). Resolvers and clients cache
until it expires, which slashes load and latency but means changes propagate slowly —
lower the TTL before a planned migration. Negative answers (NXDOMAIN) are cached too,
bounded by the SOA minimum TTL.

## Common Ports

| Port | Protocol | Service |
|------|----------|---------|
| 20/21 | TCP | FTP (data / control) |
| 22 | TCP | SSH / SCP / SFTP |
| 25 | TCP | SMTP (mail relay) |
| 53 | UDP/TCP | DNS (TCP for zone transfer / large responses) |
| 67/68 | UDP | DHCP (server / client) |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 143 | TCP | IMAP |
| 443 | TCP/UDP | HTTPS (TCP for TLS, UDP for HTTP/3/QUIC) |
| 3306 | TCP | MySQL / MariaDB |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 27017 | TCP | MongoDB |
| 9092 | TCP | Kafka |
| 11211 | TCP/UDP | Memcached |

Ports 0–1023 are "well-known" (privileged); 1024–49151 registered; 49152–65535 ephemeral
(used for client-side source ports).

## Network Scopes

| Scope | Range | Example |
|-------|-------|---------|
| PAN — Personal Area | a few meters | Bluetooth earbuds, USB tether |
| LAN — Local Area | building / campus | home Wi-Fi, office Ethernet |
| MAN — Metropolitan Area | a city | municipal fiber, cable provider region |
| WAN — Wide Area | country / global | the Internet, corporate inter-site links |

## Cast Types (Delivery Models)

| Type | Targets | Description | Where used |
|------|---------|-------------|-----------|
| Unicast | 1 → 1 | one sender, one receiver | almost all normal traffic |
| Broadcast | 1 → all (subnet) | sent to every host on the LAN | ARP, DHCP discovery |
| Multicast | 1 → group | delivered to subscribers of a group | IPTV, market data feeds, mDNS |
| Anycast | 1 → nearest of many | same address advertised from many sites; routing picks closest | CDNs, root DNS, `8.8.8.8` |

**Anycast** is the workhorse behind global scale: many physical servers share one IP, and
BGP routes each client to the topologically nearest instance. This is how a CDN edge or a
public DNS resolver gives low latency worldwide without the client knowing which box it
hit. IPv6 dropped broadcast entirely in favor of multicast.

## Eight Protocols in Brief

1. **HTTP** — stateless request/response for the web; methods (GET/POST/…), status codes, over TCP:80.
2. **HTTPS** — HTTP wrapped in TLS for encryption + server authentication; TCP:443.
3. **FTP** — file transfer with separate control/data channels; largely superseded by SFTP/HTTPS.
4. **SMTP** — pushes mail between servers; paired with IMAP/POP3 for retrieval.
5. **DNS** — name → address resolution; mostly UDP:53, falls back to TCP for big payloads.
6. **DHCP** — auto-assigns IP, gateway, DNS to hosts on join (DORA: Discover, Offer, Request, Ack).
7. **SSH** — encrypted remote shell, tunneling, and file copy; TCP:22.
8. **WebSocket** — full-duplex, persistent channel upgraded from an HTTP handshake, over TCP; ideal for chat, live dashboards, and push.

## Internet Traffic Routing Policies

DNS-based global load balancing and CDNs steer clients using policies that can be combined:

- **Geolocation / geoproximity** — route by the client's region (data residency, language, nearest POP).
- **Latency-based** — measure and send the client to the lowest-RTT endpoint.
- **Weighted** — split traffic by assigned percentages (canary rollouts, A/B, gradual migration).
- **Failover** — health-checked primary with automatic switch to standby.

In practice a CDN layers anycast (network-level "nearest") under these DNS policies for
the best of both.

## OS Fundamentals

### Process vs Thread

| | Process | Thread |
|---|---------|--------|
| Address space | own, isolated | shared within the process |
| Memory sharing | requires IPC (pipes, shm, sockets) | direct via shared heap/globals |
| Crash blast radius | isolated; one crash doesn't kill siblings | can take down the whole process |
| Context switch | heavier (swap page tables, flush TLB) | lighter (same address space) |
| Creation cost | higher | lower |

Threads share code, heap, and open files but each gets its own stack and registers.
Sharing memory is fast but forces synchronization (locks, atomics) to avoid races.

### Concurrency vs Parallelism

- **Concurrency** — *dealing with* many tasks by interleaving progress; possible on a
  single core (the scheduler time-slices). About structure.
- **Parallelism** — *doing* many tasks literally at the same instant, requiring multiple
  cores/machines. About execution.

You can have concurrency without parallelism (one core, many goroutines) and parallelism
is a way to *realize* concurrency when hardware allows.

### Paging vs Segmentation

Both implement virtual memory, decoupling logical addresses from physical RAM.

| | Paging | Segmentation |
|---|--------|--------------|
| Unit | fixed-size pages (e.g. 4 KB) | variable-size logical segments (code, stack, heap) |
| Fragmentation | internal (partial last page) | external (gaps between segments) |
| View | flat, hardware-oriented | matches program's logical structure |
| Address | page number + offset | segment selector + offset |

Modern OSes lean on paging with a **page table** (often multi-level) translated by the
MMU and cached in the TLB; pages not in RAM live on disk (swap) and trigger a **page
fault** when touched. Some architectures combine both (segmented paging).

### Deadlock — Coffman Conditions

A deadlock requires **all four** conditions to hold simultaneously:

1. **Mutual exclusion** — a resource is held in non-shareable mode.
2. **Hold and wait** — a process holds ≥1 resource while waiting for another.
3. **No preemption** — resources are released only voluntarily, not forcibly taken.
4. **Circular wait** — a closed chain of processes each waiting on the next.

Strategies:

- **Prevention** — break any one condition. E.g. impose a global lock-ordering to kill
  circular wait; require all-or-nothing acquisition to kill hold-and-wait.
- **Avoidance** — grant requests only if the system stays in a *safe* state
  (Banker's algorithm) using advance knowledge of max needs.
- **Detection & recovery** — let deadlocks happen, scan a wait-for graph for cycles, then
  recover by killing or rolling back a victim.
- **Ostrich algorithm** — ignore it (common in practice when deadlocks are rare).

## How a Browser Renders a Page

```
URL entered
   │
   ▼
DNS resolve ──► TCP connect ──► TLS handshake ──► HTTP request/response
                                                        │
                                                        ▼
                                          Parse HTML ──► DOM tree
                                          Parse CSS  ──► CSSOM tree
                                                        │
                                          DOM + CSSOM ─► Render tree (visible nodes)
                                                        │
                                                        ▼
                                                     Layout (reflow: geometry/positions)
                                                        │
                                                        ▼
                                                     Paint (rasterize pixels)
                                                        │
                                                        ▼
                                                     Composite layers ─► screen
```

Key points: HTML parsing builds the **DOM**; stylesheets build the **CSSOM**; together
they form the **render tree** (display:none nodes excluded). **Layout** computes box
geometry; **paint** fills pixels; **compositing** stacks GPU layers. A blocking
`<script>` pauses parsing unless marked `async`/`defer`. Changing geometry triggers a
costly **reflow**; changing only colors triggers a cheaper **repaint** — which is why
animating `transform`/`opacity` (composite-only) is preferred.
