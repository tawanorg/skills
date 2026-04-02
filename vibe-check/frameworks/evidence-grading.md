# Evidence Grading Framework

## Grade Definitions

Every key finding, assumption, and claim receives an evidence grade. Grade is assigned based on the best available evidence for that specific claim:

| Grade | Label | Meaning | Acceptable Sources |
|-------|-------|---------|-------------------|
| **A** | Observed | Directly witnessed or measured. You saw it happen or a system recorded it. | Analytics, usability test recordings, A/B test results, direct observation during user session, production logs |
| **B** | Reported | User described their own behavior. You did not witness it, but they told you directly. | User interviews, support tickets with specific quotes, survey free-text, customer calls |
| **C** | Inferred | Logically derived from multiple other evidence sources. No single source states it directly. | Cross-referenced from 2+ Grade B sources, market research synthesis, triangulated from indirect signals |
| **D** | Assumed | No evidence. Believed to be true based on intuition, experience, or "common knowledge." | Founder intuition, industry folklore, analogies to other markets, "everyone knows" |
| **F** | Contradicted | Evidence exists that argues against this claim. | Data showing opposite outcome, user feedback contradicting stated belief, failed experiment |

---

## Decision Rules

**Grade A evidence:**
- Most reliable. Weight heavily in confidence scoring.
- Still requires context — observed behavior in one context may not transfer.

**Grade B evidence:**
- Reliable but subject to recall bias and social desirability bias.
- Weight higher when consistent across multiple independent sources.
- A single B-grade source is not sufficient for a critical decision.

**Grade C evidence:**
- Acceptable for supporting decisions, not sufficient for critical ones alone.
- Must be explicit about which B sources were cross-referenced.

**Grade D evidence:**
- Flag every D-grade finding explicitly.
- **No critical decision should rest on D-grade evidence.** This is a hard rule.
- D-grade findings must have a validation plan before the project proceeds.
- If a D-grade finding is in the "fatal assumption" position, the project should not advance to BUILD.

**Grade F evidence:**
- Must be resolved immediately. Do not continue the interview.
- Resolution options: (1) disprove the contradicting evidence, (2) update the claim to reflect reality, (3) escalate to a PIVOT or WALK AWAY verdict.
- F-grade findings are often more valuable than A-grade — they tell you what is actually true.

---

## Triangulation Requirement

For any finding that will drive a critical decision, require at least **two independent source types** before assigning Grade B or above:

```
Triangulation check:
  Source 1: ___  Type: Observed / Reported / Inferred
  Source 2: ___  Type: Observed / Reported / Inferred
  Agreement: Yes / Partial / No
  If No: resolve contradiction before assigning grade
```

A finding supported by three user interviews saying the same thing is still Grade B — multiple instances of the same source type do not upgrade the grade. You need a different source type.

---

## Grade Assignment Process

1. Identify the claim or finding
2. List every source of evidence for it
3. Assign the grade based on the **best single source**, not an average
4. Note any contradicting evidence (Grade F if it exists)
5. Record the source explicitly so it can be audited

```
Evidence record format:
  Claim:    ___
  Grade:    A / B / C / D / F
  Source:   ___  (specific — not "user interviews")
  Date:     ___
  Contradicting evidence: ___  (or: None)
```

---

## Grade by Phase

Minimum grade standards per phase:

| Phase | Minimum Grade for Key Findings |
|-------|-------------------------------|
| 01 Motivation Audit | C (inferred from multiple stated sources) |
| 02 Problem Validation | B (requires at least one reported + one observed) |
| 03 Assumption Mining | D acceptable to list, but validation plan required for Critical items |
| 04 Jobs-to-be-Done | B (requires interview evidence) |
| 05 Scope Surgery | B (demand claims must be reported or observed) |
| 06 Business Model | B for pricing (confirmed by user), A preferred for CAC |
| 07 User Deep Dive | B minimum (interview required for each persona) |
| 08 Technical Reality | A or B (no technical decisions on pure assumption) |
| 09 Market Positioning | B for competitor claims |
| 10 Failure Modes | C acceptable (inferred from architecture and team data) |

---

## Common Grading Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Upgrading D to C because "it makes sense" | Logical coherence does not change the grade. No evidence = Grade D. |
| Treating multiple assumed sources as triangulation | Five people guessing the same thing is still Grade D |
| Forgetting to record contradicting evidence | Always check: does evidence exist against this claim? |
| Averaging grades across a phase | Each claim is graded independently. A phase can have A and D grades simultaneously. |
| Treating "I read it in a report" as Grade A | Secondary research is Grade C at best. The original study may be Grade A. |
