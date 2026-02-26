# Trade-off Analysis

Frameworks for evaluating architectural options and making decisions.

## Core Principle

There are no perfect solutions, only trade-offs. Make trade-offs explicit.

## Decision Matrix

For any significant decision, create a matrix:

| Criteria | Weight | Option A | Option B | Option C |
|----------|--------|----------|----------|----------|
| Performance | 3 | 4 (12) | 3 (9) | 5 (15) |
| Simplicity | 2 | 5 (10) | 3 (6) | 2 (4) |
| Cost | 2 | 3 (6) | 4 (8) | 2 (4) |
| Team expertise | 1 | 4 (4) | 5 (5) | 2 (2) |
| **Total** | | **32** | **28** | **25** |

Steps:
1. List evaluation criteria
2. Weight criteria by importance (1-3)
3. Score each option per criteria (1-5)
4. Multiply and sum

**Warning:** Don't let numbers override judgment. The matrix clarifies thinking, it doesn't make the decision.

## CAP Theorem

In distributed systems, choose two of three:

```
        Consistency
           /\
          /  \
         /    \
        /  CA  \
       /________\
      /\        /\
     /  \  CP  /  \
    / AP \    /    \
   /______\  /______\
Availability  Partition Tolerance
```

| Choice | Guarantees | Sacrifices | Use Cases |
|--------|------------|------------|-----------|
| **CA** | Consistency + Availability | Partition tolerance | Single-node databases |
| **CP** | Consistency + Partition tolerance | Availability during partition | Banking, inventory |
| **AP** | Availability + Partition tolerance | Consistency during partition | Social media, caching |

**Reality:** Network partitions happen, so you're choosing between CP and AP.

## PACELC Theorem

Extends CAP: "If Partition, choose A or C; Else, choose Latency or Consistency"

| System | During Partition | Normal Operation |
|--------|------------------|------------------|
| Traditional RDBMS | PC (consistency) | EC (consistency) |
| DynamoDB | PA (availability) | EL (latency) |
| Cassandra | PA (availability) | EL (latency) |
| MongoDB | PC (consistency) | EC (consistency) |

## Common Trade-offs

### Consistency vs Availability

| Strong Consistency | Eventual Consistency |
|--------------------|----------------------|
| All reads see latest write | Reads may see stale data |
| Higher latency | Lower latency |
| Lower availability | Higher availability |
| Simpler programming model | Requires conflict handling |

**Choose strong when:** Financial transactions, inventory, bookings
**Choose eventual when:** Social feeds, analytics, caching

### Latency vs Throughput

| Optimize for Latency | Optimize for Throughput |
|----------------------|------------------------|
| Respond fast to each request | Process many requests total |
| More resources per request | Batch processing |
| Smaller batches | Larger batches |

**Choose latency when:** User-facing APIs, real-time systems
**Choose throughput when:** Batch jobs, data pipelines, analytics

### Simplicity vs Flexibility

| Simple | Flexible |
|--------|----------|
| Fewer abstractions | More extension points |
| Faster to build | Slower to build |
| Easier to understand | Harder to understand |
| Less adaptable | More adaptable |

**Choose simple when:** MVP, small team, unclear requirements
**Choose flexible when:** Platform, multiple use cases, stable domain

### Build vs Buy

| Build | Buy/Use Managed |
|-------|-----------------|
| Full control | Vendor dependency |
| Customizable | Limited customization |
| Engineering cost | Licensing cost |
| You maintain it | They maintain it |

**Choose build when:** Core differentiator, unique requirements
**Choose buy when:** Commodity functionality, faster time to market

### Monolith vs Microservices

| Monolith | Microservices |
|----------|---------------|
| Simple deployment | Complex deployment |
| Easy debugging | Distributed tracing |
| Fast development (small team) | Fast development (large team) |
| Shared database | Database per service |
| Single codebase | Multiple codebases |

**Choose monolith when:** Small team, new product, unclear domains
**Choose microservices when:** Large team, proven domains, independent scaling

## Quality Attributes

Use this table to identify which attributes matter most:

| Attribute | Definition | Trade-offs Against |
|-----------|------------|-------------------|
| Performance | Response time, throughput | Cost, simplicity |
| Scalability | Handle growth | Complexity, cost |
| Availability | Uptime percentage | Cost, consistency |
| Reliability | Correct behavior | Performance |
| Security | Protection from threats | Usability, performance |
| Maintainability | Ease of change | Initial development time |
| Testability | Ease of testing | Simplicity |
| Portability | Run on different platforms | Performance optimization |

## Decision Template

For significant trade-offs, document:

```markdown
## Trade-off: [Name]

### Context
What decision are we making? What constraints exist?

### Options
1. **Option A**: [Description]
   - Pros: ...
   - Cons: ...

2. **Option B**: [Description]
   - Pros: ...
   - Cons: ...

### Analysis
[Decision matrix or qualitative analysis]

### Decision
We choose [Option X] because [primary reasons].

### Accepted Trade-offs
- We accept [downside] because [justification]
- We mitigate [risk] by [mitigation]

### Review Trigger
Revisit this decision if:
- [Condition that would change the decision]
```
