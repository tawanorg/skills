# API and Web Development

A practical reference for designing, transporting, and securing APIs. Covers architectural styles, HTTP mechanics, real-time delivery, pagination, versioning, the proxy/gateway family, and security — vendor-neutral, with concrete examples.

## API Architectural Styles

Pick the style by coupling, payload shape, and traffic pattern — not by hype.

| Style | Transport | Payload | Contract | Best for | Avoid when |
|-------|-----------|---------|----------|----------|------------|
| REST | HTTP/1.1+ | JSON (usually) | OpenAPI (optional) | CRUD resources, public APIs, broad client reach | Chatty multi-resource fetches, strict typing needs |
| GraphQL | HTTP (single POST) | JSON | Schema (SDL), strongly typed | Aggregating many sources, client-driven field selection, mobile bandwidth | Simple CRUD, heavy caching, file uploads |
| gRPC | HTTP/2 | Protobuf (binary) | `.proto`, codegen | Internal service-to-service, low latency, streaming | Browser clients without a proxy, human-debuggable payloads |
| SOAP | HTTP/SMTP | XML | WSDL, XSD | Legacy enterprise, formal contracts, WS-Security | Greenfield, mobile, anything latency-sensitive |
| WebSocket | TCP (HTTP upgrade) | Anything | None (app-defined) | Bidirectional real-time: chat, games, live cursors | Request/response semantics, cacheable reads |
| Webhooks | HTTP (server→server) | JSON | Provider-defined | Event notifications you don't want to poll for | Client behind a firewall, ordered/guaranteed delivery |

Rules of thumb: **REST** for public/external; **gRPC** for internal microservice meshes; **GraphQL** when clients need flexible aggregation; **WebSocket** for push that must be bidirectional; **webhooks** for "tell me when X happens" across org boundaries.

## REST Principles & Resource Modeling

REST is an architectural style, not a protocol. The constraints that matter in practice:

- **Resource-oriented**: model nouns, not verbs. `/orders/42`, not `/getOrder?id=42`.
- **Statelessness**: each request carries all context (auth, params). No server-side session affinity → easy horizontal scaling.
- **Uniform interface**: HTTP methods convey intent; URIs identify resources; representations (JSON) are interchangeable.
- **Cacheability**: responses declare whether/how they can be cached.
- **HATEOAS** (aspirational): responses include links to related actions. Rarely fully implemented; partial linking is common.

Resource modeling conventions:

```
GET    /users                 # collection
POST   /users                 # create in collection
GET    /users/123             # single resource
GET    /users/123/orders      # sub-collection (relationship)
GET    /users/123/orders/9    # nested resource
```

Use plural nouns, lowercase, hyphens for multi-word paths (`/shipping-addresses`). Keep nesting shallow (≤2 levels); beyond that, prefer query filters: `/orders?userId=123`.

### HTTP Methods: Safety & Idempotency

- **Safe**: read-only, no server state change (GET, HEAD, OPTIONS).
- **Idempotent**: N identical calls leave the server in the same state as 1 call.

| Method | Purpose | Safe | Idempotent | Body |
|--------|---------|------|-----------|------|
| GET | Retrieve a resource | Yes | Yes | No |
| HEAD | GET headers only (existence, size) | Yes | Yes | No |
| OPTIONS | Discover allowed methods / CORS preflight | Yes | Yes | No |
| POST | Create resource / non-idempotent action | No | No | Yes |
| PUT | Replace resource entirely (create at known URI) | No | Yes | Yes |
| PATCH | Partial update | No | No* | Yes |
| DELETE | Remove resource | No | Yes | Optional |

\*PATCH can be made idempotent depending on semantics (e.g. setting absolute fields) but isn't guaranteed.

Why idempotency matters: networks retry. A client that times out on POST cannot safely retry — it may double-charge. Solution: **idempotency keys** — the client sends a unique `Idempotency-Key` header; the server records the first result and replays it for duplicates. PUT/DELETE need no key because they're idempotent by definition.

## HTTP Status Codes That Matter

