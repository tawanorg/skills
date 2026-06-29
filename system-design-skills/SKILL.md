---
name: system-design-skills
description: >
  A practical encyclopedia of system design building blocks — APIs, databases,
  caching, scalability, distributed systems, architecture, DevOps/CI-CD,
  security, networking, payments, and AI systems. Use when designing or
  reviewing a backend/web system, choosing between technologies, preparing for
  a system design interview, or when asked about "system design", "how does X
  work", "REST vs GraphQL", "SQL vs NoSQL", "how to scale", "caching", "OAuth",
  "microservices", or similar fundamentals.
license: MIT
metadata:
  author: tawanorg
  version: '1.0.0'
---

# System Design Skills

A self-contained reference map of the building blocks behind modern systems —
the protocols, data stores, patterns, and trade-offs you assemble into a design,
plus the heuristics for choosing between them. It is the encyclopedia of
components you design with, paired with rules for applying them.

Topic coverage and structure mirror the categories popularized by ByteByteGo's
"System Design 101" (used here purely as a topic map — all content is original,
synthesized, and vendor-neutral).

## How to Use This Skill

1. **Designing something new?** Start with `references/system-design-interview.md`
   for the end-to-end framework (clarify → estimate → API → data → scale), then
   pull in the component references below as needed.
2. **Need depth on one component?** Jump straight to the matching reference file.
3. **Reviewing a design or PR?** Use the **Rules** as a checklist of the
   highest-leverage heuristics.
4. **Choosing between options?** Lean on the **Rules** below and the trade-off
   tables in `references/software-architecture.md` and
   `references/scalability-and-distributed-systems.md`.

## The Building Blocks at a Glance

A typical web-scale request flows through layers, each a design decision:

```
Client ─► DNS ─► CDN ─► Load Balancer ─► API Gateway ─► Stateless App Servers
                                                              │
                          ┌───────────────┬───────────────┬──┴────────────┐
                          ▼               ▼               ▼               ▼
                       Cache          Primary DB      Message Queue    Object Store
                     (Redis)        (+ replicas,     (async work,     (S3/blobs)
                                      sharded)         events)
                          │               │
                          ▼               ▼
                     Search Index    Analytics / OLAP
```

Every arrow is a trade-off. This skill covers what lives at each box and how to
choose.

## Topic Map → Reference Files

| Area | What it covers | Reference |
|------|----------------|-----------|
| **APIs & Web** | REST/GraphQL/gRPC, HTTP versions, status codes, gateways, proxies, pagination, real-time delivery | `references/api-and-web-development.md` |
| **Databases & Storage** | SQL vs NoSQL, indexing, replication, sharding, ACID/isolation, consistency | `references/databases-and-storage.md` |
| **Caching & Performance** | Cache patterns, invalidation, eviction, CDNs, stampede defense, latency budgets | `references/caching-and-performance.md` |
| **Scalability & Distributed** | Horizontal scaling, CAP/PACELC, consensus, sagas, queues, rate limiting, resilience | `references/scalability-and-distributed-systems.md` |
| **Software Architecture** | Monolith→microservices, event-driven, CQRS, DDD, patterns, 12-factor | `references/software-architecture.md` |
| **DevOps & CI/CD** | Pipelines, deployment strategies, containers/K8s, IaC, SLI/SLO, DORA | `references/devops-and-cicd.md` |
| **Security & Auth** | Sessions/cookies, JWT, OAuth2/OIDC, SSO, TLS, RBAC, OWASP, secrets | `references/security-and-auth.md` |
| **Networking & Fundamentals** | OSI/TCP-IP, TCP vs UDP, DNS, IP/NAT, ports, processes/threads, deadlock | `references/networking-and-computer-fundamentals.md` |
| **Payments & Fintech** | Card flow, ledgers/double-entry, idempotency, reconciliation, PCI | `references/payment-and-fintech.md` |
| **AI & ML Systems** | LLMs/agents, RAG, vector DBs, data pipelines, LLM app architecture | `references/ai-and-ml-systems.md` |
| **Case Studies** | Distilled, reusable lessons from how large systems scaled | `references/real-world-case-studies.md` |
| **Interview Framework** | A repeatable approach + capacity estimation + common prompts | `references/system-design-interview.md` |

