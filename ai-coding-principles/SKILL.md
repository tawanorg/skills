---
name: ai-coding-principles
description: >
  Principles and patterns for AI coding agents to write production-quality code.
  Use when generating code, modifying existing codebases, implementing features,
  fixing bugs, or when asked about "writing better code", "code quality",
  "best practices", or "production-ready code".
license: MIT
metadata:
  author: tawanorg
  version: '1.0.0'
---

# AI Coding Principles

Actionable principles for AI coding agents to write production-quality code.

## Golden Rules

These four rules apply to every coding task:

### 1. Read Before Writing

**Always explore the codebase before generating code.**

- Search for similar implementations first
- Understand existing patterns and conventions
- Check how errors are handled elsewhere
- Look at test patterns already in use

```
WRONG: Immediately generating a new utility function
RIGHT: Searching for existing utilities, then extending or reusing
```

### 2. Minimal Change Principle

**Make the smallest change that solves the problem.**

- One bug fix = one focused change
- Don't refactor unrelated code
- Don't add features not requested
- Don't "improve" working code opportunistically

```
WRONG: Fixing a typo and also reformatting the file
RIGHT: Fixing only the typo
```

### 3. Match the Codebase

**Consistency over personal preference.**

| If the codebase uses... | Use that, not... |
|-------------------------|------------------|
| `snake_case` | `camelCase` |
| Tabs | Spaces |
| Single quotes | Double quotes |
| Class components | Functional components |
| Callbacks | Promises |

Even if you "know better," match existing patterns unless explicitly asked to migrate.

### 4. Verify Understanding

**Ask when uncertain. Assumptions cause rework.**

Ask before proceeding when:
- Requirements are ambiguous
- Multiple valid approaches exist
- Change might break existing functionality
- You're unsure about business logic

Don't ask when:
- Implementation is straightforward
- You can verify with tests
- The choice is purely stylistic

## Error Handling

> See `references/error-handling-patterns.md` for language-specific examples.

### Principles

1. **Explicit over silent** - Never swallow errors without logging
2. **Context is required** - Include what failed, where, and why
3. **Fail fast in dev** - Crash early to surface bugs
4. **Graceful in prod** - Degrade functionality, don't crash users

### Good vs Bad

```typescript
// BAD: Silent failure, no context
try {
  await saveUser(data);
} catch (e) {
  return null;
}

// GOOD: Explicit handling with context
try {
  await saveUser(data);
} catch (error) {
  logger.error('Failed to save user', {
    userId: data.id,
    error: error.message,
    stack: error.stack,
  });
  throw new UserSaveError(`Could not save user ${data.id}`, { cause: error });
}
```

### Error Handling Checklist

- [ ] Are all errors logged with context?
- [ ] Do error messages help debugging?
- [ ] Are expected errors handled differently from unexpected?
- [ ] Is sensitive data excluded from error messages?

## Security Fundamentals

> See `references/security-checklist.md` for the full checklist.

### Non-Negotiable Rules

1. **Validate all external input** - User input, API responses, file contents
2. **Use parameterized queries** - Never concatenate SQL
3. **Manage secrets properly** - Environment variables, never in code
4. **Never log sensitive data** - Passwords, tokens, PII

### Quick Security Check

Before completing any task:

```
[ ] User input is validated and sanitized
[ ] No secrets in code or logs
[ ] SQL uses parameterized queries
[ ] File paths are validated (no path traversal)
[ ] External data is treated as untrusted
```

### Common Vulnerabilities to Avoid

| Vulnerability | Prevention |
|--------------|------------|
| SQL Injection | Parameterized queries |
| XSS | Output encoding, CSP |
| Path Traversal | Validate file paths |
| Command Injection | Avoid shell commands, use APIs |
| Secret Exposure | Environment variables, secret managers |

## Testing Philosophy

> See `references/testing-patterns.md` for implementation patterns.

### What to Test

**Critical paths:**
- User authentication flows
- Payment/transaction logic
- Data persistence operations
- Core business rules

