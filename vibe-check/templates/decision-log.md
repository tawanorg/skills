# Decision Log Template

Populated primarily during skill 08 (Technical Reality) but captures all significant decisions across the full discovery process — strategic, product, and technical.

---

## Decision Log

Add every decision that is not trivially reversible. When in doubt, log it:

```
DECISION LOG
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
#  | Decision                    | Options Considered           | Choice Made     | Rationale              | Reversibility | Confidence | Date   | Owner
---|----------------------------|------------------------------|-----------------|------------------------|---------------|------------|--------|-------
1  |                            | Option A / Option B / None   |                 |                        | Easy          | High       |        |
2  |                            |                              |                 |                        | Hard          | Med        |        |
3  |                            |                              |                 |                        | Permanent     | Low        |        | ← FLAG
4  |                            |                              |                 |                        |               |            |        |
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Reversibility:
  Easy      — Can be changed in a day with no user impact
  Hard      — Requires migration, refactor, or user communication. Weeks of work.
  Permanent — Cannot be changed without breaking existing users or losing data

Confidence:
  High — Evidence supports this choice. Would make the same call with more information.
  Med  — Reasonable choice given current information. May revisit.
  Low  — Made under uncertainty. Needs to be confirmed when more information is available.
```

---

## High-Risk Decisions

Decisions marked Permanent + Low Confidence require immediate attention. These are the ones that will be regretted:

```
HIGH-RISK DECISIONS (Permanent + Low Confidence)
─────────────────────────────────────────────────────────
Decision #___: ___
  Why permanent:    ___
  Why low confidence: ___
  What would raise confidence: ___
  Resolution plan:  ___
  Deadline:         ___

Decision #___: ___
  Why permanent:    ___
  Why low confidence: ___
  What would raise confidence: ___
  Resolution plan:  ___
  Deadline:         ___
─────────────────────────────────────────────────────────
```

---

## Decision Categories

Use these categories to filter and review the log:

| Category | What it covers |
|----------|---------------|
| Strategic | Build vs buy vs partner, target market, business model |
| Product | Core feature set, scope boundaries, user prioritization |
| Architecture | Data model, infrastructure, API design, technology choices |
| Process | Team structure, development methodology, release cadence |
| Compliance | Data handling, security posture, regulatory approach |

---

## Review Triggers

Re-examine any decision in the log when:
- New evidence contradicts the rationale
- A previously Hard decision is now being treated as Permanent without review
- Confidence has decreased due to failed assumptions
- A new team member needs to understand why choices were made

```
DECISION REVIEW LOG
─────────────────────────────────────────────────────────
Decision # | Review Date | Trigger | Outcome
-----------|-------------|---------|--------
           |             |         | Confirmed / Revised / Reversed
─────────────────────────────────────────────────────────
```

---

## Log Status Summary

```
DECISION LOG STATUS
─────────────────────────────────────────────────────────
Total decisions logged:            ___

By reversibility:
  Easy:      ___
  Hard:      ___
  Permanent: ___

By confidence:
  High:      ___
  Med:       ___
  Low:       ___

HIGH RISK (Permanent + Low):       ___  ← Must reach 0 or have resolution plan
Decisions pending review:          ___
─────────────────────────────────────────────────────────
```
