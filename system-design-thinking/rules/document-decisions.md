---
title: Document Decisions
impact: MEDIUM
impactDescription: preserves context for future maintainers
tags: documentation, adr, communication
---

## Document Decisions

Record significant architectural decisions with context, options considered, and rationale. Future you (and your team) will thank you.

**Context:** Making architectural decisions that are hard to reverse or have significant impact.

**Heuristic:** For every significant decision, write an Architecture Decision Record (ADR):

```markdown
# ADR-001: [Decision Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
What is the issue we're facing? What forces are at play?

## Options Considered
1. Option A - [brief description]
2. Option B - [brief description]
3. Option C - [brief description]

## Decision
We will use [Option X] because [reasons].

## Consequences
- [Positive consequence]
- [Negative consequence / trade-off]
- [Risks and mitigations]
```

**What to document:**

| Document | Don't Document |
|----------|----------------|
| Choice of database | Which ORM methods to use |
| Service boundaries | Internal class structure |
| Communication patterns (sync/async) | HTTP client library |
| Authentication approach | Password hashing rounds |
| Deployment strategy | CI pipeline details |

**Apply when:**
- Decision is hard to reverse
- Decision affects multiple teams or systems
- Decision has significant cost implications
- You're choosing between viable alternatives

**Skip when:**
- Decision is easily reversible
- Decision is purely implementation detail
- Following established patterns/standards

**Why:** Architectural decisions outlive their creators. Without documented rationale, future teams either repeat the analysis or blindly change things, breaking assumptions.

**Example:**

```markdown
# ADR-003: Use PostgreSQL for Order Data

## Status
Accepted

## Context
We need a database for order data. Expected volume is 100K orders/day
with complex queries for reporting. Team has strong SQL experience.

## Options Considered
1. PostgreSQL - Relational, strong consistency, excellent query capabilities
2. MongoDB - Flexible schema, horizontal scaling
3. DynamoDB - Managed, auto-scaling, but limited query patterns

## Decision
PostgreSQL because:
- Complex reporting queries are critical
- Team expertise reduces risk
- Strong consistency matches business requirements
- Vertical scaling sufficient for projected 3-year growth

## Consequences
- ✅ Rich query capabilities for reporting
- ✅ ACID transactions for order integrity
- ⚠️ Will need to revisit if we exceed 1M orders/day
- ⚠️ Schema migrations require careful planning
```
