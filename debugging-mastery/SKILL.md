---
name: debugging-mastery
description: >
  Systematic debugging methodology used by senior engineers at top tech companies.
  Use when investigating bugs, troubleshooting production issues, analyzing error logs,
  debugging performance problems, or when asked about "why isn't this working",
  "how to debug", "root cause analysis", or "production issues".
license: MIT
metadata:
  author: tawanorg
  version: '1.0.0'
---

# Debugging Mastery

Systematic debugging methodology. No guessing—scientific method applied to code.

## The Debugging Mindset

**Debugging is not about fixing. It's about understanding.**

```
Wrong approach:
  Bug reported → Try random fixes → Hope it works → Ship

Right approach:
  Bug reported → Reproduce → Isolate → Understand → Fix → Verify → Prevent
```

Senior engineers spend 80% of debugging time understanding the problem and 20% fixing it. Juniors invert this ratio and end up spending more time overall.

## The Scientific Method for Debugging

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE DEBUGGING LOOP                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. OBSERVE                                                    │
│      └── What exactly is happening? (not what you think)        │
│                                                                 │
│   2. HYPOTHESIZE                                                │
│      └── What could cause this? (list multiple hypotheses)      │
│                                                                 │
│   3. PREDICT                                                    │
│      └── If hypothesis X is true, what else should be true?     │
│                                                                 │
│   4. TEST                                                       │
│      └── Design experiment to confirm/refute hypothesis         │
│                                                                 │
│   5. CONCLUDE                                                   │
│      └── Was hypothesis correct? If not, return to step 2       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 1: Reproduce

**If you can't reproduce it, you can't fix it.**

### Reproduction Checklist

```markdown
- [ ] Can I reproduce locally?
- [ ] Can I reproduce consistently (not flaky)?
- [ ] What's the minimal reproduction case?
- [ ] What are the exact steps?
- [ ] What's the environment? (OS, versions, config)
- [ ] What's the input data?
- [ ] What's the expected vs actual behavior?
```

### Reproduction Strategies

| Situation | Strategy |
|-----------|----------|
| Can reproduce locally | Ideal—proceed to isolate |
| Only in production | Add logging, capture state, replicate data |
| Intermittent/flaky | Increase frequency: loops, load, timing changes |
| User-reported | Get exact steps, screenshots, network logs |
| Race condition | Add delays, increase concurrency, stress test |

### The Minimal Reproduction

Strip away everything unnecessary:

```
Full system:         Minimal repro:
┌────────────┐       ┌────────────┐
│ Frontend   │       │            │
│ + API      │  ───► │  Single    │
│ + Auth     │       │  Function  │
│ + Cache    │       │  + Input   │
│ + Database │       │            │
└────────────┘       └────────────┘
```

**Why minimal?**
- Fewer variables = faster isolation
- Shareable with others
- Becomes regression test
- Often reveals the bug itself

## Phase 2: Isolate

**Binary search the problem space.**

### Binary Search Debugging

```
Don't: Check every line sequentially (O(n))
Do:    Binary search the problem space (O(log n))

┌─────────────────────────────────────────────┐
│              Entire codebase                │
└─────────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
┌─────────────────┐     ┌─────────────────┐
│   First half    │     │   Second half   │
│   (works?)      │     │   (works?)      │
└─────────────────┘     └─────────────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│ 1/4   │ │ 2/4   │    ← Bug is here
└───────┘ └───────┘
```

### Isolation Techniques

| Technique | How | When |
|-----------|-----|------|
| **Git bisect** | Binary search commits | Bug worked before, broken now |
| **Comment out** | Disable code sections | Isolate problematic component |
| **Hardcode** | Replace variables with constants | Isolate data vs logic |
| **Mock** | Replace dependencies | Isolate integration vs logic |
| **Simplify** | Remove features until bug disappears | Find minimum trigger |

### Git Bisect Example

```bash
git bisect start
git bisect bad                 # Current commit is broken
git bisect good abc123         # This commit worked
# Git checks out middle commit
# Test and mark:
git bisect good                # or git bisect bad
# Repeat until found
git bisect reset
```

## Phase 3: Understand

**Don't fix what you don't understand.**

### Read the Error Message

```
90% of developers don't actually read error messages.

Error: Cannot read property 'map' of undefined
       at UserList (UserList.js:15:23)
       at renderWithHooks (react-dom.js:1234)
       at mountIndeterminateComponent (react-dom.js:5678)

This tells you:
1. WHAT: Trying to call .map() on undefined
2. WHERE: UserList.js, line 15, column 23
3. CONTEXT: During React render

Don't guess. Read.
```

### Understanding Questions

Before fixing, answer:

1. **What is the expected behavior?**
2. **What is the actual behavior?**
3. **What changed?** (code, data, environment, dependencies)
4. **Where does the incorrect state originate?**
5. **Why does the code behave this way?** (not "why is it broken")

### Trace the Data Flow

