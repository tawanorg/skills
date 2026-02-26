# Production Debugging

Debugging in production is fundamentally different from local debugging. You have limited access, real users affected, and no debugger.

## The Production Debugging Mindset

```
Local debugging:         Production debugging:
- Full access            - Read-only mindset
- Debugger available     - Logs and metrics only
- Can break things       - Cannot break things
- Take your time         - Time pressure
- One user (you)         - Millions of users
```

## The Production Debugging Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCTION INCIDENT FLOW                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. ASSESS                                                      │
│     └── Impact? Scope? Trend?                                   │
│                                                                 │
│  2. MITIGATE                                                    │
│     └── Can we reduce impact without fixing?                    │
│                                                                 │
│  3. INVESTIGATE                                                 │
│     └── Gather evidence from logs, metrics, traces              │
│                                                                 │
│  4. HYPOTHESIZE                                                 │
│     └── What could cause this? (based on evidence)              │
│                                                                 │
│  5. VALIDATE                                                    │
│     └── Safe experiments to confirm hypothesis                  │
│                                                                 │
│  6. FIX                                                         │
│     └── Minimal fix, staged rollout                             │
│                                                                 │
│  7. VERIFY                                                      │
│     └── Confirm resolution via metrics                          │
│                                                                 │
│  8. POSTMORTEM                                                  │
│     └── Document and prevent recurrence                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Assessment Phase

### Impact Assessment Checklist

```markdown
- [ ] How many users are affected?
- [ ] What percentage of requests are failing?
- [ ] Is the problem getting worse?
- [ ] What's the business impact? (revenue, reputation)
- [ ] Which features/services are affected?
- [ ] When did it start? (correlate with events)
```

### Severity Levels

| Severity | Description | Response |
|----------|-------------|----------|
| SEV1 | Complete outage, all users affected | All hands, immediate escalation |
| SEV2 | Major feature broken, many users | Dedicated team, page oncall |
| SEV3 | Partial impact, workaround exists | Normal priority investigation |
| SEV4 | Minor issue, few users affected | Standard bug process |

## Mitigation Strategies

Before fixing, consider if you can reduce impact:

```markdown
## Quick Mitigations

### Traffic Management
- [ ] Route traffic away from failing service
- [ ] Enable maintenance page
- [ ] Reduce traffic via feature flag

### Rollback
- [ ] Rollback to previous version
- [ ] Revert recent config change
- [ ] Restore from database backup

### Circuit Breaking
- [ ] Disable failing feature
- [ ] Return cached/stale data
- [ ] Failover to backup system

### Scaling
- [ ] Scale up resources
- [ ] Add capacity
- [ ] Clear caches
```

## Evidence Gathering

### Log Analysis

```bash
# Find errors around incident time
grep -E "ERROR|Exception|FATAL" /var/log/app.log | \
  grep "2024-01-15 14:" | \
  head -100

# Count errors by type
grep "ERROR" /var/log/app.log | \
  grep "2024-01-15" | \
  sed 's/.*ERROR \([^:]*\):.*/\1/' | \
  sort | uniq -c | sort -rn

# Find first occurrence
grep "specific error message" /var/log/app.log | head -1

# Follow logs in real-time
tail -f /var/log/app.log | grep --line-buffered "ERROR"
```

### Metrics Analysis

```markdown
Key metrics to check:

### Request Metrics
- Request rate (is traffic unusual?)
- Error rate (4xx, 5xx)
- Latency percentiles (p50, p95, p99)

### Resource Metrics
- CPU usage
- Memory usage
- Disk I/O
- Network I/O

### Application Metrics
- Queue depths
- Connection pool usage
- Cache hit rates
- Database query times

### Correlation
- Plot metrics vs. deployment times
- Plot metrics vs. traffic changes
- Compare healthy vs. unhealthy instances
```

### Trace Analysis

```
Follow a request through the system:

Request ID: abc-123
────────────────────────────────────────────────────────────────
14:23:01.001 │ GATEWAY    │ Received POST /api/orders
14:23:01.003 │ GATEWAY    │ Auth validated
14:23:01.010 │ ORDERS     │ CreateOrder called
14:23:01.015 │ ORDERS     │ Validating order...
14:23:01.050 │ INVENTORY  │ CheckStock called
14:23:01.055 │ INVENTORY  │ ERROR: Connection refused
14:23:01.056 │ ORDERS     │ ERROR: Inventory check failed
14:23:01.057 │ GATEWAY    │ Returning 503
────────────────────────────────────────────────────────────────
                                      │
                           Problem identified: Inventory service unreachable
```

## Safe Investigation Techniques

### Read-Only Operations

```bash
# Safe: Read-only queries
SELECT COUNT(*) FROM orders WHERE status = 'failed';
SELECT * FROM orders WHERE id = 123 LIMIT 1;

# Dangerous: Never in production investigation
# UPDATE, DELETE, TRUNCATE
# DROP, ALTER
```

### Feature Flags for Debugging

```typescript
// Enable verbose logging for specific request
if (featureFlags.isEnabled('debug_order_flow', { userId })) {
  logger.setLevel('DEBUG');
  logger.debug('Full order state', { order, user, context });
}

// Disable suspected problematic code
if (!featureFlags.isEnabled('new_payment_processor')) {
  return legacyPaymentProcessor.charge(amount);
}
```

### Canary Debugging

```
Deploy debug version to small percentage:

100% traffic ─────────────────────────────┐
                                          │
     ┌────────────────────────────────────┤ 99%
     │         Production v1.0            │
     └────────────────────────────────────┤
                                          │
     ┌────────────────────────────────────┤ 1%
     │    Debug v1.0 (verbose logging)    │
     └────────────────────────────────────┘

Now you have detailed logs for 1% without risking production.
```

## Common Production Issues

### Memory Issues

```markdown
Symptoms:
- OOM kills
- Increasing memory over time
- GC pauses

Investigation:
1. Check memory metrics trend
2. Take heap dump (if safe)
3. Analyze object retention
4. Check for missing cleanup
```

### Connection Exhaustion

```markdown
Symptoms:
- "Connection refused" errors
- Timeouts
- Increasing latency

Investigation:
1. Check connection pool metrics
2. Count open connections: `netstat -an | grep ESTABLISHED | wc -l`
3. Check for connection leaks
4. Verify pool size vs load
```

### Database Issues

```markdown
Symptoms:
- Slow queries
- Timeouts
- Lock contention

Investigation:
1. Check slow query log
2. EXPLAIN problematic queries
3. Check for table locks
4. Monitor connection count
5. Check replication lag
```

### External Service Issues

```markdown
Symptoms:
- Timeout errors
- HTTP 5xx from dependency
- Partial failures

Investigation:
1. Check dependency status page
2. Test direct connectivity
3. Check SSL certificate expiry
4. Verify credentials still valid
5. Check rate limits
```

## Post-Incident

### Verification

```markdown
After deploying fix:
- [ ] Error rate decreased
- [ ] Latency normalized
- [ ] User reports stopped
- [ ] Metrics match baseline
- [ ] No new issues introduced
```

### Communication

```markdown
## Incident Update Template

**Status:** [Investigating | Identified | Monitoring | Resolved]

**Impact:** [Description of user impact]

**Current state:** [What's happening now]

**Next update:** [Time of next update]

---

Example:
**Status:** Identified
**Impact:** ~5% of checkout requests failing
**Current state:** Identified database connection issue, deploying fix
**Next update:** 15 minutes
```