| Code | Meaning | Typical use |
|------|---------|-------------|
| 200 OK | Success with body | GET/PUT/PATCH success |
| 201 Created | Resource created | POST that creates; return `Location` header |
| 202 Accepted | Queued, processing async | Long-running jobs |
| 204 No Content | Success, empty body | DELETE, PUT with nothing to return |
| 301 Moved Permanently | Permanent redirect | URL restructuring; clients update bookmarks |
| 302 Found | Temporary redirect | Transient relocation |
| 304 Not Modified | Cache still valid | Conditional GET with `If-None-Match`/`ETag` |
| 400 Bad Request | Malformed request | Validation failure, bad JSON |
| 401 Unauthorized | Not authenticated | Missing/invalid credentials (really "unauthenticated") |
| 403 Forbidden | Authenticated but not allowed | Authorization failure |
| 404 Not Found | Resource absent | Unknown URI (or hide existence on purpose) |
| 405 Method Not Allowed | Method unsupported here | POST to a read-only endpoint |
| 409 Conflict | State conflict | Duplicate create, optimistic-lock version mismatch |
| 422 Unprocessable Entity | Semantically invalid | Well-formed but business-rule violation |
| 429 Too Many Requests | Rate limited | Include `Retry-After` |
| 500 Internal Server Error | Unhandled server fault | Bug/exception — never leak stack traces |
| 502 Bad Gateway | Upstream returned garbage | Proxy/gateway can't parse backend |
| 503 Service Unavailable | Down/overloaded | Maintenance, shedding load; send `Retry-After` |
| 504 Gateway Timeout | Upstream too slow | Backend exceeded gateway timeout |

Discipline: 4xx = client's fault (don't retry without changing the request); 5xx = server's fault (safe to retry with backoff).

## HTTP/1.1 vs HTTP/2 vs HTTP/3

| Aspect | HTTP/1.1 (1997) | HTTP/2 (2015) | HTTP/3 (2022) |
|--------|-----------------|---------------|---------------|
| Transport | TCP | TCP | QUIC (over UDP) |
| Framing | Text | Binary | Binary |
| Concurrency | 1 request/connection (pipelining broken) | Multiplexed streams over 1 TCP conn | Multiplexed streams over QUIC |
| Head-of-line blocking | App-level (per connection) | Removed at HTTP layer, **remains at TCP layer** | Removed entirely (per-stream loss isolation) |
| Header compression | None (verbose) | HPACK | QPACK |
| Server push | No | Yes (deprecated in practice) | No |
| Handshake | TCP + separate TLS | TCP + TLS | Combined QUIC+TLS 1.3 (0-RTT possible) |

What actually makes each faster:
- **HTTP/1.1 → /2**: one connection carries many interleaved requests (multiplexing), and headers are compressed. Eliminates the "6 connections per origin" hack and redundant header bytes.
- **HTTP/2 → /3**: HTTP/2 still rides TCP, so a single lost packet stalls *all* streams (TCP head-of-line blocking). QUIC moves multiplexing into the transport, so packet loss only stalls the affected stream. QUIC also merges transport + TLS handshakes (faster connect) and supports **connection migration** — a phone switching Wi-Fi→cellular keeps the same connection ID instead of reconnecting.

## HTTP Headers Worth Knowing

- **Auth/identity**: `Authorization: Bearer <token>`, `WWW-Authenticate`, `Set-Cookie`/`Cookie`.
- **Content negotiation**: `Accept`, `Content-Type`, `Accept-Encoding`/`Content-Encoding` (gzip, br), `Accept-Language`.
- **Caching**: `Cache-Control` (`no-store`, `max-age`, `private`), `ETag`, `If-None-Match`, `Last-Modified`, `If-Modified-Since`, `Vary`.
- **Rate limiting**: `Retry-After`, plus de-facto `X-RateLimit-Limit/Remaining/Reset`.
- **CORS**: `Origin`, `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`.
- **Security**: `Strict-Transport-Security` (HSTS), `Content-Security-Policy`, `X-Content-Type-Options: nosniff`.
- **Tracing/proxy**: `X-Forwarded-For`, `X-Forwarded-Proto`, `Forwarded`, `X-Request-Id`, `Host`.

