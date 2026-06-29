# DevOps and CI/CD

DevOps is the practice of shortening the loop between writing code and running it reliably in production by automating build, test, release, and operations. CI/CD is the automated spine of that loop: every change is integrated, validated, and shipped through a repeatable pipeline rather than a manual hand-off.

## CI vs CD

Continuous Integration (CI) means every commit is merged to a shared mainline frequently and validated automatically (build + tests + lint). Continuous Delivery keeps the mainline *always releasable* — the pipeline produces a deployable artifact and pushes it through pre-prod environments, but the final production push is a human decision (a button). Continuous Deployment removes that button: every change that passes the gates ships to production automatically.

| Aspect | Continuous Integration | Continuous Delivery | Continuous Deployment |
| --- | --- | --- | --- |
| Scope | Merge + validate code | Always-releasable artifact through staging | Auto-release to prod |
| Prod release trigger | n/a | Manual approval | Fully automated |
| Human gate | Code review | Release approval | None (gates are automated) |
| Prerequisite | Fast test suite | Strong CI + env parity | Strong CD + canary/rollback + observability |
| Risk per release | n/a | Low (human checkpoint) | Lowest *per change* (small batches), needs guardrails |

Key idea: the two "CD"s share the same pipeline; the only difference is whether a person approves the last step. Continuous deployment is only safe when automated verification (canary analysis, health checks, rollback) is trustworthy.

## Anatomy of a CI/CD Pipeline

```
 commit/PR
    │
    ▼
┌────────┐   ┌───────┐   ┌──────┐   ┌─────────┐   ┌──────┐   ┌────────┐   ┌────────┐
│ SOURCE │──▶│ BUILD │──▶│ TEST │──▶│ PACKAGE │──▶│ SCAN │──▶│ DEPLOY │──▶│ VERIFY │
└────────┘   └───────┘   └──────┘   └─────────┘   └──────┘   └────────┘   └────────┘
 trigger      compile     unit/      artifact      SAST/      env rollout   smoke,
 lint         deps        integ/     image+tag     deps/      canary        health,
 secrets      cache       e2e        SBOM          secrets    flags         rollback?
```

- **Source** — A push or pull request triggers the pipeline. The exact commit SHA is captured so every downstream artifact is traceable. Pre-checks: formatting, lint, commit hygiene, secret scanning.
- **Build** — Compile code / resolve dependencies / build the container image. Use a clean, reproducible environment and a dependency cache for speed. Output is a single immutable artifact promoted unchanged through later stages ("build once, deploy many").
- **Test** — Layered gates: fast unit tests first (fail early), then integration tests against real dependencies (DB, queues), then slower end-to-end tests. Coverage and quality thresholds can block the merge.
- **Package / Artifact** — Produce a versioned, immutable artifact (container image, jar, wheel, binary) tagged with the commit SHA and pushed to a registry/artifact store. Never rebuild between environments — promote the *same* artifact.
- **Scan** — Static analysis (SAST), dependency/vulnerability scanning, license checks, container image scanning, IaC misconfig checks, and SBOM generation. Sign the artifact here.
- **Deploy** — Roll the artifact into an environment using a chosen strategy (rolling, canary, blue-green). Configuration and secrets are injected at this layer, not baked in.
- **Verify** — Post-deploy smoke tests, health checks, and metric/canary analysis. If signals are bad, roll back automatically. This stage closes the loop and is what makes continuous deployment safe.

## Deployment Strategies

| Strategy | How it works | Risk exposure | Rollback speed | Cost / complexity |
| --- | --- | --- | --- | --- |
| Recreate | Stop all old, start all new | High (downtime + full blast radius) | Slow (redeploy old) | Low |
| Rolling | Replace instances in batches | Medium (mixed versions live) | Medium (roll back batch by batch) | Low |
| Blue-green | Two full envs; switch traffic at once | Low-medium (instant cutover, full switch) | Instant (flip back to blue) | High (2x capacity) |
| Canary | Send small % to new version, ramp up | Low (limited blast radius) | Fast (route 100% back) | Medium (needs traffic routing + analysis) |
| Shadow / dark | Mirror real traffic to new version, discard responses | Very low (no user impact) | n/a (never served) | High (duplicate compute) |
| Feature flags | Ship code dark, toggle behavior per cohort | Very low (decouple deploy from release) | Instant (flip flag) | Medium (flag system + cleanup debt) |

Notes: rolling and canary both keep mixed versions running, so the app and database schema must be backward/forward compatible (expand-then-contract migrations). Blue-green gives the cleanest rollback but doubles infra. Shadow launches are ideal for validating performance/correctness of a rewrite without risk. Feature flags decouple *deploy* (code is in prod) from *release* (users see it) — the most powerful lever for progressive delivery, at the cost of flag debt that must be actively pruned.

