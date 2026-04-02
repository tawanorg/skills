# 11 — Synthesis

**Goal:** Combine all prior artifacts into a crystallized spec and deliver an honest, direct verdict.

**When to use independently:** After completing at least skills 01, 03, and 05. Full synthesis requires all relevant skills for the project type to be complete.

---

## Prerequisites

Before running synthesis, confirm:

```
SYNTHESIS READINESS CHECK
──────────────────────────────────────────────────────────
Sub-skill                    Completed   Confidence   Gate
─────────────────────────────────────────────────────────
01. Motivation Audit         [ ]         ___%         80%
02. Problem Validation       [ ]         ___%         80%
03. Assumption Mining        [ ]         ___%         80%
04. Jobs-to-be-Done          [ ]         ___%         80%
05. Scope Surgery            [ ]         ___%         80%
06. Business Model           [ ]         ___%         75%
07. User Deep Dive           [ ]         ___%         80%
08. Technical Reality        [ ]         ___%         80%
09. Market Positioning       [ ]         ___%         75%
10. Failure Modes            [ ]         ___%         80%
─────────────────────────────────────────────────────────
Overall Confidence:          ___%        Required: 95%

Unresolved contradictions:   ___  (must be 0 to proceed)
D-grade critical decisions:  ___  (must be 0 to proceed)
F-grade evidence items:      ___  (must be 0 to proceed)
──────────────────────────────────────────────────────────
```

If overall confidence is below 95%, return to the lowest-confidence sub-skill and continue probing before synthesizing.

---

## Synthesis Process

1. Pull the output artifact from each completed sub-skill
2. Identify all contradictions across artifacts — resolve before writing
3. Identify the moment in the interview where stated intent and revealed intent diverged
4. Write each artifact below in order — each one informs the next
5. Deliver the verdict last, not first

---

## The 8 Synthesis Artifacts

### Artifact 1: True Intent Statement

One paragraph. What this project actually is, stripped of aspiration, marketing language, and wishful thinking. Written as if explaining to a skeptical investor with 30 seconds of patience who will ask "so what?" at the end of every sentence.

```
TRUE INTENT STATEMENT
─────────────────────────────────────────────────────────
[One paragraph — specific, honest, stripped of aspiration]

The moment stated intent diverged from revealed intent:
[Describe the specific exchange or answer that showed the real driver]
─────────────────────────────────────────────────────────
```

---

### Artifact 2: JTBD Summary

```
JTBD SUMMARY
─────────────────────────────────────────────────────────
Primary Job:
  When [situation],
  I want to [functional job],
  so I can [emotional / social outcome],
  without [primary anxiety].

Emotional Job:   I want to feel [emotion] when [activity].
Social Job:      I want to be seen as [perception] by [audience].

Key Anxieties:
  1. ___
  2. ___
  3. ___

Key Trade-offs (what user gives up):
  1. ___
  2. ___

Evidence grade: A / B / C / D / F
─────────────────────────────────────────────────────────
```

---

### Artifact 3: Validated Assumption Register

See `templates/assumption-register.md` for full template.

```
VALIDATED ASSUMPTION REGISTER (summary)
─────────────────────────────────────────────────────────
Fatal assumption:  ___  Status: Validated / UNVALIDATED
Validated:         ___ assumptions
Unvalidated:       ___ assumptions  ← Must be zero for BUILD verdict
Contradicted:      ___ assumptions  ← Must be resolved before any verdict
─────────────────────────────────────────────────────────
```

---

### Artifact 4: Risk Matrix

See `templates/risk-matrix.md` for full template.

```
RISK MATRIX
─────────────────────────────────────────────────────────
                     Low Impact          High Impact
                  ┌─────────────────┬─────────────────┐
 High Probability │ MITIGATE        │ BLOCK — resolve  │
                  │ ___  ___  ___   │ before building  │
                  │                 │ ___  ___  ___    │
                  ├─────────────────┼─────────────────┤
 Low Probability  │ ACCEPT          │ CONTINGENCY plan │
                  │ ___  ___  ___   │ required         │
                  │                 │ ___  ___  ___    │
                  └─────────────────┴─────────────────┘

BLOCK items (must resolve before BUILD verdict):
  1. ___
  2. ___
─────────────────────────────────────────────────────────
```

---

### Artifact 5: Scope Definition

```
SCOPE DEFINITION (final)
─────────────────────────────────────────────────────────
ATOMIC CORE:  ___  (the one interaction that proves the idea)

IN SCOPE (with evidence grade)
  Feature                | JTBD      | Grade | Priority
  -----------------------|-----------|-------|----------
                         |           |       | P0
                         |           |       | P0
                         |           |       | P1

OUT OF SCOPE (explicit, with reason)
  ___  Reason: ___
  ___  Reason: ___

DEFERRED
  ___  Revisit when: ___
  ___  Revisit when: ___
─────────────────────────────────────────────────────────
```

---

### Artifact 6: Decision Log

See `templates/decision-log.md` for full template.

```
KEY DECISIONS (summary)
─────────────────────────────────────────────────────────
Decision          | Choice  | Reversibility | Confidence
------------------|---------|---------------|------------
                  |         | Permanent     | Low ← FLAG
                  |         | Hard          | Med
                  |         | Easy          | High

Permanent + Low Confidence decisions (require resolution):
  ___
─────────────────────────────────────────────────────────
```

---

### Artifact 7: Kill Criteria

Specific, measurable conditions under which the project should be abandoned — not pivoted, abandoned. Agreed upon before building starts:

```
KILL CRITERIA
─────────────────────────────────────────────────────────
These conditions trigger project abandonment, not pivot.
Agreement required from: ___  Date agreed: ___

Condition 1: ___
  Threshold:    ___  (specific measurable number)
  Measurement:  ___  (how and when)
  Deadline:     ___  (by when)

Condition 2: ___
  Threshold:    ___
  Measurement:  ___
  Deadline:     ___

Condition 3: ___
  Threshold:    ___
  Measurement:  ___
  Deadline:     ___

Note: Kill criteria that cannot be agreed upon before building
indicate the team is not aligned on what success requires.
─────────────────────────────────────────────────────────
```

---

### Artifact 8: Honest Verdict

Delivered last. Direct and compassionate. Not hedged:

```
HONEST VERDICT
─────────────────────────────────────────────────────────
VERDICT:  BUILD / VALIDATE FIRST / PIVOT / WALK AWAY

─── BUILD ───────────────────────────────────────────────
Evidence supports proceeding. Risks are manageable.

Proceed with these specific caveats:
  1. ___
  2. ___

Watch closely for:
  ___

─── VALIDATE FIRST ──────────────────────────────────────
The problem is real. The approach may not be.

Validate these specific assumptions before committing:
  1. ___  Method: ___  Cost: ___  Timeline: ___
  2. ___  Method: ___  Cost: ___  Timeline: ___

Return to full discovery after validation results are in.

─── PIVOT ───────────────────────────────────────────────
The problem is real. The current approach is wrong.

What needs to change:
  ___

What stays the same:
  ___

Suggested direction:
  ___

─── WALK AWAY ───────────────────────────────────────────
[Specific reasons] make this unviable at this time.
That is a good outcome. Clarity is valuable.

The real insight from this process:
  ___

A better use of this energy might be:
  ___

─────────────────────────────────────────────────────────
OVERALL CONFIDENCE AT VERDICT:  ___%
EVIDENCE GRADE ACROSS FINDINGS: A / B / C / D / F
DATE:  ___
─────────────────────────────────────────────────────────
```