### URL / URI / URN

```
                              URI
   ┌───────────────────────────┴────────────────────────────┐
   URL (locates + how to access)                  URN (names only)
   https://api.example.com:443/v1/users?role=admin#bio       urn:isbn:0451450523
   └─┬─┘   └──────┬───────┘└┬┘└───┬───┘└────┬────┘└┬┘
  scheme       host       port  path     query   fragment
```

- **URI**: superset — any identifier (name and/or location).
- **URL**: a URI that says *where* a resource is and *how* to reach it (scheme matters).
- **URN**: a URI that *names* a resource persistently without saying where it lives (`urn:isbn:...`).
Every URL is a URI; not every URI is a URL.

## Real-Time Delivery Options

| Technique | Direction | Connection | Latency | Server cost | Use when |
|-----------|-----------|------------|---------|-------------|----------|
| Short polling | Client pulls | New request each tick | Poll interval | Wasteful (empty responses) | Simple, infrequent updates |
| Long polling | Client pulls | Held open until data/timeout | Near real-time | Held connections | Push needed, no WS support |
| SSE | Server → client | One long-lived HTTP stream | Real-time | 1 conn/client | Server-push feeds, dashboards, notifications |
| WebSocket | Bidirectional | Persistent TCP (HTTP upgrade) | Real-time | 1 conn/client | Chat, games, collaborative editing |
| Webhooks | Server → server | Provider POSTs to your URL | Event-driven | None until event | Cross-system events, integrations |

```
Short poll:   C →req→ S (no data)   ...wait...   C →req→ S (data!)
Long poll:    C →req→ S [held...] →resp(data)→ C  →req→ S [held...]
SSE:          C →req→ S ═════ stream: event, event, event ═════►
WebSocket:    C ⇄ S  full-duplex frames both directions
Webhook:      Event! → Provider →POST→ Your endpoint
```

SSE is text-only, auto-reconnects, and rides plain HTTP (firewall-friendly); pick it over WebSocket when you only need server→client. Webhooks need a public HTTPS endpoint, signature verification, and idempotent handlers (providers retry and may deliver duplicates/out-of-order).

## Pagination Strategies

| Strategy | How | Pros | Cons |
|----------|-----|------|------|
| Offset/limit | `?limit=20&offset=40` | Trivial; random page access | Slow at deep offsets; items shift/skip on inserts |
| Cursor/keyset | `?limit=20&after=<id_or_ts>` | Stable under writes; fast (uses index) | No jump-to-page; needs a sortable unique key |
| Page token | `?pageSize=20&pageToken=<opaque>` | Hides cursor internals; provider can evolve it | Opaque; can't be constructed by client |

Keyset example — query `WHERE (created_at, id) < (:last_ts, :last_id) ORDER BY created_at DESC, id DESC LIMIT 20`. This stays O(log n) regardless of depth, unlike `OFFSET 100000` which scans and discards rows. Default to **cursor/keyset for large or fast-changing datasets**; offset is fine for small, stable lists with a UI page-picker.

## API Versioning

| Approach | Example | Pros | Cons |
|----------|---------|------|------|
| URI path | `GET /v2/users` | Obvious, cache-friendly, easy routing | Couples version to URL; "v2" everywhere |
| Header | `Accept: application/vnd.example.v2+json` | Clean URLs; content-negotiation native | Harder to test/curl; opaque to caches |
| Query param | `GET /users?version=2` | Easy to try | Clutters params; weak cache semantics |

Tips for safe evolution:
- Prefer **additive, backward-compatible** changes (new optional fields) over breaking ones — version only when you must break.
- Never change the meaning/type of an existing field; never remove one without a deprecation window.
- Treat unknown fields as ignorable on both sides (tolerant reader).
- Publish a **deprecation policy**: `Deprecation` and `Sunset` headers, changelog, migration guide.
- Default to **URI versioning for public APIs** (discoverable); reserve major versions for true breaks.

## Load Balancer vs Reverse Proxy vs API Gateway

