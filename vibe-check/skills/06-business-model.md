# 06 — Business Model

**Goal:** Ensure the project is economically viable, not just technically interesting or emotionally satisfying.

**When to use independently:** When evaluating whether to commit real resources to a project, or when a project feels right but the economics have never been stress-tested.

---

## Question Matrix

| # | Question | What It Exposes | Evidence Type |
|---|----------|-----------------|---------------|
| 1 | How does this make money, save money, or save time that converts to money? Give the specific mechanism, not the category. | Revenue model clarity vs vague monetization hope | Stated |
| 2 | What would a user pay for this? Not "would they pay" — give me the specific number and the reasoning behind it. | Price anchoring quality and willingness-to-pay evidence | Stated |
| 3 | How did you arrive at that price? Did a user confirm it, did you benchmark it, or did you guess? | Pricing evidence grade | Stated |
| 4 | What does it cost you to acquire one paying user? Walk me through the math — not the marketing category. | CAC reality and acquisition model clarity | Quantitative |
| 5 | What does it cost to serve one user for one month? Include infrastructure, support time, tooling, and your own time. | Unit economics and margin reality | Quantitative |
| 6 | At what scale does this become profitable? Show the math — exact number of users or transactions. | Break-even clarity and scale requirement | Quantitative |
| 7 | What is the lifetime value of one user? How did you calculate retention to arrive at that number? | LTV assumption quality and churn modeling | Quantitative |
| 8 | What is your LTV:CAC ratio? What is your payback period? What does industry standard look like for this category? | Unit economics health check | Quantitative |
| 9 | Who else in the value chain captures revenue from this problem being solved? Are they a partner or a competitor? | Value chain position and margin capture risk | Stated |
| 10 | What is your pricing strategy if a well-funded competitor enters and prices at 30% below you? | Competitive pricing resilience | Stated |
| 11 | What does the cash flow look like in months 1, 3, 6, and 12? When do you run out of money if nothing goes right? | Runway reality and financial risk | Quantitative |
| 12 | If this never makes money but creates significant value, is that acceptable? For whom, and for how long? | Mission vs margin alignment — especially for internal tools | Stated |

---

## Unit Economics Health Check

Fill this during the phase. If numbers cannot be estimated, that is a critical finding:

```
UNIT ECONOMICS SNAPSHOT
──────────────────────────────────────────────────
Revenue per user per month:    $___
Cost to serve per month:       $___
Gross margin per user:         $___  (___%)

Customer Acquisition Cost:     $___
Lifetime (months):             ___
Lifetime Value:                $___

LTV:CAC Ratio:                 ___:1  (target: > 3:1)
Payback period:                ___ months  (target: < 12)
Gross margin:                  ___%  (target: > 60% for SaaS)

Break-even users/transactions: ___
Break-even timeline:           ___ months

STATUS
  [ ] LTV:CAC > 3:1
  [ ] Payback < 12 months
  [ ] Gross margin > 60%
  [ ] Break-even achievable within runway
──────────────────────────────────────────────────
```

---

## Follow-up Probes (use when confidence < 70%)

- You said users would pay $[X]. Has anyone committed to that price in writing or with a deposit, or is this a conversational "yeah, I'd probably pay that"?
- Your CAC assumes [channel]. What is your evidence that channel converts at that cost? Have you run any paid tests?
- Walk me through the retention assumption behind your LTV. What percentage of users renew after month 1? Month 3? Month 12?
- If your cost to serve doubles because of [specific scenario], does the model still work?
- Who is the economic buyer? Is it the same person as the end user? How does that change your pricing and acquisition strategy?

---

## Contradiction Patterns to Watch

| Pattern | Signal |
|---------|--------|
| Pricing by comparison | "Competitors charge $X so we'll charge $Y" without user willingness evidence |
| CAC optimism | Assumes cost-per-acquisition from a channel they have never tested |
| Retention fiction | LTV calculated with retention numbers from a different product category |
| Unit economics bypass | "We'll figure out monetization later" for a product with real infrastructure cost |
| Break-even at scale | Break-even requires market share that is unrealistic for the resource level |

---

## Output Artifact: Unit Economics Card

```
UNIT ECONOMICS CARD
─────────────────────────────────────────────────────────
Project:          ___
Date:             ___
Revenue Model:    ___  (subscription / transaction / licensing / internal)

PRICING
  Price per user:         $___/month  or  $___/transaction
  Pricing evidence:       Validated / Benchmarked / Assumed
  Source:                 ___

COST STRUCTURE
  Infrastructure (per user/month): $___
  Support time (per user/month):   $___
  Other (per user/month):          $___
  Total cost to serve:             $___/month

UNIT ECONOMICS
  Revenue per user:    $___
  Cost per user:       $___
  Gross margin:        $___  (___%)
  CAC:                 $___  (source: ___)
  LTV:                 $___  (retention assumption: ___ months)
  LTV:CAC:             ___:1
  Payback period:      ___ months

SCALE
  Break-even at:       ___ users
  Timeline to break-even: ___ months
  Runway available:    ___ months

CASH FLOW SKETCH
  Month 1:   $___
  Month 3:   $___
  Month 6:   $___
  Month 12:  $___

HEALTH STATUS
  LTV:CAC:      [ ] Pass (>3:1)  [ ] Fail  [ ] Unknown
  Payback:      [ ] Pass (<12m)  [ ] Fail  [ ] Unknown
  Margin:       [ ] Pass (>60%)  [ ] Fail  [ ] Unknown

PHASE CONFIDENCE:  ___% (must reach 75% before proceeding)
EVIDENCE GRADE:    A / B / C / D / F
─────────────────────────────────────────────────────────
```
