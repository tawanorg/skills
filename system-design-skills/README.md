# System Design Skills

A practical encyclopedia of the building blocks behind modern systems. Helps AI
agents (and humans) recall how each component works and which trade-offs apply
when assembling a design.

A self-contained reference covering the components you design with and the
heuristics for choosing between them. Topic taxonomy is inspired by ByteByteGo's
[System Design 101](https://github.com/ByteByteGoHq/system-design-101); all
explanatory content here is original and vendor-neutral.

## Structure

```
system-design-skills/
├── SKILL.md                                          # Main skill file (agent reads this)
├── metadata.json                                     # Version and reference info
├── README.md                                         # This file (human documentation)
├── rules/                                            # Decision heuristics
│   ├── _sections.md                                  # Category definitions
│   ├── api-version-your-apis.md
│   ├── api-paginate-large-collections.md
│   ├── api-idempotent-writes.md
│   ├── api-use-correct-status-codes.md
│   ├── data-choose-storage-by-access-pattern.md
│   ├── data-index-your-query-paths.md
│   ├── cache-define-ttl-and-invalidation.md
│   ├── cache-prevent-stampede.md
│   ├── scale-keep-services-stateless.md
│   ├── scale-design-for-failure.md
│   ├── scale-embrace-async-and-queues.md
│   ├── scale-pick-consistency-per-use-case.md
│   ├── security-never-store-plaintext-secrets.md
│   ├── security-authorize-every-request.md
│   ├── operations-rate-limit-public-endpoints.md
│   └── operations-instrument-everything.md
└── references/                                       # Deep dives
    ├── api-and-web-development.md                     # REST/GraphQL/gRPC, HTTP, gateways
    ├── databases-and-storage.md                      # SQL/NoSQL, indexing, replication, sharding
    ├── caching-and-performance.md                    # Cache patterns, CDNs, stampede defense
    ├── scalability-and-distributed-systems.md        # CAP, consensus, sagas, queues, resilience
    ├── software-architecture.md                      # Monolith→microservices, EDA, DDD, patterns
    ├── devops-and-cicd.md                             # Pipelines, deploys, containers, SLO/DORA
    ├── security-and-auth.md                           # Sessions, JWT, OAuth2/OIDC, TLS, OWASP
    ├── networking-and-computer-fundamentals.md        # OSI, TCP/UDP, DNS, NAT, OS basics
    ├── payment-and-fintech.md                         # Card flow, ledgers, idempotency, PCI
    ├── ai-and-ml-systems.md                           # LLMs, agents, RAG, vector DBs, pipelines
    ├── real-world-case-studies.md                     # Distilled scaling lessons
    └── system-design-interview.md                     # End-to-end framework + estimation
```

## Core Concepts

### The Building Blocks

A web-scale request passes through layers, each a design decision: DNS → CDN →
load balancer → API gateway → stateless app servers → cache / database (+
replicas, sharding) / message queue / object store / search / analytics. This
skill documents what lives at each layer and how to choose.

### How to Use

1. **Designing something new?** Start with `references/system-design-interview.md`
   for the framework, then pull in component references as needed.
2. **Need depth on one topic?** Open the matching reference file.
3. **Reviewing a design or PR?** Use the **rules** as a checklist.
4. **Choosing between options?** Use the **rules** and the trade-off tables in
   the architecture and scalability references.

### The 16 Rules

Atomic, high-leverage heuristics across six categories — API Design, Data &
Storage, Caching, Scalability & Distributed Systems, Security, and Operations.
Each follows **Context → Heuristic → Why** with a concrete example. See
`rules/_sections.md` for the category definitions.

## What's Inside

| Category | Content |
|----------|---------|
| APIs & Web | REST/GraphQL/gRPC/WebSocket, HTTP/1-2-3, status codes, gateway vs proxy vs LB, pagination, versioning |
| Databases & Storage | SQL vs NoSQL, indexing, ACID/isolation, replication, sharding, consistency, OLTP vs OLAP |
| Caching & Performance | cache-aside/write-through, eviction, CDNs, stampede/penetration defense, latency budgets |
| Scalability & Distributed | horizontal scaling, CAP/PACELC, quorums, consensus, sagas, queues, rate limiting, resilience |
| Software Architecture | monolith→microservices, event-driven, CQRS, DDD, hexagonal/clean, 12-factor |
| DevOps & CI/CD | pipelines, blue-green/canary, Docker/K8s, IaC/GitOps, SLI/SLO, golden signals, DORA |
| Security & Auth | sessions/cookies, JWT, OAuth2/OIDC, SSO, TLS/mTLS, RBAC/ABAC, OWASP, secrets |
| Networking & Fundamentals | OSI/TCP-IP, TCP vs UDP, DNS, IP/NAT, ports, processes/threads, deadlock |
| Payments & Fintech | auth→capture→settlement, double-entry ledgers, idempotency, reconciliation, PCI |
| AI & ML Systems | LLMs/transformers, agents, RAG, vector search, data pipelines, LLM app architecture |
| Case Studies | reusable patterns distilled from large-scale systems |
| Interview Framework | requirements → estimation → API → data → scale, plus common prompts |

## Triggers on

- System design and architecture discussions
- "How does X work?" fundamentals (REST vs GraphQL, SQL vs NoSQL, OAuth, DNS, …)
- Choosing between technologies / storage / protocols
- Scaling, caching, and reliability decisions
- System design interview preparation

## Credits

Topic taxonomy inspired by [ByteByteGo's System Design 101](https://github.com/ByteByteGoHq/system-design-101)
(CC BY-NC-ND 4.0). No text or diagrams are reproduced from it — all content here
is independently written.