```
Input → Transform → Transform → Transform → Output
  ↓         ↓           ↓           ↓         ↓
  ✓         ✓           ✗           ?         ?
                        │
                   Bug is here
                   (first incorrect state)
```

**Find the first point where data becomes incorrect.**

## Phase 4: Debug Tooling

### Debugger Over console.log

```javascript
// Junior approach: Spray and pray
console.log('here 1');
console.log('data:', data);
console.log('here 2');
console.log('wtf');

// Senior approach: Strategic breakpoints
debugger;  // Pause execution
// Inspect call stack
// Inspect variables
// Step through code
// Watch expressions
```

### When to Use Each Tool

| Tool | Use For |
|------|---------|
| **Debugger** | Understanding control flow, inspecting state |
| **Logging** | Production issues, async flows, distributed systems |
| **Profiler** | Performance issues, memory leaks |
| **Network tab** | API issues, request/response inspection |
| **Memory snapshot** | Memory leaks, object retention |
| **Flame graph** | CPU profiling, hot paths |

### Strategic Logging

```typescript
// Bad: Noise
console.log('processing');
console.log(data);

// Good: Structured, contextual, actionable
logger.debug('Processing order', {
  orderId: order.id,
  itemCount: order.items.length,
  totalAmount: order.total,
  customerId: order.customerId,
  timestamp: Date.now()
});
```

## Phase 5: Common Bug Patterns

### The Usual Suspects

| Bug Pattern | Symptom | First Check |
|-------------|---------|-------------|
| **Null/undefined** | "Cannot read property X" | Check if value exists before use |
| **Off-by-one** | Wrong count, missing item | Array bounds, loop conditions |
| **Race condition** | Intermittent failures | Async timing, shared state |
| **State mutation** | Unpredictable behavior | Who else modifies this state? |
| **Cache staleness** | Old data showing | Cache invalidation, TTLs |
| **Type coercion** | Weird comparisons | `===` vs `==`, string vs number |
| **Floating point** | Math doesn't add up | Use integers for money, compare with epsilon |
| **Timezone** | Wrong time/date | Store UTC, convert on display |
| **Encoding** | Garbled text, URL issues | UTF-8, URL encoding |
| **Memory leak** | Growing memory, OOM | Event listeners, closures, caches |

### Null Reference Checklist

```typescript
// When you see: Cannot read property 'X' of undefined

// Check the chain:
user.profile.settings.theme
// ↓
user          // defined?
user.profile  // defined?
user.profile.settings  // defined?

// Use optional chaining:
user?.profile?.settings?.theme

// Or guard clauses:
if (!user?.profile?.settings) {
  return defaultTheme;
}
```

### Race Condition Checklist

```typescript
// Symptoms:
// - Works sometimes, fails sometimes
// - Works locally, fails in production
// - Works slow, fails fast

// Check for:
// 1. Shared mutable state
// 2. Missing await
// 3. Out-of-order execution
// 4. Stale closures

// Example: Stale closure in React
useEffect(() => {
  const interval = setInterval(() => {
    setCount(count + 1);  // Bug: captures stale count
  }, 1000);
  return () => clearInterval(interval);
}, []);  // Missing dependency

// Fix:
setCount(prev => prev + 1);  // Use functional update
```

## Phase 6: Production Debugging

### Production is Different

```
Local:                    Production:
- Full access             - Limited access
- debugger works          - No debugger
- See all state           - Only logs/metrics
- Can change code         - Can't redeploy quickly
- One user                - Millions of users
- Clean data              - Messy data
```

### Production Debugging Toolkit

| Tool | Purpose |
|------|---------|
| **Structured logs** | Search and filter events |
| **Distributed tracing** | Follow request across services |
| **Error tracking** | Aggregate and group errors (Sentry, etc.) |
| **Metrics/dashboards** | Spot anomalies, correlate timing |
| **Feature flags** | Disable suspected code |
| **Canary deploys** | Limit blast radius |

### The Production Debugging Flow

```
1. ASSESS IMPACT
   - How many users affected?
   - Is it getting worse?
   - Can we mitigate without fixing?

2. GATHER EVIDENCE
   - Logs around the time of errors
   - Metrics showing anomalies
   - User reports with context
   - Recent deployments

3. FORM HYPOTHESES
   - What changed recently?
   - What's different about affected users?
   - What does the error tell us?

4. SAFE INVESTIGATION
   - Read-only queries
   - Feature flags to disable
   - Increase logging temporarily
   - Never debug by deploying fixes blindly

5. FIX AND VERIFY
   - Fix in staging first
   - Canary deployment
   - Monitor metrics
   - Confirm user reports stop
```

## Phase 7: Distributed Systems Debugging

### Following a Request

```
┌────────┐   ┌─────────┐   ┌────────┐   ┌────────┐
│ Client │──►│ Gateway │──►│ API    │──►│ DB     │
└────────┘   └─────────┘   └────────┘   └────────┘
    │             │             │            │
    └─────────────┴─────────────┴────────────┘
                  Request ID: abc-123
                  (trace this through all logs)
```

