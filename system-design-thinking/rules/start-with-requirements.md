---
title: Start with Requirements
impact: CRITICAL
impactDescription: prevents building the wrong thing
tags: foundation, requirements, discovery
---

## Start with Requirements

Never design without understanding what you're designing for. Gather functional and non-functional requirements first.

**Context:** You're asked to design a system or choose an architecture.

**Heuristic:** Before drawing boxes and arrows, answer these questions:

| Category | Questions |
|----------|-----------|
| **Users** | Who uses this? How many? What do they need? |
| **Data** | What data? How much? How fast does it grow? |
| **Performance** | Latency requirements? Throughput? |
| **Availability** | What uptime? What's the cost of downtime? |
| **Consistency** | Strong or eventual? What are the consequences of stale data? |
| **Security** | What's sensitive? Compliance requirements? |
| **Constraints** | Budget? Timeline? Team size? Existing systems? |

**Apply when:**
- Starting a new system design
- Evaluating architecture options
- Someone asks "how should we build this?"

**Skip when:**
- Requirements are already documented and understood
- Making incremental changes to existing systems

**Why:** Architecture decisions are expensive to change. Wrong assumptions compound into wrong designs. 10 minutes of questions saves 10 weeks of rework.

**Example:**

```
❌ "Let's use microservices for the new payment system"

✅ "Before choosing an architecture, let me understand:
   - Expected transaction volume? (10K/day vs 10M/day changes everything)
   - Latency requirements? (Real-time vs batch affects design)
   - Team size? (3 devs can't maintain 20 services)
   - Compliance? (PCI-DSS has specific requirements)
   - Integration with existing systems?"
```
