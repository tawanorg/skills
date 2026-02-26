# Scalability Patterns

Strategies for handling growth in users, data, and traffic.

## Scaling Strategies

### Vertical Scaling (Scale Up)

**Add more resources to existing machines.**

```
Before:        After:
┌────────┐     ┌────────────┐
│ 4 CPU  │     │  16 CPU    │
│ 8 GB   │ ──► │  64 GB     │
│ Server │     │  Server    │
└────────┘     └────────────┘
```

**Pros:**
- Simple, no code changes
- No distributed system complexity
- Works for stateful services

**Cons:**
- Hardware limits
- Single point of failure
- Expensive at high end
- Downtime during upgrade

**When to use:**
- Quick fix for immediate capacity
- Stateful services (databases)
- Before investing in horizontal scaling

### Horizontal Scaling (Scale Out)

**Add more machines.**

```
Before:            After:
┌────────┐         ┌────────┐
│ Server │         │ Server │
└────────┘         └────────┘
                   ┌────────┐
              ──►  │ Server │
                   └────────┘
                   ┌────────┐
                   │ Server │
                   └────────┘
```

**Pros:**
- Theoretically unlimited
- Fault tolerance (no single point of failure)
- Can scale incrementally

**Cons:**
- Requires stateless design
- Load balancing needed
- Distributed system complexity

**When to use:**
- Stateless services
- Need fault tolerance
- Cost-effective scaling

## Load Balancing

### Strategies

| Strategy | How It Works | Best For |
|----------|--------------|----------|
| Round Robin | Rotate through servers | Equal servers, stateless |
| Least Connections | Send to server with fewest active | Variable request duration |
| Weighted | Assign weights to servers | Mixed server capacity |
| IP Hash | Same IP to same server | Session affinity |
| Least Response Time | Fastest responding server | Mixed latency backends |

### Architecture

```
                    ┌─────────────┐
     Requests ─────►│Load Balancer│
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      ┌─────────┐     ┌─────────┐     ┌─────────┐
      │ Server  │     │ Server  │     │ Server  │
      └─────────┘     └─────────┘     └─────────┘
```

## Caching

### Cache Layers

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│ Browser │────►│   CDN   │────►│  App    │────►│   DB    │
│  Cache  │     │  Cache  │     │  Cache  │     │         │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
   L1 (ms)       L2 (10ms)      L3 (1-10ms)     L4 (10-100ms)
```

### Caching Strategies

| Strategy | How It Works | Use When |
|----------|--------------|----------|
| **Cache-Aside** | App checks cache, then DB | Read-heavy, cache misses OK |
| **Read-Through** | Cache fetches from DB on miss | Simple read caching |
| **Write-Through** | Write to cache and DB | Consistency important |
| **Write-Behind** | Write to cache, async to DB | Write-heavy, can lose data |
| **Refresh-Ahead** | Proactively refresh before expiry | Predictable access patterns |

### Cache-Aside Pattern

```typescript
async function getUser(id: string): Promise<User> {
  // Check cache
  const cached = await cache.get(`user:${id}`);
  if (cached) return cached;

  // Cache miss - fetch from DB
  const user = await db.users.findById(id);

  // Store in cache
  await cache.set(`user:${id}`, user, { ttl: 3600 });

  return user;
}
```

### Cache Invalidation

> "There are only two hard things in Computer Science: cache invalidation and naming things."

| Strategy | Approach | Trade-off |
|----------|----------|-----------|
| TTL | Expire after time | Simple but potentially stale |
| Event-based | Invalidate on change | Fresh but complex |
| Version keys | Include version in key | Fresh, simple, but more misses |

## Database Scaling

### Read Replicas

```
┌─────────┐     ┌─────────┐
│ Primary │────►│ Replica │ (reads)
│  (R/W)  │     └─────────┘
└─────────┘     ┌─────────┐
                │ Replica │ (reads)
                └─────────┘
```

**Use when:** Read-heavy workloads (10:1 read:write ratio)

### Sharding (Horizontal Partitioning)

```
┌─────────────────────────────────────────┐
│              Shard Router               │
└───────────────────┬─────────────────────┘
                    │
    ┌───────────────┼───────────────┐
    ▼               ▼               ▼
┌─────────┐   ┌─────────┐   ┌─────────┐
│ Shard 1 │   │ Shard 2 │   │ Shard 3 │
│ A-H     │   │ I-P     │   │ Q-Z     │
└─────────┘   └─────────┘   └─────────┘
```

**Sharding strategies:**

| Strategy | How | Pros | Cons |
|----------|-----|------|------|
| Range | By value range (A-H, I-P) | Simple, range queries | Hotspots possible |
| Hash | Hash of key | Even distribution | No range queries |
| Geographic | By region | Low latency | Cross-region queries hard |
| Directory | Lookup table | Flexible | Lookup bottleneck |

### Database Scaling Decision Tree

```
Need more capacity?
        │
        ├── Read-heavy? ──► Read replicas
        │
        ├── Write-heavy? ──► Sharding
        │
        └── Both? ──► Sharding + replicas per shard
```

## Asynchronous Processing

### Message Queues

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│Producer │────►│  Queue  │────►│Consumer │
└─────────┘     └─────────┘     └─────────┘
```

**Benefits:**
- Decouples producers from consumers
- Handles traffic spikes (buffer)
- Enables retry on failure
- Allows independent scaling

### When to Use Queues

| Sync | Async (Queue) |
|------|---------------|
| User needs immediate response | Background processing OK |
| Simple request-response | Multiple consumers |
| Low volume | High volume, spiky |
| Must succeed or fail fast | Can retry later |

## CDN (Content Delivery Network)

```
                    ┌─────────────────────┐
                    │      Origin         │
                    └──────────┬──────────┘
                               │
       ┌───────────────────────┼───────────────────────┐
       │                       │                       │
┌──────┴──────┐         ┌──────┴──────┐         ┌──────┴──────┐
│  CDN Edge   │         │  CDN Edge   │         │  CDN Edge   │
│  (US-East)  │         │  (EU-West)  │         │  (Asia)     │
└─────────────┘         └─────────────┘         └─────────────┘
       │                       │                       │
    [Users]                 [Users]                 [Users]
```

**What to put on CDN:**
- Static assets (images, CSS, JS)
- API responses (with caching headers)
- Video/media streaming

## Scaling Checklist

### Before Scaling

- [ ] Profile to find actual bottleneck
- [ ] Optimize the bottleneck first
- [ ] Add caching where appropriate
- [ ] Then scale the remaining bottleneck

### Scaling Services

- [ ] Make services stateless
- [ ] Use load balancer
- [ ] Add health checks
- [ ] Implement graceful shutdown

### Scaling Databases

- [ ] Optimize queries and indexes first
- [ ] Add read replicas for read scaling
- [ ] Consider caching layer
- [ ] Shard only when necessary (complexity cost)

### Capacity Planning

```
Current: 1000 req/sec
Growth: 10x in 12 months
Target: 10,000 req/sec

Headroom: 2x for spikes
Design for: 20,000 req/sec
```
