# 03 — Assumption Mining

**Goal:** Surface every belief you are treating as fact and rank them by how catastrophic it would be if wrong.

**When to use independently:** Any time before committing significant resources. Also run this when a plan feels "too solid" — that feeling is a warning sign.

---

## Question Matrix

| # | Question | What It Exposes | Risk Level |
|---|----------|-----------------|------------|
| 1 | List every assumption this project depends on — user behavior, technical feasibility, market conditions, team capability, regulatory environment. I will help, but you start. | Assumption awareness and depth of thinking | — |
| 2 | Which single assumption, if wrong, makes the entire project worthless regardless of how well everything else goes? | The fatal assumption — must be validated before building | Critical |
| 3 | What are you assuming about user behavior that you have never directly observed? Be specific about each one. | Untested behavioral hypotheses treated as requirements | High |
| 4 | What technical feasibility are you assuming that you have not prototyped or proven? | Engineering risk hidden as architecture slide | High |
| 5 | What are you assuming about market conditions that could change in the next six months? Name the conditions specifically. | Market timing and external dependency risk | Medium |
| 6 | Where are you designing for yourself instead of your user? Be brutally honest — which features would you cut if you were not the user? | Projection bias masquerading as user research | High |
| 7 | What complexity are you hiding from yourself right now? What is the thing you keep not writing down? | Scope denial and avoidance | High |
| 8 | What assumption about your competition requires you to be right and them to be wrong or slow? | Competitive assumption risk | Medium |
| 9 | What does this project assume about team size, skill, and availability that has not been confirmed? | Resource assumption gap | High |
| 10 | What regulatory, legal, or compliance assumption are you making? What happens if you are wrong? | Legal and compliance exposure | Critical |
| 11 | What assumption are you most reluctant to test? Why? | The assumption you know is weakest but are protecting | Critical |
| 12 | If you were a competitor trying to kill this idea, which assumption would you attack first? | External threat vector through your weakest assumption | High |

---

## Assumption Risk Matrix

Place every assumption from Question 1 into one quadrant. High-probability-wrong + high-impact assumptions are project blockers:

```
                         Low Impact if Wrong    High Impact if Wrong
                        ┌──────────────────────┬──────────────────────┐
 High probability wrong │ MONITOR              │ VALIDATE IMMEDIATELY │
                        │ (track quarterly)    │ (block on this)      │
                        ├──────────────────────┼──────────────────────┤
 Low probability wrong  │ ACCEPT               │ CONTINGENCY REQUIRED │
                        │ (document only)      │ (have a plan B)      │
                        └──────────────────────┴──────────────────────┘
```

Any assumption in the top-right quadrant must be validated before committing to build. If it cannot be validated cheaply, that itself is a project risk.

---

## Follow-up Probes (use when confidence < 70%)

- You listed [N] assumptions. Most projects at this stage have 20-30. What are you not listing?
- You said this assumption is "unlikely to be wrong" — what would it take to prove it wrong? Have you tried?
- The assumption you marked as "low impact if wrong" — walk me through the downstream effects if it is wrong. Are you sure the impact is low?
- What assumption did you not write down because writing it down felt dangerous?
- Your fatal assumption is [X] — what is the cheapest experiment that would tell you if it is wrong?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| Assumption laundering | Calls an untested belief a "known fact" or "established baseline" |
| Impact minimization | Consistently rates high-risk assumptions as low impact |
| Circular validation | Uses one assumption to validate another assumption |
| Missing technical assumption | Plan includes new technology with no feasibility proof |
| Regulatory blind spot | No compliance assumptions listed despite handling user data or money |

---

## Output Artifact: Assumption Register

```
ASSUMPTION REGISTER
─────────────────────────────────────────────────────────
Project:   ___
Date:      ___
Version:   ___

FATAL ASSUMPTION (the one that kills everything if wrong)
______________________________________________________
Validation plan: ___
Validation cost: ___
Timeline:        ___

FULL ASSUMPTION LIST

#  | Assumption                        | Grade | Risk if Wrong | Quadrant     | Status
---|-----------------------------------|-------|---------------|--------------|--------
1  |                                   | D     | Critical      | BLOCK        | Unvalidated
2  |                                   | B     | High          | Contingency  | Unvalidated
3  |                                   | A     | Medium        | Accept       | Validated
…  |                                   |       |               |              |

RISK MATRIX SUMMARY
  VALIDATE IMMEDIATELY (top-right):  ___ assumptions
  CONTINGENCY REQUIRED (bot-right):  ___ assumptions
  MONITOR (top-left):                ___ assumptions
  ACCEPT (bot-left):                 ___ assumptions

VALIDATION QUEUE (ordered by priority)
  1. [Assumption] — Method: ___ — Cost: ___ — Owner: ___
  2. [Assumption] — Method: ___ — Cost: ___ — Owner: ___

PHASE CONFIDENCE:  ___% (must reach 80% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```
