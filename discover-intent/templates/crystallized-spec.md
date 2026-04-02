# Crystallized Spec Template

The final output of the discover-intent process. Produced by skill 11 (Synthesis) after all required sub-skills are complete and overall confidence reaches 95%.

This document is the single source of truth for what the project is, who it is for, what is in scope, and whether to build it.

---

## Spec Header

```
CRYSTALLIZED SPEC
═════════════════════════════════════════════════════════
Project:            ___
Date:               ___
Discovery conducted by: ___
Overall confidence: ___%  (must be 95%+ for BUILD verdict)
Evidence grade:     A / B / C / D / F  (overall)
Sub-skills run:     ___  (list which ones)
Duration:           ___ minutes
═════════════════════════════════════════════════════════
```

---

## Artifact 1: True Intent Statement

```
TRUE INTENT STATEMENT
─────────────────────────────────────────────────────────
[One paragraph. What this project actually is, stripped of aspiration,
marketing language, and wishful thinking. Written as if explaining to
a skeptical investor with 30 seconds of patience.]




The moment stated intent diverged from revealed intent:
[Describe the specific exchange or question that exposed the real driver]


─────────────────────────────────────────────────────────
```

---

## Artifact 2: JTBD Summary

```
JTBD SUMMARY
─────────────────────────────────────────────────────────
Primary Job:
  When     ___
  I want   ___
  So I can ___
  Without  ___

Emotional Job: I want to feel ___ when ___.
Social Job:    I want to be seen as ___ by ___.

Key Anxieties:
  1. ___
  2. ___
  3. ___

Key Trade-offs:
  1. ___
  2. ___

Evidence grade: A / B / C / D / F
─────────────────────────────────────────────────────────
```

---

## Artifact 3: Validated Assumption Register

```
ASSUMPTION REGISTER (summary)
─────────────────────────────────────────────────────────
Fatal assumption:      ___
  Status:              Validated / UNVALIDATED

Total assumptions:     ___
  Validated:           ___
  Unvalidated:         ___  ← Must be 0 for BUILD verdict
  Contradicted:        ___  ← Must be 0 for any verdict except WALK AWAY

BLOCK quadrant items:  ___  (all must be validated for BUILD)

Full register: see templates/assumption-register.md
─────────────────────────────────────────────────────────
```

---

## Artifact 4: Risk Matrix

```
RISK MATRIX (summary)
─────────────────────────────────────────────────────────
                     Low Impact          High / Critical Impact
                  ┌─────────────────┬─────────────────────────┐
 High Probability │   MITIGATE      │   BLOCK                 │
                  │   ___  ___      │   ___  ___  ___         │
                  ├─────────────────┼─────────────────────────┤
 Low Probability  │   ACCEPT        │   CONTINGENCY           │
                  │   ___  ___      │   ___  ___              │
                  └─────────────────┴─────────────────────────┘

BLOCK items (must resolve before BUILD):
  1. ___
  2. ___

All BLOCK items resolved: Yes / No
All CONTINGENCY items have Plan B: Yes / No / Partial

Full matrix: see templates/risk-matrix.md
─────────────────────────────────────────────────────────
```

---

## Artifact 5: Scope Definition

```
SCOPE DEFINITION
─────────────────────────────────────────────────────────
ATOMIC CORE
  User action:      ___
  System response:  ___
  Job it performs:  ___

IN SCOPE (with evidence)
  Feature                    | JTBD Mapping | Evidence | Priority
  ---------------------------|--------------|----------|----------
                             |              |          | P0
                             |              |          | P0
                             |              |          | P1
                             |              |          | P2

EXPLICITLY OUT OF SCOPE
  ___  Reason: ___
  ___  Reason: ___

DEFERRED
  ___  Revisit when: ___
  ___  Revisit when: ___
─────────────────────────────────────────────────────────
```

---

## Artifact 6: Decision Log

```
DECISION LOG (summary)
─────────────────────────────────────────────────────────
Key decisions:

#  | Decision           | Choice  | Reversibility | Confidence
---|--------------------|---------|-  ------------|------------
   |                    |         | Easy          | High
   |                    |         | Hard          | Med
   |                    |         | Permanent     | Low ← FLAG

HIGH-RISK DECISIONS (Permanent + Low Confidence):
  ___  Resolution plan: ___

Full log: see templates/decision-log.md
─────────────────────────────────────────────────────────
```

---

## Artifact 7: Kill Criteria

```
KILL CRITERIA
─────────────────────────────────────────────────────────
Agreement from: ___  Date: ___

These conditions trigger project abandonment, not pivot.

Condition 1: ___
  Threshold:    ___
  Measurement:  ___
  Deadline:     ___

Condition 2: ___
  Threshold:    ___
  Measurement:  ___
  Deadline:     ___

Condition 3: ___
  Threshold:    ___
  Measurement:  ___
  Deadline:     ___

Kill criteria agreed before building: Yes / No
─────────────────────────────────────────────────────────
```

---

## Artifact 8: Honest Verdict

```
HONEST VERDICT
═════════════════════════════════════════════════════════
VERDICT:  BUILD / VALIDATE FIRST / PIVOT / WALK AWAY
═════════════════════════════════════════════════════════

─── If BUILD ────────────────────────────────────────────
Evidence supports this. Risks are manageable.

Proceed with these specific caveats:
  1. ___
  2. ___

Watch closely for:
  ___

First 30 days: ___  (what to measure to confirm the direction)

─── If VALIDATE FIRST ───────────────────────────────────
The problem is real. The approach may not be.

Validate these before committing resources:
  1. ___  Method: ___  Cost: ___  Timeline: ___
  2. ___  Method: ___  Cost: ___  Timeline: ___

Return to full discovery after results are in.

─── If PIVOT ────────────────────────────────────────────
The problem is real. The current approach is wrong.

What changes:    ___
What stays:      ___
Suggested direction: ___

─── If WALK AWAY ────────────────────────────────────────
[Specific reasons] make this unviable at this time.

The real insight from this process:  ___
A better use of this energy:         ___

═════════════════════════════════════════════════════════
OVERALL CONFIDENCE:   ___%
EVIDENCE GRADE:       A / B / C / D / F
DISCOVERY DATE:       ___
NEXT REVIEW DATE:     ___  (confidence expires — revalidate after this date)
═════════════════════════════════════════════════════════
```
