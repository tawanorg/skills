---
title: Define Boundaries First
impact: HIGH
impactDescription: enables independent change and team autonomy
tags: decomposition, boundaries, ddd, coupling
---

## Define Boundaries First

Before designing internals, identify the boundaries between components. Good boundaries are the foundation of maintainable systems.

**Context:** Decomposing a system into components, services, or modules.

**Heuristic:** Find boundaries where:
- Changes are contained (high cohesion within)
- Dependencies are minimal (low coupling across)
- Contracts are stable (interfaces rarely change)

**Signs of good boundaries:**

| Good Boundary | Bad Boundary |
|---------------|--------------|
| Changes stay within boundary | Changes ripple across boundaries |
| Clear owner (one team) | Shared ownership confusion |
| Stable interface | Interface constantly changing |
| Can deploy independently | Must deploy together |
| Own data store | Shared database |

**Boundary-finding techniques:**

1. **By domain** - Business capabilities (Orders, Payments, Users)
2. **By volatility** - Separate stable from changing parts
3. **By team** - Align with organizational structure
4. **By data** - Who owns what data?

**Apply when:**
- Designing a new system
- Breaking up a monolith
- Defining service boundaries
- Creating module structure

**Skip when:**
- Building a small, single-purpose tool
- Prototyping (boundaries emerge from learning)

**Why:** Wrong boundaries create distributed monoliths—all the complexity of microservices with none of the benefits. Changing boundaries later is expensive.

**Example:**

```
❌ Splitting by technical layer:
   - UserController service
   - UserService service
   - UserRepository service
   (All must change together for any user feature)

✅ Splitting by domain:
   - User service (all user functionality)
   - Order service (all order functionality)
   - Payment service (all payment functionality)
   (Each can evolve independently)
```

**Questions to validate boundaries:**

- Can this component be developed by one team?
- Can it be deployed without coordinating with others?
- Does it have a clear, stable API?
- Does it own its data?
