# Architecture Decision Record (ADR) Template

A template for documenting significant architectural decisions.

## What is an ADR?

An Architecture Decision Record captures a single architectural decision along with its context and consequences. It's a point-in-time snapshot of why a decision was made.

## When to Write an ADR

Write an ADR when:
- The decision is hard to reverse
- The decision affects multiple teams or systems
- The decision has significant cost or risk
- You're choosing between viable alternatives
- Future maintainers will ask "why did we do this?"

Don't write an ADR for:
- Routine implementation choices
- Decisions that follow established patterns
- Easily reversible changes

## ADR Format

```markdown
# ADR-[NUMBER]: [TITLE]

## Status

[Proposed | Accepted | Deprecated | Superseded by ADR-XXX]

## Date

[YYYY-MM-DD when the decision was made]

## Context

[Describe the issue that motivates this decision. What is the problem?
What forces are at play (technical, business, social, project)?
This section is neutral - it doesn't advocate for any particular solution.]

## Decision

[State the decision clearly. Use "We will..." language.
This is the architecture decision that was chosen.]

## Options Considered

### Option 1: [Name]
[Brief description]
- **Pros:** [List advantages]
- **Cons:** [List disadvantages]

### Option 2: [Name]
[Brief description]
- **Pros:** [List advantages]
- **Cons:** [List disadvantages]

### Option 3: [Name]
[Brief description]
- **Pros:** [List advantages]
- **Cons:** [List disadvantages]

## Consequences

### Positive
- [Benefit 1]
- [Benefit 2]

### Negative
- [Trade-off 1]
- [Trade-off 2]

### Risks
- [Risk 1]: [Mitigation strategy]
- [Risk 2]: [Mitigation strategy]

## Follow-up Actions

- [ ] [Action item 1]
- [ ] [Action item 2]

## References

- [Link to relevant documentation]
- [Link to related ADRs]
```

## Example ADR

```markdown
# ADR-003: Use PostgreSQL for Order Data

## Status

Accepted

## Date

2024-01-15

## Context

We need to select a database for storing order data in our e-commerce platform.

Key requirements:
- Expected volume: 100K orders/day, growing to 1M/day in 3 years
- Complex reporting queries (orders by customer, date ranges, product categories)
- Strong consistency for inventory and payment reconciliation
- Team has strong SQL experience, limited NoSQL experience
- Budget constraints favor managed services

Current state:
- Using MySQL 5.7 for user data
- No existing order storage
- AWS infrastructure

## Decision

We will use Amazon RDS PostgreSQL for order data storage.

## Options Considered

### Option 1: Amazon RDS PostgreSQL
Managed relational database with strong SQL capabilities.
- **Pros:**
  - Excellent query capabilities (window functions, CTEs)
  - Strong consistency with ACID transactions
  - Team expertise in SQL
  - Managed service reduces ops burden
  - JSONB for flexible attributes
- **Cons:**
  - Vertical scaling has limits
  - More expensive than DynamoDB at high scale
  - Schema migrations require planning

### Option 2: Amazon DynamoDB
Managed NoSQL with automatic scaling.
- **Pros:**
  - Automatic scaling
  - Pay-per-request pricing option
  - Low latency at any scale
- **Cons:**
  - Limited query patterns (requires careful key design)
  - Complex reporting needs separate solution
  - Team unfamiliar with NoSQL patterns
  - Eventual consistency by default

### Option 3: MongoDB Atlas
Managed document database.
- **Pros:**
  - Flexible schema
  - Good query capabilities
  - Horizontal scaling built-in
- **Cons:**
  - Another managed service to learn
  - Transaction support is newer/limited
  - Team has no MongoDB experience

## Consequences

### Positive
- Rich query capabilities for reporting without additional infrastructure
- ACID transactions ensure order integrity
- Team can be productive immediately
- Can use existing SQL tooling and knowledge

### Negative
- Will need to revisit if we exceed 1M orders/day (sharding or migration)
- Schema migrations require careful coordination
- Higher baseline cost than DynamoDB for low usage

### Risks
- **Scale ceiling:** Current approach works to ~1M orders/day
  - *Mitigation:* Design with sharding in mind; monitor growth; plan migration path to CitusDB or sharding by tenant
- **Single point of failure:**
  - *Mitigation:* Multi-AZ deployment; automated backups; read replicas

## Follow-up Actions

- [ ] Set up RDS PostgreSQL in staging
- [ ] Define schema with migration strategy
- [ ] Configure backups and monitoring
- [ ] Document runbooks for common operations
- [ ] Set alerts for capacity planning (80% thresholds)

## References

- [AWS RDS PostgreSQL Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html)
- [PostgreSQL Scaling Guide](https://www.postgresql.org/docs/current/high-availability.html)
- ADR-001: Use AWS as Cloud Provider
- ADR-002: Microservices Architecture
```

## ADR File Organization

```
docs/
└── adr/
    ├── README.md           # Index of all ADRs
    ├── adr-001-use-aws.md
    ├── adr-002-microservices.md
    ├── adr-003-postgresql-orders.md
    └── template.md         # This template
```

## ADR Index (README.md)

```markdown
# Architecture Decision Records

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [001](adr-001-use-aws.md) | Use AWS as Cloud Provider | Accepted | 2024-01-01 |
| [002](adr-002-microservices.md) | Microservices Architecture | Accepted | 2024-01-10 |
| [003](adr-003-postgresql-orders.md) | Use PostgreSQL for Order Data | Accepted | 2024-01-15 |
```

## Tips for Good ADRs

1. **Be concise** - ADRs should be readable in 5 minutes
2. **Focus on why** - The code shows what, ADRs explain why
3. **Include context** - Future readers need to understand the situation
4. **List options** - Show what you considered, not just what you chose
5. **Accept trade-offs** - Every decision has downsides; acknowledge them
6. **Keep them immutable** - Don't edit old ADRs; supersede them with new ones
7. **Review regularly** - Some decisions become obsolete