**Edge cases:**
- Empty inputs
- Boundary values
- Error conditions
- Concurrent operations

### What Not to Over-Test

- Trivial getters/setters
- Framework code
- Third-party libraries
- Pure UI styling

### Test Placement

| Test Type | When to Use |
|-----------|-------------|
| Unit tests | Pure functions, isolated logic |
| Integration tests | API endpoints, database operations |
| E2E tests | Critical user journeys only |

### Testing Checklist

- [ ] Are critical paths covered?
- [ ] Do tests verify behavior, not implementation?
- [ ] Are tests independent and repeatable?
- [ ] Do tests have meaningful names?

## Performance Considerations

### Profile First

Never optimize without evidence. The bottleneck is rarely where you think.

1. Measure current performance
2. Identify actual bottleneck
3. Fix the bottleneck
4. Measure again

### Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| N+1 queries | Batch fetching, eager loading |
| Memory leaks | Clean up listeners, close connections |
| Blocking I/O | Async operations |
| Repeated computation | Caching, memoization |
| Large payloads | Pagination, compression |

### Performance Checklist

- [ ] Database queries are batched where possible
- [ ] Large lists are paginated
- [ ] Resources are properly cleaned up
- [ ] Heavy computations are cached if repeated

## Communication Patterns

### When to Ask vs Proceed

**Ask when:**
- Multiple valid implementations exist
- Requirements seem incomplete
- Change impacts other systems
- Business logic is unclear

**Proceed when:**
- Implementation is obvious
- Tests can validate correctness
- Change is isolated and reversible
- Following existing patterns

### Explaining Changes

When describing what you did:

1. **What** - The specific change made
2. **Why** - The reason for this approach
3. **Impact** - What else might be affected
4. **Testing** - How to verify it works

```
WRONG: "Fixed the bug"
RIGHT: "Fixed null pointer in UserService.getProfile() by adding
       null check before accessing user.settings. This was
       triggered when users hadn't completed onboarding."
```

---

## Enterprise Patterns

The following sections cover enterprise-grade patterns essential for production systems.

## Architecture & SOLID

> See `references/architecture-patterns.md` for detailed examples.

### SOLID Quick Reference

| Principle | Rule | Violation Sign |
|-----------|------|----------------|
| **S**ingle Responsibility | One reason to change | Class doing validation, persistence, and notifications |
| **O**pen/Closed | Extend, don't modify | Adding `else if` branches for new types |
| **L**iskov Substitution | Subtypes are substitutable | Subclass throws "not implemented" |
| **I**nterface Segregation | Small, focused interfaces | Implementing methods that throw |
| **D**ependency Inversion | Depend on abstractions | `new ConcreteClass()` inside business logic |

### Layer Placement

| If you're writing... | It belongs in... |
|---------------------|------------------|
| Business rules | Domain layer |
| Use case orchestration | Application layer |
| Database queries | Infrastructure layer |
| HTTP handling | Presentation layer |

## Observability

> See `references/observability-patterns.md` for implementation patterns.

### The Three Pillars

| Pillar | Purpose | Question Answered |
|--------|---------|-------------------|
| **Logs** | Discrete events | What happened? |
| **Metrics** | Aggregated measurements | How is it performing? |
| **Traces** | Request flow | Where did time go? |

### Logging Rules

1. **Structured JSON** - Not string concatenation
2. **Request ID** - Propagate through all logs
3. **Context** - Include who, what, where, when
4. **Levels** - error (failures), warn (degraded), info (business events), debug (diagnostics)

### Four Golden Signals

Monitor these for every service:
- **Latency** - Request duration
- **Traffic** - Request rate
- **Errors** - Failure rate
- **Saturation** - Resource utilization

## API Design

> See `references/api-design-patterns.md` for conventions and examples.

### REST Conventions

```
GET    /resources          List
POST   /resources          Create
GET    /resources/{id}     Read
PUT    /resources/{id}     Replace
PATCH  /resources/{id}     Update
DELETE /resources/{id}     Delete
```