## Core Heuristics (the 30-second version)

- **Start simple.** A well-structured monolith beats premature microservices.
  Add complexity only when a *measured* bottleneck demands it.
- **Stateless scales.** Push session/state to a cache or DB so app servers can
  scale horizontally behind a load balancer.
- **Cache reads, queue writes.** Read-heavy paths → cache aggressively (with a
  TTL and invalidation plan). Spiky/expensive writes → async via a queue.
- **Choose storage by access pattern,** not familiarity. The query shape and
  consistency need drive the choice, not "we always use X".
- **Make writes idempotent.** Networks retry. Idempotency keys turn "did it go
  through?" from a bug into a non-event.
- **Design for failure.** Timeouts, retries-with-backoff, circuit breakers,
  graceful degradation. In distributed systems, failure is normal.
- **Pick a consistency model per use case.** Money → strong/CP. Feeds, likes,
  analytics → eventual/AP.
- **Secure every endpoint.** Authenticate, authorize, validate input, rate
  limit, and use TLS — on *every* request, not just the front door.
- **Instrument everything.** Logs, metrics, traces. You can't scale or debug
  what you can't see.

## Rules

Atomic decision heuristics. Each rule follows: Context → Heuristic → Why.

| Rule | File |
|------|------|
| Version Your APIs | `rules/api-version-your-apis.md` |
| Paginate Large Collections | `rules/api-paginate-large-collections.md` |
| Make Writes Idempotent | `rules/api-idempotent-writes.md` |
| Use Correct HTTP Status Codes | `rules/api-use-correct-status-codes.md` |
| Choose Storage by Access Pattern | `rules/data-choose-storage-by-access-pattern.md` |
| Index Your Query Paths | `rules/data-index-your-query-paths.md` |
| Define a TTL and Invalidation Plan | `rules/cache-define-ttl-and-invalidation.md` |
| Prevent Cache Stampede | `rules/cache-prevent-stampede.md` |
| Keep Services Stateless | `rules/scale-keep-services-stateless.md` |
| Design for Failure | `rules/scale-design-for-failure.md` |
| Embrace Async and Queues | `rules/scale-embrace-async-and-queues.md` |
| Pick Consistency Per Use Case | `rules/scale-pick-consistency-per-use-case.md` |
| Never Store Plaintext Secrets | `rules/security-never-store-plaintext-secrets.md` |
| Authorize and Validate Every Request | `rules/security-authorize-every-request.md` |
| Rate Limit Public Endpoints | `rules/operations-rate-limit-public-endpoints.md` |
| Instrument Everything | `rules/operations-instrument-everything.md` |

## Reference Files

Deep dives on each area.

| Topic | File |
|-------|------|
| API and Web Development | `references/api-and-web-development.md` |
| Databases and Storage | `references/databases-and-storage.md` |
| Caching and Performance | `references/caching-and-performance.md` |
| Scalability and Distributed Systems | `references/scalability-and-distributed-systems.md` |
| Software Architecture | `references/software-architecture.md` |
| DevOps and CI/CD | `references/devops-and-cicd.md` |
| Security and Authentication | `references/security-and-auth.md` |
| Networking and Computer Fundamentals | `references/networking-and-computer-fundamentals.md` |
| Payment and Fintech | `references/payment-and-fintech.md` |
| AI and ML Systems | `references/ai-and-ml-systems.md` |
| Real-World Case Studies | `references/real-world-case-studies.md` |
| System Design Interview | `references/system-design-interview.md` |

## Credits

Topic taxonomy inspired by [ByteByteGo's System Design 101](https://github.com/ByteByteGoHq/system-design-101).
All explanatory content here is original and independently written.
