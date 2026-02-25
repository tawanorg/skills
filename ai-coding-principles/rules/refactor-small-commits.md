---
title: Make Small, Incremental Refactoring Commits
impact: MEDIUM
impactDescription: enables safe rollback and easier review
tags: refactoring, git, safety
---

## Make Small, Incremental Refactoring Commits

Refactor in tiny steps with a commit after each change. Never do big-bang refactors.

**Incorrect:**

```bash
# One massive commit with everything
git commit -m "Refactor user module"
# Changed 50 files, renamed 20 functions, moved 10 classes
# If something breaks, which change caused it?
```

**Correct:**

```bash
# Small, focused commits - each one works
git commit -m "Extract validateEmail from UserService"
# Tests pass

git commit -m "Rename getUserById to findUserById"
# Tests pass

git commit -m "Move User class to domain/entities"
# Tests pass

git commit -m "Replace direct DB calls with UserRepository"
# Tests pass

# If anything breaks, you know exactly which commit caused it
# Easy to revert just the problematic change
```

**Why:** Small commits let you bisect failures, revert safely, and keep main deployable.

Reference: `references/refactoring-patterns.md`