| | Load Balancer | Reverse Proxy | API Gateway |
|--|--------------|---------------|-------------|
| Primary job | Distribute traffic across servers | Front backends; terminate TLS, cache, hide topology | API-aware entry point for many services |
| Layer | L4 (TCP) or L7 (HTTP) | L7 | L7, application-aware |
| Knows about | Servers/health | HTTP requests | Routes, auth, rate limits, API contracts |
| Typical features | Round-robin, least-conn, health checks | TLS termination, compression, static cache | AuthN/Z, rate limiting, request transform, aggregation, metering |

These overlap; one box can play several roles. A gateway is essentially a reverse proxy with API-management features bolted on.

### What an API Gateway Actually Does

A single front door between clients and your backend services:
- **Routing**: map `/orders/*` → orders service, `/users/*` → users service.
- **AuthN/AuthZ**: validate JWTs/API keys before traffic reaches services.
- **Rate limiting & quotas**: per-key throttling, burst control.
- **Transformation**: protocol/version translation (e.g. REST→gRPC), header rewriting.
- **Aggregation**: fan out one client call to several services, combine responses.
- **Cross-cutting concerns**: TLS termination, caching, observability (logs/metrics/traces), circuit breaking.

It centralizes concerns so each microservice doesn't re-implement auth and rate limiting.

### Proxy vs Reverse Proxy (Forward vs Reverse)

```
Forward proxy:   [Clients] → (Proxy) → Internet      # sits in front of CLIENTS
Reverse proxy:   Internet → (Reverse Proxy) → [Servers]   # sits in front of SERVERS
```

- **Forward proxy**: acts on behalf of clients — egress control, anonymity, corporate filtering, caching outbound requests. The destination server sees the proxy, not the client.
- **Reverse proxy**: acts on behalf of servers — clients think they're talking to one host; it load-balances, terminates TLS, caches, and shields the real backends. The client never sees individual servers.

## API Security Essentials Checklist

- **TLS everywhere**: HTTPS only; enforce HSTS; redirect HTTP→HTTPS; modern cipher suites.
- **Authentication**: OAuth2/OIDC, short-lived JWTs, or API keys for service-to-service. Rotate secrets; never embed long-lived keys in clients.
- **Authorization**: enforce per-request (RBAC/ABAC); check object-level ownership to prevent IDOR/BOLA (`/orders/42` must belong to the caller).
- **Least privilege**: scoped tokens, minimal DB grants, separate read/write credentials.
- **Rate limiting & throttling**: per-key and per-IP; protect against brute force and DoS; return 429 + `Retry-After`.
- **Input validation**: validate/whitelist all input server-side; parameterized queries (no SQL/command injection); cap payload sizes; validate content types.
- **Output hygiene**: don't leak stack traces, internal IDs, or PII in errors; consistent error envelope.
- **Transport & headers**: CORS allowlist (no wildcard with credentials); `nosniff`, CSP, no sensitive data in URLs (they hit logs).
- **Secrets management**: vault/KMS, not source control or env files in repos.
- **Audit & monitoring**: log auth events, anomalies, and abuse; alert on spikes.

## API vs SDK

- **API** (Application Programming Interface): the *contract* — endpoints, methods, payloads, and rules a service exposes. Language-agnostic; you call it over the wire (HTTP, gRPC).
- **SDK** (Software Development Kit): a *toolkit* — libraries, helpers, auth handling, types, and docs that wrap an API for a specific language so you don't hand-roll HTTP calls. An SDK is a convenience layer *on top of* an API; the API is the source of truth.

## 9 Types of API Testing

1. **Smoke** — does the API come up and respond at all? (sanity baseline).
2. **Functional** — endpoints return correct results for valid inputs per spec.
3. **Integration** — multiple APIs/services work together end-to-end.
4. **Regression** — existing behavior still holds after changes.
5. **Load** — performance under expected concurrent traffic.
6. **Stress** — behavior beyond capacity; find the breaking point and failure mode.
7. **Security** — authn/authz, injection, data exposure, rate-limit enforcement.
8. **UI/end-to-end** — API behaves correctly as consumed by the actual frontend flow.
9. **Fuzz** — feed malformed/random/boundary input to surface crashes and unhandled cases.
