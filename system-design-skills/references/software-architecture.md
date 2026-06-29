# Software Architecture

A working reference for choosing and evolving the high-level structure of a system: the
styles, the trade-offs, and the patterns that show up repeatedly. The recurring lesson is
that architecture is the set of decisions that are expensive to reverse, so defer the ones
you can and make the rest explicit.

## Architecture Styles

| Style | Unit of deployment | Best for | Avoid when |
|---|---|---|---|
| Monolith | One process/artifact | New products, small teams, unproven domains | Many teams blocked on one release train; wildly uneven scaling needs |
| Modular monolith | One artifact, enforced internal modules | Most systems; teams wanting clean boundaries without network cost | You genuinely need independent deploy/scale per module |
| Microservices | Many independently deployed services | Large orgs, independent scaling, strong team autonomy | Small teams, immature CI/CD or observability, fuzzy domain boundaries |
| Service-oriented (SOA) | Coarse services + shared middleware | Enterprise integration across heterogeneous systems | You want lightweight services without an ESB/governance layer |
| Event-driven | Producers/consumers over a broker | Reactive workflows, decoupling, fan-out, ingest pipelines | Strict end-to-end consistency or simple request/response needs |
| Serverless (FaaS) | Functions + managed services | Spiky/low traffic, glue code, event handlers, fast time-to-market | Steady high throughput, long jobs, strict latency, heavy local state |
| Micro-frontends | Independently deployed UI fragments | Many teams shipping into one web app | Single small frontend team; tight cross-UI consistency requirements |

Styles compose. A modular monolith can emit events; microservices can sit behind a BFF;
serverless functions can subscribe to a broker. Pick per concern, not per religion.

## Monolith First

> Start with a monolith. Extract services only when the pain of *not* having them exceeds
> the distributed-system tax.

Early on you do not yet understand the domain, so any service boundary you draw will be
wrong — and a wrong boundary across a network is far costlier to move than a wrong boundary
across a function call. Build the monolith with internal modularity so that extraction is
cheap later.

```
Monolith                Modular Monolith              Microservices
+-----------+           +-----------------+           +-----+ +-----+ +-----+
|  tangled  |  refactor | [Ord][Pay][Inv] |  extract  | Ord | | Pay | | Inv |
|   code    | ───────►  | clear modules,  | ───────►  +-----+ +-----+ +-----+
|           |           | one deployable  |           own DB   own DB  own DB
+-----------+           +-----------------+           independent deploy
```

The middle stage is the high-leverage one: most of the design benefit of microservices
(clear ownership, explicit interfaces, testable seams) comes from modularization, *without*
the operational tax. Only split out a module when it has a distinct scaling profile,
release cadence, team, or technology need.

## Microservices

**Benefits:** independent deploy and scale per service, team autonomy, fault isolation,
technology heterogeneity, and bounded blast radius for changes.

**Costs (the distributed-system tax):** network calls fail and add latency; debugging spans
many processes; transactions become sagas; data is eventually consistent; you now run
service discovery, distributed tracing, centralized logging, and contract testing. A method
call that was nanoseconds and always-succeeded becomes a milliseconds call that can time
out, retry, and partially fail.

