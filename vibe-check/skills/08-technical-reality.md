# 08 — Technical Reality

**Goal:** Ground the vision in engineering reality and identify irreversible technical decisions before they are made accidentally.

**When to use independently:** Before any architecture decisions are finalized, or when a plan seems technically solid but has not been pressure-tested.

---

## Question Matrix

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 1 | What are the non-negotiable technical constraints? Name every existing system this must integrate with, every compliance requirement, every latency boundary. | Hard technical boundaries that cannot be designed around | Stated |
| 2 | What has already been tried or ruled out technically? Walk me through what failed and exactly why it failed. | Engineering scar tissue — the hardest-won knowledge on the team | Observed |
| 3 | What are the real performance requirements? Give me p50, p95, and p99 latency targets. Give me throughput numbers. Give me availability SLA. Not aspirations — requirements with a consequence if missed. | Over- or under-engineering risk, and whether the architecture actually needs to be complex | Stated |
| 4 | What is the data story? Where does data originate, how does it flow through the system, who owns it, what is the schema, how large does it get, and who can access it? | Data architecture assumptions, privacy exposure, and schema rigidity | Stated |
| 5 | What is the deployment reality? Cloud or on-prem, which provider, what is the CI/CD maturity, what is the team's actual infrastructure skill level, and what is the monthly infrastructure budget? | Infrastructure constraints and the gap between ideal architecture and buildable architecture | Stated |
| 6 | Which technical decisions are irreversible or extremely expensive to change after six months? Name them specifically. | Decisions that need to be right because there is no cheap course correction | Stated |
| 7 | What third-party dependencies are you taking on? What happens when each one goes down? When they change pricing? When they sunset? | Vendor risk and single points of failure hidden inside dependency choices | Stated |
| 8 | What does the team not know how to build right now? Be specific — which parts require skills you do not currently have? | Capability gap and the build-vs-hire-vs-buy decision | Stated |
| 9 | What is the most technically risky component of this system? What is your plan if it does not work as expected? | Risk concentration and the absence or presence of a fallback | Technical |
| 10 | If traffic or data volume is 10x what you planned on day one, what breaks first? If it is 0.1x, what do you over-built? | Scaling assumption and over-engineering detection | Technical |
| 11 | What security assumptions are baked into the architecture? Where could a motivated attacker cause the most damage? | Security posture and threat model coverage | Technical |
| 12 | What does "good enough" look like technically for a first version? What are you building to be perfect that does not need to be? | Perfectionism tax and the cost of premature optimization | Stated |

---

## Technical Decision Log

Fill during this phase. Any decision marked Permanent + Low Confidence is a project red flag:

| Decision | Options Considered | Choice Made | Rationale | Reversibility | Confidence |
|----------|-------------------|-------------|-----------|---------------|------------|
| _fill_ | | | | Easy / Hard / Permanent | High / Med / Low |

---

## Follow-up Probes (use when confidence < 70%)

- You said [constraint X] is non-negotiable. What specifically happens if you violate it — who is affected and what is the consequence?
- The p95 latency target is [X]ms. Have you benchmarked the current approach against that target? What did you see?
- [Third-party dependency] — what is their uptime SLA, and what does your system do during an outage?
- You marked [decision] as reversible. Walk me through what a reversal actually costs in time and money at month six.
- The skill gap you described — when in the build timeline does that gap become blocking?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| Aspirational SLA | Performance requirements stated as "we want it to be fast" without numbers |
| Dependency blindness | Architecture diagram shows third-party APIs with no failure mode discussion |
| Reversibility inflation | Marks database schema as "easy to change" when it would require migration of production data |
| Skill gap avoidance | Plan requires unfamiliar technology but no upskilling or hiring plan exists |
| Compliance assumption | Handles PII or payments but compliance requirements are "TBD" |

---

## Output Artifact: Technical Decision Log

```
TECHNICAL DECISION LOG
─────────────────────────────────────────────────────────
Project:   ___
Date:      ___
Architect: ___

HARD CONSTRAINTS
  Integration requirements:  ___
  Compliance requirements:   ___
  Latency boundaries:        ___
  Data sovereignty:          ___

PERFORMANCE REQUIREMENTS
  Latency:      p50 ___ms  p95 ___ms  p99 ___ms
  Throughput:   ___ req/sec
  Availability: ___% SLA
  Data volume:  ___ at launch  ___ at 12 months

DATA STORY
  Origin:       ___
  Flow:         ___
  Owner:        ___
  Schema:       ___
  Size:         ___
  Access:       ___

DEPLOYMENT REALITY
  Cloud / On-prem:    ___
  Provider:           ___
  CI/CD maturity:     Low / Medium / High
  Infra budget/month: $___

DECISION LOG
Decision           | Options      | Choice | Rationale | Reversibility | Confidence
-------------------|--------------|--------|-----------|---------------|------------
                   |              |        |           | Permanent     | Low ← FLAG

THIRD-PARTY DEPENDENCIES
Dependency | Purpose | SLA | Failure Mode | Pricing Risk | Sunset Risk
-----------|---------|-----|--------------|--------------|------------
           |         |     |              |              |

SKILL GAPS
Gap: ___  Blocking at: Month ___  Resolution: Build / Hire / Buy

HIGHEST RISK COMPONENT
  Component:   ___
  Risk:        ___
  Fallback:    ___

PHASE CONFIDENCE:  ___% (must reach 80% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```
