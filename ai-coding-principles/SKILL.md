---
name: ai-coding-principles
description: Principles and patterns for AI coding agents to write production-quality code. Use when generating code, modifying existing codebases, implementing features, fixing bugs, or when asked about "writing better code", "code quality", "best practices", or "production-ready code".
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
