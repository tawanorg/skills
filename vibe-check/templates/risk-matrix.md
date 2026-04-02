# Risk Matrix Template

Populated during skill 10 (Failure Modes) and updated during synthesis. Captures all risks — not just technical — across the full project lifecycle.

---

## Risk Register

Add every risk identified across all sub-skills:

```
RISK REGISTER
─────────────────────────────────────────────────────────────────────────────
#  | Risk Description              | Category   | Prob | Impact   | Detection | Mitigation          | Owner | Status
---|-------------------------------|------------|------|----------|-----------|---------------------|-------|--------
1  |                               | Technical  | High | Critical | Slow      |                     |       | Open
2  |                               | Market     | Med  | High     | Fast      |                     |       | Open
3  |                               | Team       | Low  | High     | Slow      |                     |       | Open
4  |                               | Compliance | High | Critical | Never     |                     |       | BLOCK
5  |                               |            |      |          |           |                     |       |
─────────────────────────────────────────────────────────────────────────────

Categories: Technical / Market / Team / Financial / Compliance / Operational / External
Probability: High / Medium / Low
Impact: Critical / High / Medium / Low
Detection Speed: Fast (days) / Slow (weeks-months) / Never (found by users or regulators)
```

---

## Risk Matrix (2x2)

Place risk numbers from the register into the appropriate quadrant:

```
RISK MATRIX
═══════════════════════════════════════════════════════════════
                     Low Impact          High / Critical Impact
                  ┌─────────────────┬─────────────────────────┐
                  │                 │                         │
 High Probability │   MITIGATE      │   BLOCK                 │
                  │   Reduce        │   Must resolve before   │
                  │   probability   │   BUILD verdict         │
                  │                 │                         │
                  │   #___  #___    │   #___  #___  #___      │
                  │                 │                         │
                  ├─────────────────┼─────────────────────────┤
                  │                 │                         │
 Low Probability  │   ACCEPT        │   CONTINGENCY           │
                  │   Document &    │   Plan B required        │
                  │   monitor       │   before BUILD          │
                  │                 │                         │
                  │   #___  #___    │   #___  #___            │
                  │                 │                         │
                  └─────────────────┴─────────────────────────┘
═══════════════════════════════════════════════════════════════
```

---

## Quadrant Action Plans

### BLOCK Risks (High Probability + High/Critical Impact)

These must be resolved before a BUILD verdict is possible:

```
BLOCK RISK ACTION PLAN
─────────────────────────────────────────────────────────
Risk #___: ___
  Resolution approach: ___
  Owner:               ___
  Deadline:            ___
  Success criteria:    ___
  Current status:      Open / In Progress / Resolved

Risk #___: ___
  Resolution approach: ___
  Owner:               ___
  Deadline:            ___
  Success criteria:    ___
  Current status:      Open / In Progress / Resolved
─────────────────────────────────────────────────────────
```

### CONTINGENCY Risks (Low Probability + High/Critical Impact)

Plan B must exist before building starts:

```
CONTINGENCY PLANS
─────────────────────────────────────────────────────────
Risk #___: ___
  Trigger condition:   ___  (when to activate plan B)
  Plan B:              ___
  Lead time needed:    ___  (how early the trigger must be detected)
  Early warning:       ___  (what to monitor)

Risk #___: ___
  Trigger condition:   ___
  Plan B:              ___
  Lead time needed:    ___
  Early warning:       ___
─────────────────────────────────────────────────────────
```

### MITIGATE Risks (High Probability + Low Impact)

Reduce probability through design or process:

```
MITIGATION ACTIONS
─────────────────────────────────────────────────────────
Risk #___: ___
  Mitigation:    ___
  Owner:         ___
  Timeline:      ___
─────────────────────────────────────────────────────────
```

---

## Matrix Status Summary

```
RISK SUMMARY
─────────────────────────────────────────────────────────
Total risks identified:    ___

By quadrant:
  BLOCK:        ___  (must reach 0 resolved for BUILD verdict)
  Contingency:  ___  (must all have plan B)
  Mitigate:     ___
  Accept:       ___

By detection speed:
  Never-detected risks: ___  ← Special attention required

BLOCK risks resolved:   ___ / ___
Ready for BUILD:        Yes / No / Conditional
─────────────────────────────────────────────────────────
```
