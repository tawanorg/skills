# Confidence Scoring Framework

## Per-Question Scoring

Every answer receives a confidence score from 0-100%. Score is assigned by the interviewer, not the interviewee:

| Score Range | Meaning | Action |
|-------------|---------|--------|
| 0-40% | Low — answer is vague, contradicts other evidence, or rests entirely on assumption | Probe immediately. Do not proceed to next question. |
| 41-69% | Medium — partial evidence, plausible but not verified | Apply follow-up probes from the sub-skill's probe list |
| 70-84% | Good — evidence exists, source is credible, no direct contradictions | Accept and continue. Flag for synthesis review. |
| 85-94% | High — multiple evidence sources, consistent across phases | Accept fully |
| 95-100% | Confirmed — behavioral observation or quantitative data with no contradictions | Lock and move on |

**Rule:** If confidence for a question is below 70%, you must apply at least one follow-up probe before moving to the next question. Do not skip.

---

## Per-Phase Aggregation

Phase confidence is the weighted average of all question scores within a phase, with higher weight given to questions marked Critical or High risk:

```
Phase Confidence = (Sum of weighted question scores) / (Sum of weights)

Weight by risk level:
  Critical questions:  3x weight
  High risk questions: 2x weight
  Medium:              1.5x weight
  Standard:            1x weight
```

Each sub-skill specifies its minimum gate confidence. If a phase does not reach its gate, return to the lowest-scoring questions and probe deeper.

| Sub-skill | Minimum Gate |
|-----------|-------------|
| 01 Motivation Audit | 80% |
| 02 Problem Validation | 80% |
| 03 Assumption Mining | 80% |
| 04 Jobs-to-be-Done | 80% |
| 05 Scope Surgery | 80% |
| 06 Business Model | 75% |
| 07 User Deep Dive | 80% |
| 08 Technical Reality | 80% |
| 09 Market Positioning | 75% |
| 10 Failure Modes | 80% |
| 11 Synthesis | 95% overall required |

---

## Overall Confidence Dashboard

Display after each sub-skill completes. Update in place:

```
CONFIDENCE DASHBOARD
════════════════════════════════════════════════════════════
Sub-skill                    Score    Bar              Status
────────────────────────────────────────────────────────────
01. Motivation Audit          85%    ████████░░  ✓ Gate passed
02. Problem Validation        40%    ████░░░░░░  ✗ Below gate (80%)
03. Assumption Mining         ??%    ──────────    Not started
04. Jobs-to-be-Done           ??%    ──────────    Not started
05. Scope Surgery             ??%    ──────────    Not started
06. Business Model            ??%    ──────────    Not started
07. User Deep Dive            ??%    ──────────    Not started
08. Technical Reality         ??%    ──────────    Not started
09. Market Positioning        ??%    ──────────    Not started
10. Failure Modes             ??%    ──────────    Not started
────────────────────────────────────────────────────────────
Overall Confidence:           ??%    ──────────    Below threshold
Required for synthesis:       95%
Phases below gate:            1 (must reach gate before synthesis)
════════════════════════════════════════════════════════════
```

---

## Threshold Rules

**95% overall confidence** is required to proceed to Synthesis (skill 11) with a BUILD verdict.

**Below 95% overall — options:**
1. Continue probing in lowest-confidence phases until the score rises
2. Proceed to synthesis with explicit "Known Unknowns" section listing every sub-threshold item, its confidence, and why it cannot be resolved
3. Deliver a VALIDATE FIRST verdict for any item below 80%
4. Deliver a WALK AWAY verdict if any critical question scores below 40% and cannot be improved

**Do not average your way to 95%.** One phase at 40% cannot be compensated for by other phases at 100%. Each sub-skill must reach its gate independently.

---

## Score Calibration Guide

Use these anchors when assigning scores:

| Situation | Score |
|-----------|-------|
| "I believe this is true" with no evidence | 20-30% |
| Industry common knowledge assumed true | 35-45% |
| Logical inference from adjacent evidence | 50-60% |
| Single self-reported interview | 60-70% |
| Multiple corroborating interviews | 70-80% |
| Survey data with meaningful sample size | 75-85% |
| Analytics or behavioral data | 85-95% |
| Controlled experiment with clear result | 90-100% |
| Direct observation, multiple instances | 95-100% |
