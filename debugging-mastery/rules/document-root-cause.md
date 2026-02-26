---
title: Document Root Cause
impact: medium
tags: [debugging, documentation, learning]
---

# Document Root Cause

**Don't just fix the bug. Document what caused it and how you found it.**

## The Rule

For every non-trivial bug fix:
1. Document the root cause (not just the symptom)
2. Document how you diagnosed it
3. Share the knowledge with the team

## Why This Matters

```
Undocumented fixes:
- Knowledge stays with one person
- Team repeats same mistakes
- Similar bugs get re-debugged from scratch
- No pattern recognition across incidents

Documented fixes:
- Team learns from each bug
- Patterns become visible
- Future debugging is faster
- Prevents similar bugs
```

## Levels of Documentation

### Level 1: Commit Message

```bash
# Bad
git commit -m "Fix bug"
git commit -m "Fix login issue"

# Good
git commit -m "Fix login failure for emails with + character

Root cause: Email validation regex was missing + in the
allowed character class, causing valid emails like
user+tag@example.com to be rejected.

Diagnosis: Reproduced with test email, traced through
validation flow, found regex pattern at validation.ts:23.

Prevention: Added regression test, updated email format
documentation.

Fixes: BUG-1234"
```

### Level 2: PR Description

```markdown
## Bug Fix: Login failure for certain email formats

### Symptoms
Users with + in their email address could not log in.
Error: "Invalid email format"

### Root Cause
The email validation regex `^[a-zA-Z0-9._-]+@...` was missing
the `+` character, which is valid in email local parts per RFC 5321.

### Diagnosis Steps
1. Reproduced with email `test+tag@example.com`
2. Traced through login flow with debugger
3. Found validation failing at `src/validation.ts:23`
4. Identified missing character in regex

### Fix
Added `+` to the character class: `^[a-zA-Z0-9._%+-]+@...`

### Prevention
- Added regression test
- Updated email validation documentation
- Consider: Use established email validation library instead?

### References
- RFC 5321 Section 4.1.2 (email address format)
- BUG-1234
```

### Level 3: Postmortem (for serious bugs)

```markdown
# Postmortem: Authentication Outage 2024-01-15

## Summary
Users unable to log in for 45 minutes due to email validation bug.

## Impact
- Duration: 45 minutes
- Users affected: ~5,000 login attempts failed
- Revenue impact: Estimated $X in lost conversions

## Timeline
- 14:00 - Deploy v2.3.1 with validation changes
- 14:15 - First user reports login failure
- 14:20 - Oncall alerted, investigation begins
- 14:35 - Root cause identified
- 14:40 - Hotfix deployed
- 14:45 - Service restored

## Root Cause
Refactored email validation introduced regression.
Character class missing `+` which was present in original.
Code review didn't catch because no test covered this case.

## Detection
User reports to support. No automated detection.

## Resolution
Hotfix to restore `+` in regex.

## Lessons Learned
1. Email validation should use battle-tested library, not custom regex
2. Need tests for edge cases in validation
3. Need monitoring for login failure rate

## Action Items
- [ ] Replace custom regex with validator library
- [ ] Add monitoring alert for login failure spike
- [ ] Review all validation code for similar issues
- [ ] Add RFC compliance tests for email validation
```

## What to Document

### For Every Bug

```markdown
1. What was the symptom? (What user saw)
2. What was the root cause? (Why it happened)
3. How was it diagnosed? (Steps to find it)
4. How was it fixed? (The solution)
5. How to prevent recurrence? (Tests, monitoring, etc.)
```

### Root Cause Categories

When documenting, categorize the root cause:

| Category | Example |
|----------|---------|
| Logic error | Off-by-one in loop condition |
| Missing validation | Didn't check for null |
| Race condition | Async operations not awaited |
| Configuration | Wrong environment variable |
| Integration | API contract changed |
| Data | Unexpected data format |
| Dependencies | Library bug or breaking change |
| Infrastructure | Resource exhaustion |

## Code Comments for Subtle Bugs

```typescript
// When the fix isn't obvious, leave a comment

// BUGFIX (BUG-1234): Must check for undefined before accessing nested
// properties. User object can be partially loaded during SSR, causing
// profile to be undefined even for authenticated users.
const theme = user?.profile?.settings?.theme ?? 'default';

// BUGFIX (BUG-5678): Use >= not > for boundary check.
// Fence-post error was excluding the last valid item.
for (let i = 0; i <= items.length - 1; i++) {
```

## Knowledge Sharing

Beyond documentation, share learnings:

```markdown
- [ ] Post in team Slack/channel
- [ ] Add to team debugging wiki
- [ ] Discuss in retro/postmortem meeting
- [ ] Update onboarding docs if common pitfall
- [ ] Create lint rule if pattern is automatable
```