### Correlation IDs

```typescript
// Generate at entry point
const requestId = generateRequestId();

// Pass through all calls
await orderService.create(order, { requestId });
await paymentService.charge(amount, { requestId });
await notificationService.send(email, { requestId });

// Include in all logs
logger.info('Processing order', { requestId, orderId });

// Now you can grep:
// grep "abc-123" *.log
```

### Distributed Debugging Checklist

```markdown
- [ ] Is the request reaching the service?
- [ ] Is the request well-formed?
- [ ] Are downstream services responding?
- [ ] Are timeouts configured correctly?
- [ ] Are retries happening? Too many?
- [ ] Is there network partition?
- [ ] Is there resource exhaustion?
- [ ] Are there version mismatches?
```

## Phase 8: Rubber Duck Debugging

**Explain the bug out loud.**

```
"Okay duck, so the user clicks submit, then we call the API,
then we... wait, we don't await the response before redirecting.
Never mind duck, I found it."
```

This works because:
1. Forces you to articulate assumptions
2. Slows down your thinking
3. Often reveals gaps in understanding
4. No actual duck required (but recommended)

## Phase 9: When You're Stuck

### The Stuck Checklist

```markdown
1. [ ] Have I actually read the error message?
2. [ ] Am I debugging the right thing?
3. [ ] Have I reproduced it reliably?
4. [ ] Have I isolated the problem?
5. [ ] Have I checked recent changes?
6. [ ] Have I searched for the error?
7. [ ] Have I taken a break?
8. [ ] Have I asked someone else to look?
```

### Fresh Eyes Techniques

- **Take a break** - Walk away for 15 minutes
- **Explain to someone** - Rubber duck or real person
- **Question assumptions** - What are you certain of? Are you sure?
- **Start over** - Fresh approach, ignore previous attempts
- **Sleep on it** - Seriously, it works

## Phase 10: Post-Debugging

### The Fix Checklist

```markdown
Before committing the fix:
- [ ] Does it actually fix the bug?
- [ ] Does it fix the root cause (not just symptom)?
- [ ] Does it break anything else?
- [ ] Is there a regression test?
- [ ] Is the fix the simplest solution?
- [ ] Would future maintainers understand it?
```

### Write a Regression Test

```typescript
// The bug you just fixed should never come back
describe('UserList', () => {
  it('handles empty user array without crashing', () => {
    // This was the bug: .map() on undefined
    const { container } = render(<UserList users={[]} />);
    expect(container).toBeInTheDocument();
  });
});
```

### Document for Future

```typescript
// If the bug was subtle, leave a comment
// Bug fix: users can be undefined during initial load
// See: JIRA-1234, postmortem doc
const userList = users ?? [];
```

## Quick Reference

### The 5 Whys

```
Problem: Website is down
Why? → Server crashed
Why? → Out of memory
Why? → Memory leak
Why? → Event listeners not cleaned up
Why? → Missing cleanup in useEffect

Root cause: Missing useEffect cleanup
```

### Debugging Commands Cheat Sheet

```bash
# Git
git bisect start/good/bad/reset   # Binary search commits
git log --oneline -20             # Recent commits
git diff HEAD~5                   # Changes in last 5 commits
git blame <file>                  # Who changed what

# Logs
tail -f /var/log/app.log          # Follow logs
grep -r "error" ./logs            # Search logs
journalctl -u myservice -f        # System logs

# Network
curl -v https://api.example.com   # Verbose request
nc -zv host port                  # Check connectivity
tcpdump -i any port 8080          # Packet capture

# Process
ps aux | grep node                # Find processes
lsof -i :3000                     # What's on port 3000
top -o mem                        # Memory usage
```

## Rules Reference

| Rule | File |
|------|------|
| Reproduce Before Fix | `rules/reproduce-before-fix.md` |
| Binary Search First | `rules/binary-search-first.md` |
| Read Error Messages | `rules/read-error-messages.md` |
| Check Recent Changes | `rules/check-recent-changes.md` |
| One Change at a Time | `rules/one-change-at-a-time.md` |
| Verify the Fix | `rules/verify-the-fix.md` |
| Write Regression Test | `rules/write-regression-test.md` |
| Document Root Cause | `rules/document-root-cause.md` |
| Never Debug Blind | `rules/never-debug-blind.md` |
| Question Assumptions | `rules/question-assumptions.md` |

## References

| Topic | File |
|-------|------|
| Production Debugging | `references/production-debugging.md` |
| Performance Debugging | `references/performance-debugging.md` |
| Memory Leak Detection | `references/memory-leak-detection.md` |
| Distributed Tracing | `references/distributed-tracing.md` |
| Common Bug Patterns | `references/common-bug-patterns.md` |
| Debugging Tools Guide | `references/debugging-tools.md` |