### Response Format

```typescript
// Success
{ "data": {...}, "meta": { "requestId": "..." } }

// Error
{ "error": { "code": "USER_NOT_FOUND", "message": "...", "requestId": "..." } }
```

### API Checklist

- [ ] Correct HTTP methods and status codes
- [ ] Request validation with clear errors
- [ ] Consistent error format with codes
- [ ] Pagination for collections
- [ ] Versioning strategy defined

## Database Patterns

> See `references/database-patterns.md` for schema design and optimization.

### Schema Essentials

```sql
-- Every table needs
id            -- Primary key (UUID preferred)
created_at    -- When created
updated_at    -- When modified
-- Optional
deleted_at    -- Soft delete
```

### Query Rules

| Do | Don't |
|----|-------|
| Select specific columns | `SELECT *` |
| Batch related queries | N+1 queries in loops |
| Use cursor pagination | Offset at scale |
| Index foreign keys | Forget WHERE clause indexes |
| Parameterized queries | String concatenation |

## Resilience Patterns

> See `references/resilience-patterns.md` for implementation details.

### Everything Fails. Design for It.

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| **Retry** | Handle transient failures | Network timeouts, 503s |
| **Circuit Breaker** | Stop cascading failures | Repeated downstream failures |
| **Timeout** | Prevent hanging | Every external call |
| **Bulkhead** | Isolate failures | Critical vs non-critical paths |
| **Fallback** | Graceful degradation | Non-critical features |

### Retry Guidelines

```
Retry: 503, 504, timeouts, connection errors
Don't retry: 400, 401, 403, 404, 422
Backoff: Exponential with jitter
Max attempts: 3-5 for most cases
```

## Code Review

> See `references/code-review-checklist.md` for the complete checklist.

### Review Focus Areas

1. **Correctness** - Does it work? Edge cases?
2. **Design** - Right place? Right abstraction?
3. **Readability** - Can you understand it?
4. **Security** - Input validated? Auth checked?
5. **Testing** - Critical paths covered?

### Before Submitting (Self-Review)

- [ ] I've tested this locally
- [ ] I've reviewed my own diff
- [ ] No debugging code left in
- [ ] PR description explains what and why

## Refactoring

> See `references/refactoring-patterns.md` for safe refactoring techniques.

### The Golden Rule

**Make the change easy, then make the easy change.**

### When to Refactor

| Refactor When | Don't Refactor When |
|---------------|---------------------|
| Adding features is hard | No tests exist |
| Same bug keeps appearing | Don't understand the code |
| Code is hard to test | Tight deadline |
| You have test coverage | Part of unrelated change |

### Safe Refactoring Process

1. **Verify test coverage exists**
2. **Small incremental changes** - commit frequently
3. **Keep behavior unchanged** - tests should pass
4. **Separate from features** - refactor PRs are pure refactoring

---

## Quick Reference Checklist

Run through before completing any task:

### Pre-Change
- [ ] Read existing code in the area
- [ ] Understand current patterns
- [ ] Clarify ambiguous requirements

### During Change
- [ ] Minimal necessary changes
- [ ] Match codebase style
- [ ] Handle errors explicitly
- [ ] Validate external input
- [ ] No secrets in code

### Post-Change
- [ ] Tests pass
- [ ] No regressions introduced
- [ ] Changes are explained clearly

## Principles Summary

### Core Principles

| Principle | What It Means | Anti-Pattern |
|-----------|---------------|--------------|
| Read Before Write | Explore similar code first | Generating code immediately |
| Minimal Change | Smallest fix possible | Rewriting file for typo |
| Match Codebase | Follow existing patterns | Using camelCase in snake_case project |
| Explicit Errors | Never swallow silently | `catch (e) { return null }` |
| Security Default | Validate all external input | Trusting user data |
| Test Critical Paths | Focus on what matters | 100% coverage goal |
| Profile First | Measure before optimizing | Premature optimization |
| Ask When Uncertain | Clarify before coding | Assuming requirements |

