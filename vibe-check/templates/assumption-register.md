# Assumption Register Template

Populated during skill 03 (Assumption Mining) and updated throughout all subsequent skills. Every assumption that surfaces in any phase gets added here.

---

## Header

```
ASSUMPTION REGISTER
─────────────────────────────────────────────────────────
Project:      ___
Date created: ___
Last updated: ___
Version:      ___
Owner:        ___

FATAL ASSUMPTION
  Statement:       ___
  Evidence grade:  A / B / C / D / F
  Validation plan: ___
  Validation cost: ___  Timeline: ___  Owner: ___
  Status:          Validated / In Progress / UNVALIDATED
─────────────────────────────────────────────────────────
```

---

## Full Assumption List

Add every assumption, regardless of how confident you feel about it. The ones that feel most certain are often the most dangerous:

```
#  | Assumption Statement                 | Source Phase | Grade | Probability Wrong | Impact if Wrong | Quadrant     | Status        | Validation Plan
---|--------------------------------------|-------------|-------|-------------------|-----------------|--------------|---------------|----------------
1  |                                      | 01          | D     | High              | Critical        | BLOCK        | Unvalidated   |
2  |                                      | 02          | B     | Low               | High            | Contingency  | Validated     |
3  |                                      | 03          | D     | High              | Medium          | Monitor      | Unvalidated   |
4  |                                      | 04          | C     | Low               | Low             | Accept       | Validated     |
5  |                                      |             |       |                   |                 |              |               |
```

**Probability Wrong:** High / Medium / Low
**Impact if Wrong:** Critical / High / Medium / Low
**Quadrant:** BLOCK / Contingency / Monitor / Accept
**Status:** Validated / Partially Validated / Unvalidated / Contradicted

---

## Risk Quadrant Summary

```
                         Low Impact if Wrong    High Impact if Wrong
                        ┌──────────────────────┬──────────────────────┐
 High probability wrong │                      │                      │
                        │  MONITOR             │  BLOCK               │
                        │  (track quarterly)   │  (validate first)    │
                        │                      │                      │
                        │  #___  #___  #___    │  #___  #___  #___    │
                        ├──────────────────────┼──────────────────────┤
 Low probability wrong  │                      │                      │
                        │  ACCEPT              │  CONTINGENCY         │
                        │  (document only)     │  (plan B required)   │
                        │                      │                      │
                        │  #___  #___  #___    │  #___  #___  #___    │
                        └──────────────────────┴──────────────────────┘
```

---

## Status Summary

```
REGISTER STATUS
─────────────────────────────────────────────────────────
Total assumptions:         ___
  Validated:               ___
  Partially validated:     ___
  Unvalidated:             ___
  Contradicted:            ___  ← Must reach zero before BUILD verdict

By quadrant:
  BLOCK (top-right):       ___  ← Must all be validated before BUILD
  Contingency (bot-right): ___  ← Must all have contingency plans
  Monitor (top-left):      ___
  Accept (bot-left):       ___

Fatal assumption status:   Validated / UNVALIDATED ← Must be Validated
─────────────────────────────────────────────────────────
```

---

## Validation Queue

Ordered by priority — BLOCK quadrant items first:

```
VALIDATION QUEUE
─────────────────────────────────────────────────────────
Priority | # | Assumption | Method | Cost | Owner | By When
---------|---|------------|--------|------|-------|--------
1        |   |            |        |      |       |
2        |   |            |        |      |       |
3        |   |            |        |      |       |
─────────────────────────────────────────────────────────
```

**Validation Methods:**
- Customer interview (1-5 interviews, 1-2 weeks)
- Landing page test (2-4 weeks, measures real signup intent)
- Prototype / Wizard of Oz (2-4 weeks, measures real usage)
- Data analysis (existing data, 1-3 days)
- Competitive analysis (1-3 days)
- Technical spike / prototype (1-2 weeks)
- Legal / compliance review (1-4 weeks)
- Expert interview (1 week)
