# Clarifying Interview Questions Library

## By Request Type

### Feature Development

**Scope:**
- What's the minimum viable version of this feature?
- What's explicitly out of scope for now?
- Is this a standalone feature or part of a larger initiative?

**Users:**
- Who's the primary user of this feature?
- What's their current workflow without this feature?
- What triggers them to need this?

**Technical:**
- Are there existing patterns in the codebase to follow?
- Any specific technologies required or to avoid?
- Does this integrate with external services?

**Quality:**
- What testing approach? (unit, integration, e2e)
- Any performance requirements?
- Accessibility requirements?

### Bug Fix

**Reproduction:**
- What are the exact steps to reproduce?
- When did this start happening?
- Does it happen consistently or intermittently?

**Impact:**
- How many users are affected?
- Is there a workaround?
- What's the severity? (critical/high/medium/low)

**Context:**
- Any recent changes that might be related?
- Are there error logs or stack traces?
- What's the expected vs actual behavior?

### Refactoring

**Goals:**
- What's the primary driver? (performance, maintainability, new requirements)
- What should NOT change? (behavior, API, dependencies)
- What's the success metric?

**Scope:**
- Which files/modules are in scope?
- Are there dependent systems to consider?
- Breaking changes acceptable?

**Approach:**
- Incremental or big-bang?
- Need backward compatibility?
- Test coverage requirements?

### Integration

**External System:**
- What's the API documentation?
- Auth mechanism? (API key, OAuth, etc.)
- Rate limits or quotas?

**Data Flow:**
- What data goes in/out?
- Data transformation needed?
- How to handle sync vs async?

**Reliability:**
- What if the external system is down?
- Retry strategy?
- Fallback behavior?

## Universal Questions

### Priority & Constraints
- Timeline expectations?
- Dependencies on other work?
- Any blocking concerns?

### Existing Knowledge
- Any designs or mockups?
- Related documentation?
- Similar features to reference?

### Definition of Done
- What makes this "complete"?
- Who needs to review/approve?
- Deployment considerations?

## Question Formatting Tips

**Use multiple choice when possible:**
```
Which auth approach fits best?
A) Session-based (matches existing admin panel)
B) JWT (better for API consumers)
C) OAuth (if third-party login needed)
```

**Reference repo context:**
```
I found you're using Zustand for state management. Should this feature:
A) Use existing global store
B) Create feature-specific store
C) Use local component state (simpler)
```

**Offer recommendations:**
```
Based on similar features in your codebase, I'd recommend [approach].
Does that work, or do you have different requirements?
```
