# 02 — Problem Validation

**Goal:** Verify the problem exists at the scale you believe, with evidence — not intuition.

**When to use independently:** When you have a solution in mind and want to check if the problem actually justifies it.

---

## Question Matrix

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 1 | Describe the problem in one sentence without mentioning your solution. If you cannot do this, stop here. | Whether you understand the problem independent of your solution | Stated |
| 2 | What is the existing ugly workflow people use right now? Describe it step by step — every manual step, workaround, and tool. | The real incumbent: often a spreadsheet, email chain, or human process | Behavioral |
| 3 | Why is that current solution "good enough" that people have not already switched? What keeps them there? | Switching cost, inertia, and the actual bar you need to clear | Behavioral |
| 4 | How many people have this problem? Show your math — not your estimate. Where does each number come from? | Market sizing rigor and honesty about source quality | Quantitative |
| 5 | How often does this problem occur for one person? Daily, weekly, monthly, once? Show evidence, not assumption. | Frequency determines urgency and product usage patterns | Quantitative |
| 6 | What is the cost of this problem in money, time, or pain? Give specific numbers for a specific person. | Willingness-to-pay signal and severity of the problem | Quantitative |
| 7 | Quote exactly what a potential user said about this problem — their words, not your interpretation. When and where did they say it? | Confirmation bias check and interview quality | Behavioral |
| 8 | Have you observed someone experiencing this problem without you prompting them? What did you see? | The difference between reported and real behavior | Observed |
| 9 | What percentage of people who have this problem actively seek a solution? What percentage just live with it? | Urgency segmentation — not every problem gets solved | Quantitative |
| 10 | Where does this problem rank in the user's list of actual priorities? Are they losing sleep over it or just mildly annoyed? | Priority placement in the user's real life | Stated + Behavioral |
| 11 | What have users already tried and abandoned? Why did those solutions fail for them specifically? | Graveyard evidence — previous attempts reveal real constraints | Behavioral |
| 12 | What would the user have to believe is true for this problem to be worth solving with a new tool? | Worldview requirements for adoption | Inferred |

---

## Evidence Triangulation Requirements

Before leaving this phase, confirm:
- At least **one behavioral observation** (what people actually do, not what they say they do)
- At least **one quantitative data point** (usage data, market research, measured frequency)
- At least **two independent sources** for any critical finding

Flag explicitly if all evidence is self-reported stated preferences — that is not validation.

---

## Follow-up Probes (use when confidence < 70%)

- You described what users say the problem is. What have you seen them do that proves it?
- Your market size number came from [source] — walk me through how you got from that number to the addressable segment for this product.
- The cost you estimated — how did you calculate it? Did a user confirm that number or did you derive it?
- You said users are frustrated by [X] — how do you know they would pay to fix it versus just complain about it?
- Who benefits most from this problem staying unsolved? Are they in a position to resist a solution?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| Solution disguised as problem | Cannot describe problem without mentioning the product |
| Market size inflation | Numbers come from TAM reports, not bottoms-up calculation |
| Quote laundering | Paraphrases user sentiment instead of quoting directly |
| Frequency optimism | Assumes daily use for a problem that occurs monthly |
| Severity mismatch | Describes massive pain but cannot point to users spending money on alternatives |

---

## Output Artifact: Problem Validation Scorecard

```
PROBLEM VALIDATION SCORECARD
─────────────────────────────────────────────────────────
Project:   ___
Date:      ___

PROBLEM STATEMENT (solution-free)
______________________________________________________

EXISTING WORKFLOW (step by step)
  Step 1:  ___
  Step 2:  ___
  Step N:  ___
  Tools:   ___
  Pain:    ___

SCALE
  Addressable population:   ___ (source: ___)
  Frequency per person:     ___ (source: ___)
  Cost per occurrence:      ___ (source: ___)

EVIDENCE LOG
  Behavioral observation:   ___  Grade: A/B/C/D/F
  Quantitative data:        ___  Grade: A/B/C/D/F
  Direct user quotes:
    "[exact quote]" — [person, date, context]
    "[exact quote]" — [person, date, context]

URGENCY SIGNAL
  % actively seeking solution: ___%
  Priority in user's life:     High / Medium / Low / Unknown

GRAVEYARD (solutions tried and abandoned)
  ___  Failed because: ___
  ___  Failed because: ___

VALIDATION STATUS
  [ ] Problem described without solution
  [ ] Existing workflow documented step by step
  [ ] At least one behavioral observation (Grade A or B)
  [ ] At least one quantitative data point
  [ ] Two independent sources for key findings
  [ ] Direct user quotes recorded verbatim

PHASE CONFIDENCE:  ___% (must reach 80% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```
