# Integration Patterns

How services communicate: APIs, events, and data sharing.

## Synchronous Patterns

### REST API

**Request-response over HTTP.**

```
┌─────────┐  GET /users/123  ┌─────────┐
│ Client  │─────────────────►│ Server  │
│         │◄─────────────────│         │
└─────────┘  {id: 123, ...}  └─────────┘
```

**When to use:**
- Simple CRUD operations
- Public APIs
- Request-response needed

**Best practices:**
- Use proper HTTP methods and status codes
- Version your API
- Implement pagination for lists
- Use HATEOAS for discoverability

### gRPC

**Binary protocol, strongly typed, bidirectional streaming.**

```
┌─────────┐  Protocol Buffers  ┌─────────┐
│ Client  │◄──────────────────►│ Server  │
└─────────┘                    └─────────┘
```

**When to use:**
- Service-to-service communication
- High performance required
- Streaming needed
- Polyglot environments

**Trade-offs vs REST:**

| REST | gRPC |
|------|------|
| Text (JSON) | Binary (protobuf) |
| HTTP/1.1 | HTTP/2 |
| Easy to debug | Harder to debug |
| Flexible schema | Strict schema |
| Widely supported | Limited browser support |

### GraphQL

**Query language for APIs, client specifies data shape.**

```graphql
query {
  user(id: 123) {
    name
    orders(last: 5) {
      total
      items { name }
    }
  }
}
```

**When to use:**
- Multiple clients with different data needs
- Avoiding over-fetching/under-fetching
- Rapid frontend iteration

## Asynchronous Patterns

### Message Queue

**Point-to-point message delivery.**

```
┌──────────┐     ┌─────────┐     ┌──────────┐
│ Producer │────►│  Queue  │────►│ Consumer │
└──────────┘     └─────────┘     └──────────┘
```

**Characteristics:**
- One consumer receives each message
- Messages consumed in order (usually)
- Consumer acknowledges receipt
- Failed messages can be retried/dead-lettered

**Use cases:**
- Work distribution
- Load leveling
- Decoupling services

### Publish/Subscribe

**One-to-many message distribution.**

```
                    ┌──────────┐
               ┌───►│Consumer A│
┌──────────┐   │    └──────────┘
│ Publisher│───┼───►┌──────────┐
└──────────┘   │    │Consumer B│
               │    └──────────┘
               └───►┌──────────┐
                    │Consumer C│
                    └──────────┘
```

**Characteristics:**
- All subscribers receive each message
- Subscribers can filter by topic
- Good for broadcasting events

**Use cases:**
- Event notification
- Data replication
- Audit logging

### Event Sourcing

**Store events, not current state.**

```
┌───────────────────────────────────────────────────┐
│                  Event Store                       │
├───────────────────────────────────────────────────┤
│ 1. OrderCreated { orderId: 123, items: [...] }   │
│ 2. PaymentReceived { orderId: 123, amount: 100 } │
│ 3. OrderShipped { orderId: 123, tracking: "..." }│
└───────────────────────────────────────────────────┘
                        │
                        ▼ (rebuild)
                 Current State
                 { status: "shipped" }
```

**Benefits:**
- Complete audit trail
- Can rebuild state at any point
- Natural fit for event-driven

**Challenges:**
- Event schema evolution
- Rebuilding state can be slow
- More complex than CRUD

### CQRS (Command Query Responsibility Segregation)

**Separate read and write models.**

```
              ┌─────────────────────────────────────┐
              │                                     │
   Commands   │    ┌─────────────┐                 │
─────────────►├───►│Write Model  │                 │
              │    └──────┬──────┘                 │
              │           │ events                  │
              │           ▼                         │
              │    ┌─────────────┐                 │
              │    │ Read Model  │◄────────────────┤
              │    └─────────────┘      Queries    │
              │                                     │
              └─────────────────────────────────────┘
```

**When to use:**
- Read and write patterns differ significantly
- Need different scaling for reads vs writes
- Complex querying requirements

## Distributed Transaction Patterns

### Saga Pattern

**Sequence of local transactions with compensating actions.**

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│ Create  │────►│ Reserve │────►│ Charge  │
│ Order   │     │Inventory│     │ Payment │
└─────────┘     └─────────┘     └─────────┘
     │               │               │
     │               │               │ (if fails)
     ▼               ▼               ▼
┌─────────┐     ┌─────────┐     ┌─────────┐
│ Cancel  │◄────│ Release │◄────│  Void   │
│ Order   │     │Inventory│     │ Payment │
└─────────┘     └─────────┘     └─────────┘
```

**Choreography (event-based):**
```
Order Service ──► OrderCreated event
                       │
            ┌──────────┴──────────┐
            ▼                     ▼
    Inventory Service      Payment Service
         │                       │
    InventoryReserved       PaymentCharged
         │                       │
         └──────────┬────────────┘
                    ▼
              OrderConfirmed
```

**Orchestration (coordinator-based):**
```
┌──────────────────────────────────────────┐
│             Saga Orchestrator            │
├──────────────────────────────────────────┤
│ 1. Call Order Service: Create            │
│ 2. Call Inventory Service: Reserve       │
│ 3. Call Payment Service: Charge          │
│ 4. Call Order Service: Confirm           │
│                                          │
│ On failure: Execute compensating actions │
└──────────────────────────────────────────┘
```

| Choreography | Orchestration |
|--------------|---------------|
| Decentralized | Centralized logic |
| Loose coupling | Easier to understand |
| Hard to track | Single point to monitor |
| No single point of failure | Orchestrator can fail |

## API Gateway Pattern

**Single entry point for all clients.**

```
┌─────────┐         ┌─────────────┐
│ Mobile  │────────►│             │────►┌─────────┐
└─────────┘         │             │     │Service A│
                    │ API Gateway │     └─────────┘
┌─────────┐         │             │
│   Web   │────────►│ - Auth      │────►┌─────────┐
└─────────┘         │ - Rate limit│     │Service B│
                    │ - Routing   │     └─────────┘
┌─────────┐         │ - Transform │
│ Partner │────────►│             │────►┌─────────┐
└─────────┘         └─────────────┘     │Service C│
                                        └─────────┘
```

**Responsibilities:**
- Authentication/Authorization
- Rate limiting
- Request routing
- Response transformation
- Caching
- Monitoring

## Service Mesh

**Infrastructure layer for service-to-service communication.**

```
┌─────────────────────────────────────────────────┐
│                  Service Mesh                    │
│  ┌─────────┐ ┌───────┐       ┌─────────┐       │
│  │Service A│─│ Proxy │───────│ Proxy   │─┌─────┤
│  └─────────┘ └───────┘       └───────┬─┘ │     │
│                                      │   │     │
│  ┌─────────┐ ┌───────┐       ┌──────┴┐  │     │
│  │Service B│─│ Proxy │───────│Service│──┘     │
│  └─────────┘ └───────┘       │   C   │        │
│                              └───────┘        │
└─────────────────────────────────────────────────┘
```

**Features:**
- Automatic retries and circuit breaking
- mTLS between services
- Traffic management
- Observability (metrics, tracing)

## Integration Decision Tree

```
Need to communicate between services?
              │
              ├── Need immediate response?
              │         │
              │         Yes ──► REST/gRPC
              │         │
              │         No ──► Events/Queue
              │
              ├── Multiple consumers?
              │         │
              │         Yes ──► Pub/Sub
              │         │
              │         No ──► Point-to-point Queue
              │
              └── Distributed transaction?
                        │
                        ├── Can use eventual consistency? ──► Saga
                        │
                        └── Need strong consistency? ──► Consider monolith
```
