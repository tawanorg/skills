---
name: discover-intent
description: >
  Enterprise-grade intent discovery framework. Interviews you relentlessly about
  a project idea until reaching 95% confidence about what you actually want.
  Use when starting a new project, validating an idea, or when you hear "grill me",
  "stress test this", "what am I missing", "discover intent", or before writing any plan.
license: MIT
metadata:
  author: tawanorg
  version: '3.0.0'
---

# Discover Intent

Exposes the gap between what you **think you want** and what you **actually need** — before a single line of code is written.

One question at a time. Recommended answer included. Codebase explored before asking. 95% confidence required before synthesis.

---

## Entry Points

Choose based on where you are:

### 1. "Pitch a new project idea"
Run skills in order: **01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11**
Full discovery for a greenfield idea. Expect 60-90 minutes.

### 2. "Stress-test an existing plan"
Run: **03 → 05 → 10 → then 11**
You have a plan. Find what's wrong with it before you build it.

### 3. "Quick gut check"
Run: **01 → 03 → then mini-assessment**
15-minute sanity check. Produces a simplified verdict: proceed, validate first, or stop.

---

## Adaptive Routing by Project Type

After the first exchange, detect project type and adjust:

| Project Type        | Required Skills              | Optional Skills     | Est. Time |
|---------------------|------------------------------|---------------------|-----------|
| Greenfield product  | 01–11                        | —                   | 90 min    |
| Internal tool       | 01, 03, 04, 05, 07, 08, 10, 11 | 02, 06, 09        | 60 min    |
| Feature addition    | 01, 03, 04, 05, 07, 08, 11  | 02, 06, 09, 10      | 45 min    |
| Migration/refactor  | 01, 03, 05, 08, 10, 11       | All others          | 40 min    |
| API/platform        | 01, 02, 03, 05, 06, 07, 08, 10, 11 | 04, 09       | 75 min    |
| Quick gut check     | 01, 03                       | —                   | 15 min    |

---

## Confidence Dashboard

Maintain and display this after each sub-skill completes:

```
Sub-skill                    Confidence  Evidence    Status
────────────────────────────────────────────────────────────
01. Motivation Audit            ??%      —           Not started
02. Problem Validation          ??%      —           Not started
03. Assumption Mining           ??%      —           Not started
04. Jobs-to-be-Done             ??%      —           Not started
05. Scope Surgery               ??%      —           Not started
06. Business Model              ??%      —           Not started
07. User Deep Dive              ??%      —           Not started
08. Technical Reality           ??%      —           Not started
09. Market Positioning          ??%      —           Not started
10. Failure Modes               ??%      —           Not started
11. Synthesis                   ??%      —           Not started
────────────────────────────────────────────────────────────
Overall Confidence:             ??%      Below threshold (95%)
```

Do not proceed to synthesis until overall confidence reaches **95%** or all unresolved items are explicitly flagged as known unknowns with a rationale.

---

## Interview Protocol

For every question across all sub-skills:

1. **Ask** — one question at a time, never multiple
2. **Recommend** — provide your best-guess answer based on evidence gathered so far
3. **Score** — assign confidence 0-100% to the answer given
4. **Grade** — assign evidence grade A/B/C/D/F (see `frameworks/evidence-grading.md`)
5. **Detect** — flag contradictions with prior answers (see `frameworks/contradiction-detection.md`)
6. **Probe** — if confidence < 70%, ask follow-up questions from the sub-skill's probe list
7. **Explore** — if the answer can be found in the codebase or attached data, do that instead of asking

---

## Sub-skill Index

| # | Sub-skill | Goal | Artifact |
|---|-----------|------|----------|
| 01 | [Motivation Audit](skills/01-motivation-audit.md) | Why this, why you, why now | Motivation Audit Card |
| 02 | [Problem Validation](skills/02-problem-validation.md) | Does the problem exist at scale | Problem Validation Scorecard |
| 03 | [Assumption Mining](skills/03-assumption-mining.md) | What beliefs are treated as facts | Assumption Register |
| 04 | [Jobs-to-be-Done](skills/04-jobs-to-be-done.md) | What job is the user hiring for | JTBD Canvas |
| 05 | [Scope Surgery](skills/05-scope-surgery.md) | Cut to the essential core | Scope Definition |
| 06 | [Business Model](skills/06-business-model.md) | Unit economics and viability | Unit Economics Card |
| 07 | [User Deep Dive](skills/07-user-deep-dive.md) | Evidence-based personas | Persona Card(s) |
| 08 | [Technical Reality](skills/08-technical-reality.md) | Architecture and constraints | Technical Decision Log |
| 09 | [Market Positioning](skills/09-market-positioning.md) | Competitive truth | Positioning Map |
| 10 | [Failure Modes](skills/10-failure-modes.md) | Second-order effects and risks | Failure Mode Table |
| 11 | [Synthesis](skills/11-synthesis.md) | Crystallized spec and verdict | 8-artifact final output |

---

## Framework References

- **Scoring rules:** `frameworks/confidence-scoring.md`
- **Evidence grading:** `frameworks/evidence-grading.md`
- **Contradiction detection:** `frameworks/contradiction-detection.md`
- **Routing logic:** `frameworks/adaptive-routing.md`

## Template References

- `templates/assumption-register.md`
- `templates/risk-matrix.md`
- `templates/decision-log.md`
- `templates/jtbd-canvas.md`
- `templates/crystallized-spec.md`
