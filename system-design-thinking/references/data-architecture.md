# Data Architecture

Storage patterns, consistency models, and data management strategies.

## Storage Selection

### Decision Matrix

| Need | Consider | Examples |
|------|----------|----------|
| Structured data, ACID | Relational | PostgreSQL, MySQL |
| Flexible schema | Document | MongoDB, DynamoDB |
| Key-value, caching | Key-Value | Redis, Memcached |
| Relationships, graphs | Graph | Neo4j, Amazon Neptune |
| Time series | Time Series | InfluxDB, TimescaleDB |
| Full-text search | Search | Elasticsearch, Algolia |
| Analytics (OLAP) | Columnar | BigQuery, Snowflake, ClickHouse |
| File/blob storage | Object Store | S3, GCS, Azure Blob |
| Streaming | Log | Kafka, Kinesis |

### Relational vs Document

| Relational (PostgreSQL) | Document (MongoDB) |
|-------------------------|-------------------|
| Fixed schema | Flexible schema |
| ACID transactions | Limited transactions |
| Complex queries, JOINs | Simple queries |
| Normalized data | Denormalized data |
| Schema changes need migration | Schema evolves naturally |
| Mature, well-understood | Good for rapid iteration |

**Choose relational when:**
- Data is structured and relationships matter
- Complex queries required
- Strong consistency needed
- Team has SQL expertise

**Choose document when:**
- Schema is evolving rapidly
- Data is hierarchical/nested
- Horizontal scaling is priority
- Simple access patterns

## Consistency Models

### Strong Consistency

**All reads see the latest write.**

```
Client A writes X=1
         │
         ▼
    ┌─────────┐
    │   DB    │ X=1
    └─────────┘
         │
         ▼
Client B reads X → always returns 1
```

**Characteristics:**
- Simplest programming model
- Higher latency
- Lower availability during failures

**Use when:**
- Financial transactions
- Inventory management
- Booking systems

### Eventual Consistency

**All replicas converge to same value eventually.**

```
Client A writes X=1
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│Replica│ │Replica│
│  X=1  │ │ X=0   │ (not yet replicated)
└───────┘ └───────┘
    │         │
    ▼         ▼ (later)
┌───────┐ ┌───────┐
│  X=1  │ │  X=1  │ (eventually consistent)
└───────┘ └───────┘
```

**Characteristics:**
- Higher availability
- Lower latency
- Temporary inconsistency

**Use when:**
- Social media feeds
- Shopping carts
- Analytics
- Caching

### Consistency Levels (Cassandra example)

| Level | Meaning | Use Case |
|-------|---------|----------|
| ONE | Write to one replica | Logging, metrics |
| QUORUM | Majority of replicas | Balance of consistency/availability |
| ALL | All replicas | Highest consistency, lowest availability |
| LOCAL_QUORUM | Majority in local datacenter | Multi-region with local consistency |

## Data Patterns

### Single Source of Truth

**Each piece of data has exactly one authoritative source.**

```
┌─────────────┐
│User Service │ ◄── Source of truth for user data
└──────┬──────┘
       │ publishes user events
       ▼
┌──────────────┐     ┌──────────────┐
│Order Service │     │Email Service │
│ (cached copy)│     │ (cached copy)│
└──────────────┘     └──────────────┘
```

**Rules:**
- Only the owner can modify the data
- Others subscribe to updates or query the owner
- Cached copies are explicitly "cache" not "source"

### Database per Service

**Each service owns its data store.**

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│User Service │     │Order Service│     │Product Svc  │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
   ┌───┴───┐           ┌───┴───┐           ┌───┴───┐
   │User DB│           │Order  │           │Product│
   │       │           │  DB   │           │  DB   │
   └───────┘           └───────┘           └───────┘
```

**Benefits:**
- Services are independent
- Can choose best storage for each
- No schema coordination

**Challenges:**
- No cross-service joins
- Distributed transactions hard
- Data duplication

### Shared Database (Anti-pattern in microservices)

```
┌─────────────┐     ┌─────────────┐
│User Service │     │Order Service│
└──────┬──────┘     └──────┬──────┘
       │                   │
       └─────────┬─────────┘
             ┌───┴───┐
             │Shared │
             │  DB   │
             └───────┘
```

**Problems:**
- Services are coupled through schema
- Can't deploy independently
- Changes affect multiple services

**When it's OK:**
- Transitioning from monolith
- Small team, simple system
- Read-only shared data (reference data)

## Data Replication Patterns

### Primary-Replica

```
┌─────────────┐
│   Primary   │ ◄── All writes
└──────┬──────┘
       │ async replication
   ┌───┴───┐
   ▼       ▼
┌─────┐ ┌─────┐
│Repl1│ │Repl2│ ◄── Reads can go here
└─────┘ └─────┘
```

**Replication modes:**

| Mode | Description | Trade-off |
|------|-------------|-----------|
| Async | Primary doesn't wait | Fast but data can be lost |
| Semi-sync | Wait for one replica | Balance |
| Sync | Wait for all replicas | Slow but durable |

### Multi-Primary

```
┌─────────┐         ┌─────────┐
│Primary A│◄───────►│Primary B│
│(Region 1)│        │(Region 2)│
└─────────┘         └─────────┘
```

**Challenges:**
- Conflict resolution needed
- Eventually consistent

**Use when:**
- Multi-region deployment
- Need local writes everywhere

## CQRS + Event Sourcing

**Separate write (commands) from read (queries), store events.**

```
          Commands                      Queries
              │                            ▲
              ▼                            │
      ┌───────────────┐           ┌───────────────┐
      │ Command Model │           │  Read Model   │
      │   (Events)    │──────────►│ (Projections) │
      └───────────────┘   build   └───────────────┘
              │
              ▼
      ┌───────────────┐
      │  Event Store  │
      │ 1. Created    │
      │ 2. Updated    │
      │ 3. Shipped    │
      └───────────────┘
```

**Benefits:**
- Optimized read/write models
- Complete audit trail
- Can rebuild any view
- Scale reads/writes independently

**Complexity:**
- Two models to maintain
- Eventual consistency between them
- Event versioning

## Data Migration Strategies

### Expand and Contract

```
Phase 1: Add new column (nullable)
┌──────────────────────────────┐
│ users                        │
│ - id                         │
│ - name (old)                 │
│ - full_name (new, nullable)  │
└──────────────────────────────┘

Phase 2: Migrate data, dual-write
Phase 3: Switch reads to new column
Phase 4: Drop old column
```

### Parallel Run

```
┌─────────────────────────────────────────────┐
│              Application                     │
│                   │                          │
│         ┌────────┴────────┐                 │
│         ▼                 ▼                 │
│    ┌─────────┐       ┌─────────┐           │
│    │ Old DB  │       │ New DB  │  (shadow) │
│    │ (source)│       │ (verify)│           │
│    └─────────┘       └─────────┘           │
└─────────────────────────────────────────────┘
```

**Steps:**
1. Write to both, read from old
2. Compare results, fix discrepancies
3. Switch reads to new
4. Stop writing to old

## Data Architecture Checklist

### For Each Service

- [ ] What data does it own?
- [ ] What storage technology fits?
- [ ] What consistency level needed?
- [ ] How does it share data with others?
- [ ] What's the backup/recovery strategy?

### For the System

- [ ] Single source of truth for each entity?
- [ ] How do services get data they need?
- [ ] What's the replication strategy?
- [ ] How do we handle cross-service queries?
- [ ] What's the data retention policy?
