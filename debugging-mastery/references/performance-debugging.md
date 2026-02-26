# Performance Debugging

Systematic approach to finding and fixing performance bottlenecks.

## The Golden Rule

**Profile first. Optimize second.**

```
Wrong:
"This looks slow" → Optimize → No improvement → Wasted time

Right:
Measure → Identify bottleneck → Optimize → Measure again → Confirm improvement
```

## The Performance Debugging Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    PERFORMANCE DEBUGGING                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DEFINE                                                      │
│     └── What is "slow"? Define metrics.                         │
│                                                                 │
│  2. MEASURE                                                     │
│     └── Baseline current performance                            │
│                                                                 │
│  3. PROFILE                                                     │
│     └── Where is time being spent?                              │
│                                                                 │
│  4. IDENTIFY                                                    │
│     └── What's the biggest bottleneck?                          │
│                                                                 │
│  5. HYPOTHESIZE                                                 │
│     └── Why is this slow?                                       │
│                                                                 │
│  6. OPTIMIZE                                                    │
│     └── Fix the bottleneck                                      │
│                                                                 │
│  7. VERIFY                                                      │
│     └── Measure again. Did it improve?                          │
│                                                                 │
│  8. REPEAT                                                      │
│     └── Find next bottleneck                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Defining Performance Goals

```markdown
Bad goal: "Make it faster"
Good goal: "API response time p95 < 200ms"

## Metrics to Define

### Latency
- p50 (median): Typical experience
- p95: Most users' worst experience
- p99: Tail latency

### Throughput
- Requests per second
- Transactions per second

### Resource Usage
- CPU utilization target
- Memory usage limit
- Network bandwidth

### User Experience
- Time to first byte (TTFB)
- Time to interactive (TTI)
- Core Web Vitals
```

## Profiling Tools

### CPU Profiling

```javascript
// Node.js CPU profiling
const inspector = require('inspector');
const fs = require('fs');
const session = new inspector.Session();
session.connect();

session.post('Profiler.enable', () => {
  session.post('Profiler.start', () => {
    // Run your code here
    doSlowOperation();

    session.post('Profiler.stop', (err, { profile }) => {
      fs.writeFileSync('profile.cpuprofile', JSON.stringify(profile));
    });
  });
});

// Open profile.cpuprofile in Chrome DevTools
```

### Memory Profiling

```javascript
// Node.js heap snapshot
const v8 = require('v8');
const fs = require('fs');

// Take snapshot
const snapshotFile = v8.writeHeapSnapshot();
console.log(`Heap snapshot written to ${snapshotFile}`);

// Or programmatically
const heapStats = v8.getHeapStatistics();
console.log('Heap used:', heapStats.used_heap_size / 1024 / 1024, 'MB');
```

### Database Profiling

```sql
-- PostgreSQL: Find slow queries
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

-- MySQL: Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- queries over 1 second

-- Explain query plan
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;
```

## Common Performance Bottlenecks

### N+1 Queries

```typescript
// Problem: N+1 queries
async function getOrders() {
  const orders = await db.orders.findAll();  // 1 query
  for (const order of orders) {
    order.user = await db.users.findById(order.userId);  // N queries
  }
  return orders;
}

// Solution: Eager loading
async function getOrders() {
  const orders = await db.orders.findAll({
    include: ['user']  // 1 query with JOIN
  });
  return orders;
}

// Or batch loading
async function getOrders() {
  const orders = await db.orders.findAll();
  const userIds = [...new Set(orders.map(o => o.userId))];
  const users = await db.users.findAll({ where: { id: userIds }});  // 2 queries total
  const userMap = new Map(users.map(u => [u.id, u]));
  orders.forEach(o => o.user = userMap.get(o.userId));
  return orders;
}
```

### Missing Indexes

```sql
-- Problem: Full table scan
SELECT * FROM orders WHERE user_id = 123;
-- If no index on user_id, scans all rows

-- Identify missing indexes
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
-- Shows "Seq Scan" instead of "Index Scan"

-- Solution: Add index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Verify
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
-- Now shows "Index Scan"
```

### Blocking Operations

```typescript
// Problem: Blocking in async context
app.get('/data', async (req, res) => {
  const data = fs.readFileSync('large-file.json');  // Blocks!
  res.json(JSON.parse(data));
});

// Solution: Use async operations
app.get('/data', async (req, res) => {
  const data = await fs.promises.readFile('large-file.json');
  res.json(JSON.parse(data));
});
```

### Unnecessary Work

```typescript
// Problem: Redundant computation
function renderPage(user) {
  const permissions = calculatePermissions(user);  // Expensive
  const header = renderHeader(user, permissions);
  const sidebar = renderSidebar(user, permissions);  // Recalculates internally
  const content = renderContent(user, permissions);  // Recalculates internally
}

// Solution: Calculate once, pass around
function renderPage(user) {
  const permissions = calculatePermissions(user);  // Once
  const header = renderHeader(user, permissions);
  const sidebar = renderSidebar(permissions);  // Passed
  const content = renderContent(permissions);  // Passed
}
```

### Memory-Bound Operations

```typescript
// Problem: Loading entire file into memory
async function processLargeFile(filename) {
  const data = await fs.promises.readFile(filename);  // All in memory
  return processData(data);
}

// Solution: Stream processing
async function processLargeFile(filename) {
  const stream = fs.createReadStream(filename);
  let result = [];
  for await (const chunk of stream) {
    result.push(processChunk(chunk));  // Process incrementally
  }
  return result;
}
```

## Flame Graphs

```
Reading a flame graph:

     ┌─────────────────────────────────────────────────┐
     │                     main                         │
     └─────────────────────────────────────────────────┘
     ┌────────────────────────────┐┌──────────────────┐
     │       processOrders        ││   renderPage     │
     └────────────────────────────┘└──────────────────┘
     ┌──────────┐┌───────────────┐
     │validateDB││  calculateTax │ ← Wide = slow
     └──────────┘└───────────────┘
                 ┌───────────────┐
                 │  dbQuery      │ ← Tallest stack under slow function
                 └───────────────┘

- Width = Time spent
- Height = Call stack depth
- Look for wide bars = where time is spent
- Look at children of wide bars to find root cause
```

## Performance Testing

### Load Testing

```javascript
// Using k6 for load testing
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<200'],  // 95% under 200ms
  },
};

export default function () {
  let res = http.get('https://api.example.com/orders');
  check(res, { 'status was 200': (r) => r.status == 200 });
  sleep(1);
}
```

### Benchmarking

```typescript
// Micro-benchmarking with Node.js
const { performance } = require('perf_hooks');

function benchmark(name, fn, iterations = 1000) {
  const start = performance.now();
  for (let i = 0; i < iterations; i++) {
    fn();
  }
  const end = performance.now();
  console.log(`${name}: ${((end - start) / iterations).toFixed(3)}ms per iteration`);
}

benchmark('Method A', () => methodA());
benchmark('Method B', () => methodB());
```

## Performance Checklist

```markdown
## Before Optimizing
- [ ] Defined clear performance goals
- [ ] Measured baseline performance
- [ ] Profiled to find actual bottleneck

## During Optimization
- [ ] Changing one thing at a time
- [ ] Measuring after each change
- [ ] Documenting what worked/didn't

## After Optimizing
- [ ] Verified improvement meets goal
- [ ] Checked for regressions
- [ ] Added performance tests
- [ ] Documented optimization for future
```
