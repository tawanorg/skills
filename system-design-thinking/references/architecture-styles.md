# Architecture Styles

Comparison of major architectural patterns and when to use each.

## Overview

| Style | Best For | Team Size | Complexity |
|-------|----------|-----------|------------|
| Monolith | Starting out, small teams | 1-8 | Low |
| Modular Monolith | Growing product | 5-15 | Medium |
| Microservices | Large organizations | 10+ per service | High |
| Serverless | Event-driven, variable load | Any | Medium |
| Event-Driven | Loose coupling, async flows | Any | Medium-High |

## Monolith

**Single deployable unit containing all functionality.**

```
┌─────────────────────────────────────┐
│            Monolith                 │
│  ┌─────────┐ ┌─────────┐ ┌───────┐ │
│  │  Users  │ │ Orders  │ │Payments│ │
│  └─────────┘ └─────────┘ └───────┘ │
│         ┌─────────────┐            │
│         │  Database   │            │
│         └─────────────┘            │
└─────────────────────────────────────┘
```

**Pros:**
- Simple to develop, test, deploy
- Easy debugging and profiling
- No network latency between components
- Single database, simple transactions
- Low operational overhead

**Cons:**
- Scaling requires scaling everything
- Large codebase can become unwieldy
- Long build/deploy times as it grows
- Team coupling (everyone in same codebase)
- Technology lock-in

**When to use:**
- New products with unclear domains
- Small teams (< 8 engineers)
- Simple scaling requirements
- Tight deadlines

## Modular Monolith

**Monolith with enforced module boundaries.**

```
┌─────────────────────────────────────┐
│         Modular Monolith            │
│  ┌─────────┐ ┌─────────┐ ┌───────┐ │
│  │  Users  │ │ Orders  │ │Payments│ │
│  │ Module  │ │ Module  │ │ Module │ │
│  └────┬────┘ └────┬────┘ └───┬───┘ │
│       │           │          │      │
│       └─────┬─────┴─────┬────┘      │
│             │ API Only  │           │
│         ┌───┴───────────┴───┐       │
│         │     Database      │       │
│         └───────────────────┘       │
└─────────────────────────────────────┘
```

**Key principle:** Modules communicate only through defined interfaces, not by reaching into each other's internals.

**Pros:**
- Monolith simplicity with better organization
- Easier to extract services later
- Clear ownership boundaries
- Prevents spaghetti code

**Cons:**
- Requires discipline to maintain boundaries
- Still single deployment unit
- Database still shared (but can be partitioned)

**When to use:**
- Growing monolith that needs structure
- Preparing for potential service extraction
- Medium-sized teams (5-15)

## Microservices

**Multiple small, independently deployable services.**

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Users  │     │ Orders  │     │Payments │
│ Service │     │ Service │     │ Service │
└────┬────┘     └────┬────┘     └────┬────┘
     │               │               │
┌────┴────┐     ┌────┴────┐     ┌────┴────┐
│   DB    │     │   DB    │     │   DB    │
└─────────┘     └─────────┘     └─────────┘
```

**Pros:**
- Independent deployment and scaling
- Technology flexibility per service
- Team autonomy
- Fault isolation
- Smaller, focused codebases

**Cons:**
- Distributed system complexity
- Network latency and failures
- Data consistency challenges
- Operational overhead (many deployments)
- Testing complexity

**When to use:**
- Large organizations (100+ engineers)
- Need independent scaling of components
- Different services need different tech
- Mature, stable domain boundaries

**Prerequisites:**
- Strong DevOps/platform capabilities
- Good monitoring and observability
- Clear domain boundaries
- Teams that can own services end-to-end

## Serverless

**Functions triggered by events, managed infrastructure.**

```
┌─────────┐     ┌─────────────┐
│  Event  │────►│  Function   │
│ Source  │     │   (Lambda)  │
└─────────┘     └──────┬──────┘
                       │
                       ▼
                ┌─────────────┐
                │  Database   │
                │ (DynamoDB)  │
                └─────────────┘
```

**Pros:**
- No server management
- Automatic scaling (including to zero)
- Pay per execution
- Quick to deploy individual functions

**Cons:**
- Cold start latency
- Execution time limits
- Vendor lock-in
- Complex local development
- Harder to test end-to-end
- State management challenges

**When to use:**
- Event-driven workloads
- Variable/spiky traffic
- Simple, stateless operations
- Cost optimization for low-traffic services

## Event-Driven Architecture

**Components communicate through events.**

```
┌─────────┐     ┌───────────────┐     ┌─────────┐
│ Service │────►│  Event Bus    │────►│ Service │
│    A    │     │ (Kafka/SQS)   │     │    B    │
└─────────┘     └───────────────┘     └─────────┘
                       │
                       ▼
                ┌─────────┐
                │ Service │
                │    C    │
                └─────────┘
```

**Pros:**
- Loose coupling between services
- Easy to add new consumers
- Natural audit trail
- Resilient to failures
- Good for async workflows

**Cons:**
- Eventual consistency
- Harder to debug (no request/response)
- Event schema evolution
- Ordering challenges
- Duplicate handling

**When to use:**
- Multiple consumers of same events
- Async workflows
- Need for audit trail
- Loose coupling is priority

## Hybrid Approaches

Most real systems combine patterns:

```
┌───────────────────────────────────────────────────┐
│                   API Gateway                      │
└───────────────────────┬───────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
   ┌─────────┐    ┌─────────┐    ┌───────────┐
   │  Core   │    │ Orders  │    │ Analytics │
   │Monolith │    │ Service │    │ (Lambda)  │
   └────┬────┘    └────┬────┘    └───────────┘
        │              │               ▲
        │              │               │
        └──────────────┴───────────────┘
                       │
                ┌──────┴──────┐
                │ Event Bus   │
                └─────────────┘
```

**Common combinations:**
- Monolith + extracted microservices for scale
- Microservices + serverless for events
- Event-driven core + sync APIs for queries

## Migration Path

```
Monolith ──► Modular Monolith ──► Microservices
                                       │
                                       ├──► + Serverless (for events)
                                       └──► + Event-Driven (for integration)
```

**Key:** Extract services only when you have:
- Proven need (scaling, team independence)
- Clear boundaries
- Operational maturity
