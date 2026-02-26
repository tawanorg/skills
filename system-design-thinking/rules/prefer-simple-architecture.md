---
title: Prefer Simple Architecture
impact: HIGH
impactDescription: reduces complexity and operational burden
tags: style, monolith, microservices, simplicity
---

## Prefer Simple Architecture

Start with the simplest architecture that could work. Evolve to complexity only when you have evidence it's needed.

**Context:** Choosing between architectural styles (monolith, microservices, serverless).

**Heuristic:** Default to monolith. Add complexity only when you can articulate the specific problem it solves.

```
Monolith ──► Modular Monolith ──► Microservices
    │
    └── Most projects never need to leave
```

**Complexity has costs:**

| Simple (Monolith) | Complex (Microservices) |
|-------------------|------------------------|
| One deployment | Many deployments |
| One database | Distributed data |
| Function calls | Network calls |
| Easy debugging | Distributed tracing |
| One team | Team coordination |

**When to consider microservices:**

- [ ] Team is > 8-10 engineers
- [ ] Different components need independent scaling
- [ ] Different components need different tech stacks
- [ ] You need independent deployments (regulatory, risk)
- [ ] Domain boundaries are proven and stable

**Apply when:**
- Starting a new product
- Team is small (< 8 people)
- Domain is not well understood
- Speed of iteration matters

**Skip when:**
- Proven scale requirements (millions of users)
- Large organization with multiple teams
- Mature, stable domain boundaries

**Why:** Distributed systems are hard. Network failures, eventual consistency, operational overhead. Don't pay the complexity tax until you need the benefits.

**Example:**

```
❌ "We should use microservices because that's what Netflix does"

✅ "We're a team of 4 building an MVP. Let's start with a well-structured
   monolith with clear module boundaries. We can extract services later
   if we need independent scaling or deployment."
```
