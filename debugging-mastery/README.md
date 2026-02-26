# Debugging Mastery

Systematic debugging methodology used by senior engineers at top tech companies.

## Overview

This skill teaches the scientific method applied to debugging. No more guessing—instead, you'll follow a systematic approach that top engineers use to find and fix bugs efficiently.

## Core Philosophy

**Debugging is not about fixing. It's about understanding.**

Senior engineers spend 80% of their debugging time understanding the problem and 20% fixing it. Juniors invert this ratio—and end up spending more time overall.

## What's Included

### The Debugging Loop

1. **Observe** - What exactly is happening?
2. **Hypothesize** - What could cause this?
3. **Predict** - If hypothesis X is true, what else should be true?
4. **Test** - Design experiment to confirm/refute
5. **Conclude** - Was hypothesis correct?

### 10 Core Rules

| Rule | Description |
|------|-------------|
| Reproduce Before Fix | Never fix what you can't reproduce |
| Binary Search First | Halve the problem space with each step |
| Read Error Messages | The answer is often right there |
| Check Recent Changes | Something changed—find what |
| One Change at a Time | Scientific method requires single variables |
| Verify the Fix | A fix isn't done until proven to work |
| Write Regression Test | The bug should never return |
| Document Root Cause | Share knowledge with the team |
| Never Debug Blind | Get visibility before guessing |
| Question Assumptions | The bug hides in what you're certain of |

### 6 Reference Guides

| Guide | Coverage |
|-------|----------|
| Production Debugging | Debugging in production environments safely |
| Performance Debugging | Profiling, bottleneck analysis, optimization |
| Memory Leak Detection | Finding and fixing memory leaks |
| Distributed Tracing | Following requests across services |
| Common Bug Patterns | The usual suspects to check first |
| Debugging Tools | Comprehensive tool guide |

## Installation

```bash
npx skills add tawanorg/skills/debugging-mastery
```

## When This Skill Activates

- Investigating bugs
- Troubleshooting production issues
- Analyzing error logs
- Debugging performance problems
- Questions about "why isn't this working"
- Questions about "how to debug"
- Root cause analysis

## Key Concepts

### The Scientific Method

```
Bug reported → Reproduce → Isolate → Understand → Fix → Verify → Prevent
```

### Binary Search Debugging

Don't check every line (O(n)). Binary search the problem space (O(log n)).

### Production vs Local

| Local | Production |
|-------|------------|
| Full access | Limited access |
| Debugger works | Logs only |
| Can change code | Read-only mindset |
| One user | Millions of users |

## Common Bug Patterns Covered

- Null/undefined references
- Off-by-one errors
- Race conditions
- State mutation bugs
- Type coercion issues
- Async/await pitfalls
- Closure bugs
- Memory leaks

## License

MIT