**Prerequisites — do not start microservices without these:**
- **Team autonomy:** independent teams owning services end-to-end (you ship your org chart — Conway's Law).
- **CI/CD maturity:** automated build, test, and independent deploy per service.
- **Observability:** centralized logs, metrics, distributed tracing, health checks, alerting.
- **Operational platform:** containerization/orchestration, automated provisioning, secret management.

**When NOT to use:** small team, early-stage product, undiscovered domain boundaries,
no automated deployment, or a problem that a modular monolith solves with far less ceremony.
"Distributed monolith" — services that must deploy together — is the worst of both worlds.

## Inter-Service Communication

**Synchronous (REST, gRPC):** caller blocks for a response. Simple mental model; good for
queries and request/response. Couples availability — if a downstream is down, you are down
unless you add timeouts, retries with backoff, and circuit breakers. gRPC adds typed
contracts (protobuf), streaming, and lower overhead than JSON/REST.

**Asynchronous (events, queues):** producer emits and moves on; consumers process later via
a broker (Kafka, RabbitMQ, SQS). Decouples availability and smooths load spikes, at the cost
of eventual consistency and harder end-to-end reasoning. Prefer async for workflows that
tolerate delay and for fan-out.

**API composition / aggregation:** one request needs data from several services. Either an
aggregator service / gateway fans out and joins, or you maintain a read model (CQRS) updated
by events. Naive in-request fan-out couples latency to your slowest dependency.

**Service mesh (sidecar):** a proxy (e.g. Envoy) deployed beside each service handles
retries, mTLS, traffic shaping, and telemetry, moving cross-cutting network concerns out of
app code. Useful at scale; overkill for a handful of services.

```
   sync                          async
A ──REST/gRPC──► B          A ──emit──►[ broker ]──► C
   (A waits, coupled)          (A done; C consumes later, decoupled)
```

## Event-Driven Architecture

**Events vs commands.** A *command* is an instruction to do something (`ReserveInventory`) —
directed, may be rejected, expects an actor. An *event* is a fact that already happened
(`OrderPlaced`) — past tense, immutable, broadcast to whoever cares. Commands couple sender
to receiver; events invert that dependency.

**Three ways to use events:**
- **Event notification:** event carries minimal data ("order 123 changed"); consumers call
  back for details. Low coupling, but chatty and adds load on the source.
- **Event-carried state transfer:** event carries the full payload so consumers keep their
  own local copy and need no callback. Decoupled and fast to read, but data is duplicated
  and eventually consistent.
- **Event sourcing:** the event log *is* the source of truth; current state is a fold over
  events. Gives a full audit trail and temporal queries (replay to any point), at the cost
  of schema-evolution and snapshotting complexity.

**CQRS** (Command Query Responsibility Segregation): separate the write model from one or
more read models optimized for queries, often kept in sync via events. Pairs naturally with
event sourcing but is independent of it. Use only where read and write needs genuinely
diverge — it doubles your models.

**Eventual consistency** is the price of decoupling: after a write, replicas/read models
converge *eventually*, not instantly. Design UIs and APIs to tolerate stale reads, make
consumers **idempotent** (events get redelivered), and never assume ordering you did not
explicitly guarantee.

## Domain-Driven Design (quick reference)

- **Ubiquitous language:** one shared vocabulary per domain, used identically in code,
  conversation, and schema. Ambiguity in language is ambiguity in the model.
- **Bounded context:** an explicit boundary within which a model and its language are
  consistent. "Customer" in Billing ≠ "Customer" in Support; each context owns its meaning.
- **Aggregate:** a cluster of objects treated as one consistency unit, mutated only through
  its **root**. Keep aggregates small; one transaction touches one aggregate.
- **Domain event:** something meaningful that happened in the domain, named in the past
  tense — the natural integration point between contexts.

**Finding service boundaries:** align services with bounded contexts. Aim for **high
cohesion** (things that change together live together) and **low coupling** (a change to one
service rarely forces a change to another). A boundary that produces chatty cross-service
calls or shared mutable data is in the wrong place — usually you split an aggregate that
should have stayed whole.

## Common Architectural Patterns

- **Layered (n-tier):** presentation → application → domain → data. Familiar and simple;
  risks the domain depending on infrastructure and turning into anemic CRUD.
- **Hexagonal (ports & adapters):** domain at the center exposes *ports* (interfaces);
  *adapters* implement them for HTTP, DB, queues. Dependencies point inward; infrastructure
  is swappable and the core is testable in isolation.
- **Clean architecture:** concentric layers (entities → use cases → interface adapters →
  frameworks) with the **dependency rule** — source dependencies point only inward. Same
  spirit as hexagonal, more prescriptive.
- **BFF (backend-for-frontend):** a tailored backend per client type (web, mobile, partner)
  that aggregates and shapes data for that client, keeping each from carrying others' baggage.
- **Strangler fig:** migrate incrementally by routing slices of traffic from the legacy
  system to new implementations behind a façade, shrinking the old system until it's gone —
  no big-bang rewrite.
- **Saga:** a long-running business transaction as a sequence of local transactions, each
  with a compensating action to undo on failure. **Orchestrated** (a coordinator drives
  steps) or **choreographed** (services react to each other's events). Replaces distributed
  ACID transactions across services.
- **Sidecar:** a helper process deployed alongside a service for cross-cutting concerns
  (proxy, logging, config) — the building block of a service mesh.
- **Ambassador:** a sidecar that brokers *outbound* calls (retries, TLS, routing) so app
  code talks to a local proxy instead of remote endpoints directly.

```
Ports & Adapters:           HTTP ─┐                ┌─ Postgres
                                  ├─►[ DOMAIN ]◄───┤
                          Queue ──┘  (no infra      └─ S3
                                      deps inside)
```

## The 12-Factor App (concise)

1. **Codebase** — one repo tracked in version control, many deploys.
2. **Dependencies** — declare and isolate explicitly; no system-wide assumptions.
3. **Config** — store in the environment, never in code.
4. **Backing services** — treat DBs, queues, caches as attachable resources via URL/creds.
5. **Build, release, run** — strictly separate the three stages.
6. **Processes** — run as stateless, share-nothing processes; persist state in backing services.
7. **Port binding** — export services by binding to a port; be self-contained.
8. **Concurrency** — scale out via the process model (more processes, not bigger ones).
9. **Disposability** — fast startup, graceful shutdown; tolerate sudden death.
10. **Dev/prod parity** — keep environments as similar as possible.
11. **Logs** — emit to stdout as an event stream; let the platform route them.
12. **Admin processes** — run one-off tasks in an identical environment.

## Data Ownership

- **Database-per-service:** each service privately owns its data; others reach it only
  through its API or its events. This is what makes independent deploy and schema change
  possible.
- **Avoid shared databases:** a database touched by multiple services is hidden coupling —
  one team's schema change silently breaks another, and nobody truly owns the data. It is the
  most common reason microservices fail to deliver autonomy.
- **Anti-corruption layer (ACL):** a translation layer at a context boundary (especially
  against a legacy or third-party model) that maps foreign concepts into your own model so
  their design does not leak into and corrupt yours.

```
Order Svc ─owns─► [Order DB]      Pay Svc ─owns─► [Pay DB]
        ▲                                 ▲
        └── API / events only ────────────┘   (never reach into each other's DB)
```

## Cross-Cutting Concerns

- **Config:** externalize from artifacts (env vars / config service); same build promotes
  across environments. Validate config at startup and fail fast on missing values.
- **Secrets:** never in code or images; use a secrets manager / vault with rotation and
  least-privilege access. Inject at runtime, audit access.
- **Feature flags:** decouple deploy from release; enable canary, kill-switch, and A/B.
  Keep flags short-lived and remove them — stale flags become permanent hidden branches.
- **Idempotency:** make retried/redelivered operations safe via idempotency keys or natural
  dedup, so at-least-once delivery and client retries don't double-charge or double-create.
- **API versioning:** evolve contracts without breaking clients — additive changes only
  within a version; version via URL path, header, or media type; deprecate on a published
  timeline. Treat published contracts as promises.

## Trade-offs

Every architecture decision is a position on these axes; "good architecture" is choosing the
right point for *this* system at *this* time, then revisiting as it changes.

- **Coupling vs autonomy:** loose coupling buys independent change and deploy, but pays in
  duplication, eventual consistency, and operational moving parts. Tight coupling is simpler
  and consistent until teams contend on one release train.
- **Consistency vs availability:** under partition (CAP) you choose. Strong consistency
  simplifies reasoning but limits availability and adds latency; high availability with
  eventual consistency scales and survives partitions but pushes reconciliation onto you.
- **Simplicity vs flexibility:** abstraction and indirection enable future change but add
  cognitive load now. Add flexibility when the change is likely and the option is cheap;
  otherwise prefer the simple thing you can refactor later.

**Heuristic:** optimize for the cost of *change*. The best architecture is the one that makes
tomorrow's most-likely change cheap, while keeping today's system small enough to understand.