### Enterprise Principles

| Principle | What It Means | Anti-Pattern |
|-----------|---------------|--------------|
| SOLID Architecture | Single responsibility, depend on abstractions | God classes, tight coupling |
| Observable Systems | Structured logs, metrics, traces | `console.log("here")` |
| API Consistency | Standard conventions, clear errors | Random status codes, unstructured errors |
| Database Discipline | Indexes, parameterized queries, migrations | N+1 queries, string concatenation |
| Design for Failure | Timeouts, retries, circuit breakers | Assuming dependencies are reliable |
| Review Before Merge | Systematic code review | LGTM without reading |
| Refactor Safely | Tests first, small changes, separate PRs | Big bang rewrites |

## Reference Files

Comprehensive guides for deep understanding:

| Topic | File | Use When |
|-------|------|----------|
| Error Handling | `references/error-handling-patterns.md` | Implementing error handling |
| Security | `references/security-checklist.md` | Security review, input handling |
| Testing | `references/testing-patterns.md` | Writing tests |
| Architecture | `references/architecture-patterns.md` | Designing components, applying SOLID |
| Observability | `references/observability-patterns.md` | Adding logging, metrics, tracing |
| API Design | `references/api-design-patterns.md` | Building REST APIs |
| Database | `references/database-patterns.md` | Schema design, query optimization |
| Resilience | `references/resilience-patterns.md` | Handling failures, external calls |
| Code Review | `references/code-review-checklist.md` | Reviewing code |
| Refactoring | `references/refactoring-patterns.md` | Improving existing code |

## Rules

Atomic, enforceable rules for code review. Each rule follows: Incorrect → Correct → Why.

### Security (CRITICAL)

| Rule | File |
|------|------|
| Always Use Parameterized Queries | `rules/security-parameterize-queries.md` |
| Validate All External Input | `rules/security-validate-input.md` |
| Never Hardcode Secrets | `rules/security-no-secrets-in-code.md` |
| Never Log Sensitive Data | `rules/security-no-sensitive-logs.md` |

### Error Handling (HIGH)

| Rule | File |
|------|------|
| Never Swallow Errors Silently | `rules/error-never-swallow.md` |
| Include Context in Error Messages | `rules/error-include-context.md` |

### Database (HIGH)

| Rule | File |
|------|------|
| Never Use SELECT * | `rules/db-no-select-star.md` |
| Prevent N+1 Query Problems | `rules/db-prevent-n-plus-one.md` |
| Always Index Foreign Keys | `rules/db-index-foreign-keys.md` |

### API Design (HIGH)

| Rule | File |
|------|------|
| Use Correct HTTP Status Codes | `rules/api-correct-status-codes.md` |
| Use Consistent Error Format | `rules/api-consistent-errors.md` |
| Validate All Request Bodies | `rules/api-validate-requests.md` |

### Architecture (HIGH)

| Rule | File |
|------|------|
| Single Responsibility Principle | `rules/arch-single-responsibility.md` |
| Depend on Abstractions | `rules/arch-depend-on-abstractions.md` |

### Testing (MEDIUM)

| Rule | File |
|------|------|
| Test Behavior, Not Implementation | `rules/test-behavior-not-implementation.md` |
| Prioritize Critical Paths | `rules/test-critical-paths.md` |

### Observability (MEDIUM)

| Rule | File |
|------|------|
| Use Structured Logging | `rules/obs-structured-logging.md` |
| Include Request ID in All Logs | `rules/obs-include-request-id.md` |

### Resilience (MEDIUM)

| Rule | File |
|------|------|
| Always Set Timeouts | `rules/resilience-always-timeout.md` |
| Only Retry Transient Failures | `rules/resilience-retry-transient.md` |

### Refactoring (MEDIUM)

| Rule | File |
|------|------|
| Have Tests Before Refactoring | `rules/refactor-tests-first.md` |
| Make Small, Incremental Commits | `rules/refactor-small-commits.md` |
