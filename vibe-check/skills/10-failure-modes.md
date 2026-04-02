# 10 — Failure Modes

**Goal:** Think past launch day. Map what happens when this works at scale — and when it breaks, quietly or catastrophically.

**When to use independently:** Before committing to build, when a project has been approved and the team feels confident — that feeling is when failure mode analysis matters most.

---

## Question Matrix

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 1 | If this works perfectly, what new problem does success create? Name the specific second-order effect that nobody is planning for. | Unplanned scaling burden, support load, expectation inflation | Stated |
| 2 | At 10x your projected users, what breaks first? At 100x? Walk through the system — not just infrastructure, but team, process, and support. | Scaling assumption gaps beyond the architecture diagram | Technical |
| 3 | What is the hardest thing to change after you have shipped and have real users? Be specific about what locks in. | Irreversible design decisions that will haunt the product | Technical |
| 4 | What decision are you making permanent right now that should remain reversible? | Over-commitment to approach before evidence warrants it | Stated |
| 5 | If this works but you hate running it in two years, what do you do? Have you thought through that scenario? | Sustainability and founder-operator fit | Stated |
| 6 | What is the worst realistic thing that happens if you ship a significant bug? Data loss? Financial loss? Safety risk? Regulatory consequence? | Failure severity calibration and recovery cost | Stated |
| 7 | What happens if the key technical person leaves mid-project? What knowledge exists only in their head right now? | Bus factor and knowledge concentration risk | Stated |
| 8 | What regulatory or compliance requirements apply to this product? Have you confirmed that with someone who knows the regulation, not just your interpretation of it? | Legal and compliance exposure hiding in technical decisions | Stated |
| 9 | What is your support model when something breaks for a paying user at 2 AM? Who handles it, with what tools, on what SLA? | Operational readiness and support cost model | Stated |
| 10 | What is the realistic timeline — not the optimistic one? Now add 50%. Is it still worth doing at that timeline? | Timeline honesty and opportunity cost | Stated |
| 11 | What organizational dependencies could block this at 80% complete? Other teams, budget cycles, legal review, third-party contracts? | Execution blockers that appear late when cost of stopping is highest | Stated |
| 12 | What does failure look like at 6 months? At 12 months? What signal will you see first, and how will you know it is time to stop versus time to adjust? | Early warning system and kill criteria | Stated |

---

## Failure Mode Analysis Table

Build this during the phase. Any row with High probability + High/Critical impact + Slow/Never detection is a project blocker:

| Failure Scenario | Probability | Impact | Detection Speed | Mitigation | Owner |
|-----------------|-------------|--------|-----------------|------------|-------|
| _fill_ | Low/Med/High | Low/Med/High/Critical | Fast/Slow/Never | _fill_ | _fill_ |

---

## Organizational Readiness Assessment

```
ORGANIZATIONAL READINESS CHECK
──────────────────────────────────────────────────────
Team Readiness
  [ ] Skills to build: confirmed / gaps identified
  [ ] Bus factor > 1 for all critical components
  [ ] Key knowledge documented, not in one person's head
  [ ] Oncall rotation defined for post-launch

Operational Readiness
  [ ] Support model defined (who, SLA, tools)
  [ ] Incident response process exists
  [ ] Monitoring and alerting planned
  [ ] Rollback procedure defined and tested

Compliance Readiness
  [ ] Regulatory requirements confirmed by qualified person
  [ ] Data privacy requirements mapped
  [ ] Security review planned or completed
  [ ] Insurance / liability coverage reviewed

Organizational Readiness
  [ ] All blocking dependencies identified
  [ ] Budget approved through realistic timeline
  [ ] Legal review timeline accounted for
  [ ] Kill criteria agreed upon by decision-makers
──────────────────────────────────────────────────────
```

---

## Follow-up Probes (use when confidence < 70%)

- You said support will be handled by [person/team]. What is their capacity right now? Have they agreed to this?
- The failure scenario you rated as low probability — what would it take for it to become high probability? Is that scenario plausible in the next 12 months?
- You described [X] as the worst-case bug. What does "fix and recover" look like for that scenario in practice?
- The regulatory requirement you mentioned — have you talked to legal counsel, or is that your interpretation of the regulation?
- At 10x users, you said [component] breaks. What is the lead time to fix that? How much runway do you need before it becomes an emergency?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| Success blindness | Plans for failure scenarios but not for success scenarios that create new problems |
| Complexity undercount | Says "it's just a bug fix" for scenarios that would require data migration or rollback |
| Compliance confidence | Describes regulatory requirements with certainty but has not consulted legal |
| Single-threaded operation | Entire project depends on one person's availability through completion |
| Kill criteria avoidance | Cannot or will not define conditions under which the project should be abandoned |

---

## Output Artifact: Failure Mode Analysis

```
FAILURE MODE ANALYSIS
─────────────────────────────────────────────────────────
Project:   ___
Date:      ___

SECOND-ORDER EFFECTS OF SUCCESS
  Success creates: ___
  Planning response: ___

SCALING BREAK POINTS
  10x users — breaks first: ___  Lead time to fix: ___
  100x users — breaks first: ___  Lead time to fix: ___

IRREVERSIBLE DECISIONS (locked in at ship)
  1. ___  Cost to reverse: ___
  2. ___  Cost to reverse: ___

WORST-CASE BUG
  Scenario:  ___
  Impact:    Data / Financial / Safety / Reputational / Regulatory
  Recovery:  ___  Time: ___  Cost: ___

BUS FACTOR
  Key person: ___  Unique knowledge: ___  Mitigation: ___

COMPLIANCE CONFIRMATION
  Requirement: ___  Confirmed by: ___  Date: ___

SUPPORT MODEL
  Who:      ___
  SLA:      ___
  Hours:    24/7 / Business hours / Best effort
  Tools:    ___

REALISTIC TIMELINE
  Optimistic: ___ months
  Realistic:  ___ months  (+ 50% buffer)
  Still worth it at realistic timeline: Yes / No / Conditional

FAILURE MODE TABLE
Scenario         | Prob | Impact   | Detection | Mitigation        | Owner
-----------------|------|----------|-----------|-------------------|------
                 | High | Critical | Never     |                   |  ← BLOCK

KILL CRITERIA (agreed before building)
  Condition 1: ___  Threshold: ___  Measurement: ___
  Condition 2: ___  Threshold: ___  Measurement: ___
  Condition 3: ___  Threshold: ___  Measurement: ___

ORGANIZATIONAL READINESS:  Ready / Partially Ready / NOT READY

PHASE CONFIDENCE:  ___% (must reach 80% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```