## Progressive Delivery and Automated Rollback

Progressive delivery = ship to a growing slice of traffic while machines watch the metrics. A typical canary ramp: 1% → 5% → 25% → 50% → 100%, pausing at each step for an automated **canary analysis** that compares the canary's error rate, latency, and saturation against the stable baseline. If the canary degrades beyond a threshold, the pipeline halts and rolls back without a human.

Health signals come from probes:

- **Liveness probe** — "is the process wedged?" If it fails, the orchestrator restarts the container. Should test only the process itself, not downstream dependencies (or a DB outage causes a restart storm).
- **Readiness probe** — "can this instance serve traffic right now?" If it fails, the instance is pulled from the load balancer but not killed. Used during startup/warmup and transient dependency loss.
- **Startup probe** — guards slow-booting apps so liveness doesn't kill them before they finish initializing.

Rollback should be a first-class, rehearsed operation: keep the previous artifact and config one click (or one automated trigger) away. Forward-fix is fine for low-severity issues; for incidents, roll back first and diagnose later.

## Containers and Orchestration

A **Docker image** is a stack of read-only **layers** built from a Dockerfile; each instruction adds a layer, and layers are cached and shared between images (change a late layer, reuse the early ones). A **container** is a running instance of an image with a thin writable layer on top. Images are pushed to a **registry** (Docker Hub, ECR, GCR, GHCR) and pulled by runtime hosts. Best practices: small base images, multi-stage builds, pin versions, run as non-root, one process concern per image.

Kubernetes core objects (conceptual):

| Object | Role |
| --- | --- |
| **Pod** | Smallest deployable unit: one or more tightly-coupled containers sharing network/storage. Ephemeral. |
| **Deployment** | Declares desired replica count + image; manages rollouts and rollbacks via ReplicaSets. |
| **Service** | Stable virtual IP / DNS name load-balancing across a set of Pods (Pods come and go; the Service endpoint is stable). |
| **Ingress** | L7 HTTP routing (host/path rules, TLS) from outside the cluster to Services. |
| **ConfigMap** | Non-secret configuration injected as env vars or mounted files. |
| **Secret** | Same as ConfigMap but for sensitive data (base64-encoded, ideally encrypted at rest / sourced from a vault). |

Kubernetes runs a **reconciliation loop**: you declare desired state, controllers continuously drive actual state toward it. The **Horizontal Pod Autoscaler (HPA)** adds/removes Pod replicas based on observed metrics (CPU, memory, or custom/queue depth). Compare: the Vertical Pod Autoscaler resizes a Pod's resources, and the Cluster Autoscaler adds/removes nodes when Pods can't be scheduled.

## Infrastructure as Code (IaC)

IaC manages infrastructure through version-controlled files instead of console clicks, making environments reproducible, reviewable, and auditable.

- **Declarative** (Terraform, CloudFormation, Kubernetes manifests) — you describe the desired end state; the tool computes the diff and converges. Preferred for infra.
- **Imperative** (scripts, CLIs, Ansible tasks in step form) — you specify the sequence of actions. More control, less self-healing.
- **Idempotency** — applying the same definition repeatedly yields the same result; re-running is safe and a no-op if nothing changed. This is what makes declarative tooling reliable.

**Terraform** uses providers to manage resources and keeps a **state file** mapping config to real-world resources; `plan` shows the diff, `apply` enacts it. **Pulumi** offers the same model but defines infra in general-purpose languages (TypeScript, Go, Python). Both favor immutable infrastructure: replace servers rather than mutating them in place.

**GitOps** makes Git the single source of truth for *both* app and infra state. In the **pull-based** model (Argo CD, Flux), an in-cluster agent continuously compares the live cluster to the desired state in Git and reconciles drift — credentials stay inside the cluster, and any manual change is auto-reverted. In **push-based** delivery, a CI runner holds credentials and pushes changes outward. Pull-based GitOps is generally safer (no external cluster credentials, automatic drift correction, Git history as audit log).

## Environments, Promotion, and Config

A change flows **dev → staging → prod** (sometimes via QA/UAT/pre-prod). The principle: promote the *same immutable artifact* across environments, changing only configuration. Environment parity (staging closely mirrors prod) prevents "works in staging" surprises.

Config/secrets management follows **12-factor**: strict separation of config from code, with config supplied by the environment (env vars / injected files), never committed. Secrets live in a dedicated manager (Vault, AWS Secrets Manager, SealedSecrets), are rotated, scoped least-privilege, and never logged. This lets one build run anywhere by swapping only its environment.

## Observability and SRE Basics

You cannot operate what you cannot see. Observability rests on **three pillars**:

- **Logs** — discrete, timestamped events (structured/JSON for queryability).
- **Metrics** — numeric time series, cheap to store, ideal for dashboards and alerting.
- **Traces** — the path of a single request across services, exposing where latency accrues in distributed systems.

The **four golden signals** to watch on every service: **latency** (how long requests take, p50/p95/p99), **traffic** (demand/throughput), **errors** (failed-request rate), and **saturation** (how full the system is — CPU, memory, queue depth). For infra-centric views, the USE method (Utilization, Saturation, Errors) complements this.

Reliability targets:

| Term | Meaning |
| --- | --- |
| **SLI** | A measured indicator of service health (e.g., % of requests under 300ms). |
| **SLO** | The internal target for an SLI (e.g., 99.9% of requests succeed monthly). |
| **SLA** | A contractual promise to customers with consequences if breached (looser than the SLO). |
| **Error budget** | `1 − SLO`. The allowed amount of unreliability; spending it pauses risky launches, having surplus permits faster shipping. |

**Alert on symptoms, not causes.** Page humans on user-visible pain (SLO burn: rising error rate, latency) rather than on every internal cause (one node at 90% CPU). Cause-based alerts are noisy and breed alert fatigue; symptom-based alerts map to "is the customer hurting?" Use multi-window burn-rate alerts so fast budget burn pages immediately and slow burn opens a ticket.

## DORA Metrics

The four DORA metrics correlate software-delivery performance with organizational outcomes:

| Metric | Measures | Healthy direction |
| --- | --- | --- |
| **Deployment frequency** | How often you ship to prod | Higher (elite: on-demand/multiple per day) |
| **Lead time for changes** | Commit → running in prod | Lower (elite: < 1 day) |
| **Change failure rate** | % of deploys causing a failure/rollback | Lower (elite: 0-15%) |
| **MTTR / failed-deploy recovery** | Time to restore service after a failure | Lower (elite: < 1 hour) |

The first two measure **throughput**; the last two measure **stability**. The key insight: they move *together* — small frequent batches with strong automation are both faster *and* safer. Throughput without stability (move fast, break things) and stability without throughput (slow and "safe") are both anti-patterns.

## Supply-Chain and Build Security

The build pipeline is itself an attack surface. Core defenses:

- **SBOM (Software Bill of Materials)** — a machine-readable inventory of every component and dependency in an artifact, enabling fast "are we affected?" answers when a CVE drops.
- **Dependency / vulnerability scanning** — automatically flag known-vulnerable packages and base images in CI; fail the build on critical findings.
- **Signed artifacts & provenance** — cryptographically sign images/artifacts (e.g., Sigstore/cosign) and record provenance attestations so deploy targets can verify *what* was built, *from which source*, and *by which pipeline* (SLSA framework). Verify signatures at admission time.
- **Hardened pipelines** — least-privilege CI credentials, pinned action/tool versions, isolated ephemeral build runners, and protected branches so a compromised dependency can't tamper with the build.

## How a Big Company Does CI/CD (Synthesized)

A large, high-velocity engineering org (think Uber/Netflix scale) typically converges on a pattern like this:

1. **Repo strategy** — Either a large **monorepo** with a build graph that tests only affected targets, or many service repos ("polyrepo") with a shared, templated golden pipeline. Either way, the goal is consistency: every team inherits the same gates by default.
2. **Trunk-based development** — Engineers merge small changes to mainline many times a day behind feature flags, avoiding long-lived divergent branches and merge hell.
3. **Automated gates, no manual QA bottleneck** — PRs run a tiered suite: fast unit tests → integration → a curated set of end-to-end checks, plus lint, security scan, and SBOM generation. Flaky tests are quarantined so they don't erode trust in the pipeline.
4. **Build once, immutable artifact** — A signed container image tagged by commit SHA is promoted unchanged from staging to prod; config and secrets are injected per environment.
5. **Progressive, automated rollout** — Deploys go out region-by-region / cell-by-cell as a canary. An **automated canary analysis** service compares the new version's golden signals against a control; if metrics regress, it halts and rolls back with no human in the loop.
6. **Feature flags everywhere** — Code ships dark and is released to cohorts via flags, so a bad feature is disabled instantly without a redeploy.
7. **Observability-driven operation** — Every service emits metrics, structured logs, and distributed traces; on-call is paged on SLO burn, guarded by error budgets that gate how aggressively teams can ship.
8. **Self-service platform** — A central platform/SRE team owns the paved road (pipeline templates, deploy tooling, observability, secrets) so product teams ship safely without reinventing infrastructure.

The throughline: small batches + immutable artifacts + automated verification + fast rollback. That combination is what lets such organizations deploy thousands of times a day while keeping change failure rates low.
