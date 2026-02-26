---
title: Check Recent Changes
impact: high
tags: [debugging, git, methodology]
---

# Check Recent Changes

**If something stopped working, something changed. Find what.**

## The Rule

When a bug appears:
1. Assume something changed
2. Check what changed recently
3. Focus investigation on changes

## Why This Matters

```
Software doesn't spontaneously break.

If it worked before and doesn't now:
- Code changed
- Data changed
- Environment changed
- Dependencies changed
- Configuration changed

Find the change, find the bug.
```

## The Change Categories

```
┌─────────────────────────────────────────────────────────────┐
│                    WHAT COULD HAVE CHANGED?                  │
├─────────────────────────────────────────────────────────────┤
│ CODE          │ Your commits, merged PRs, dependencies      │
│ DATA          │ Database content, user input, API responses │
│ ENVIRONMENT   │ Server config, env vars, certificates       │
│ INFRASTRUCTURE│ Memory, CPU, network, disk                  │
│ EXTERNAL      │ Third-party APIs, services, CDNs            │
│ TIME          │ Expired tokens, scheduled jobs, timezones   │
└─────────────────────────────────────────────────────────────┘
```

## Incorrect Approach

```typescript
// Bug report: "Checkout broken since this morning"

// Developer starts debugging current code
// Spends 2 hours in the codebase
// Can't find anything wrong

// Never checked: Deployment at 8am introduced bug
```

## Correct Approach

```bash
# Bug report: "Checkout broken since this morning"

# Step 1: What time did it break?
# "Around 9am"

# Step 2: What changed around then?
git log --since="2024-01-15 07:00" --until="2024-01-15 10:00" --oneline

# Output:
# abc123 9:15am - Refactor checkout validation
# def456 8:30am - Update pricing calculation  ← Likely culprit
# ghi789 8:00am - Add logging

# Step 3: Check the suspect commit
git show def456

# Step 4: Compare behavior before/after
git checkout def456^   # Before the commit
npm test              # Works

git checkout def456    # The commit
npm test              # Fails

# Found: def456 introduced the bug
```

## What to Check

### Recent Code Changes

```bash
# Last 24 hours of commits
git log --since="24 hours ago" --oneline

# Changes to specific file
git log --oneline -10 -- path/to/file.ts

# Diff between working and broken
git diff last-known-good..HEAD

# Who changed what
git blame path/to/broken/file.ts
```

### Recent Deployments

```bash
# Check deployment history
kubectl rollout history deployment/myapp

# Check when pods were created
kubectl get pods -o wide --sort-by=.metadata.creationTimestamp

# Check recent releases
gh release list --limit 5
```

### Dependency Changes

```bash
# Check for dependency updates
git diff HEAD~10 package-lock.json
git diff HEAD~10 yarn.lock

# Check when dependencies were last updated
npm outdated
```

### Environment Changes

```bash
# Compare current vs previous environment
diff .env .env.backup

# Check for config changes
git log --oneline -- config/
git log --oneline -- *.env*
```

### External Changes

```markdown
Questions to ask:
- Did a third-party API change?
- Did SSL certificates expire?
- Did DNS records change?
- Did cloud provider have issues?
- Did rate limits kick in?
```

## Timeline Correlation

```
Create a timeline:

8:00 AM - Deploy v1.2.3
8:30 AM - First error in logs
9:00 AM - User reports issue
9:15 AM - More errors spike

Correlation: Deploy → Errors
Likely cause: Something in v1.2.3
```

## The "Nothing Changed" Fallacy

When someone says "nothing changed":

```markdown
Things that "didn't change" but actually did:
- [ ] Auto-updated dependencies
- [ ] Expired certificates/tokens
- [ ] Background migration ran
- [ ] Cron job modified data
- [ ] Cache expired
- [ ] Rate limit reset
- [ ] External API changed
- [ ] Cloud provider maintenance
- [ ] Someone else's deploy
```

Something always changed. Keep looking.
