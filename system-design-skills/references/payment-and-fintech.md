# Payment and Fintech

Money systems trade latency and convenience for correctness. This reference covers how card payments move, how to model money safely with ledgers, and the operational patterns that keep a payment system trustworthy under partial failure.

## The Players in a Card Payment

A single card swipe involves five-plus independent parties, each with a distinct role, contract, and fee.

| Party | Role |
|---|---|
| Cardholder | The consumer who owns the card and authorizes the purchase. |
| Merchant | The business selling goods/services; wants to get paid reliably. |
| Acquirer (acquiring bank) | The merchant's bank. Holds the merchant account, submits transactions into the network, receives funds on the merchant's behalf, and carries chargeback liability. |
| Card network (Visa, Mastercard, Amex) | The "rails" that route messages between acquirer and issuer, set interchange rules, and define the message format. Amex/Discover are also issuers (closed-loop). |
| Issuing bank (issuer) | The cardholder's bank. Issued the card, owns the credit/funds, approves or declines authorizations, and bills the cardholder. |
| Payment gateway | Merchant-facing software that captures card data at checkout, encrypts/tokenizes it, and forwards to the processor. Think of it as the on-ramp. |
| Payment processor | Operational backend that formats and transmits transaction messages to the network on the acquirer's behalf and handles clearing/settlement files. |
| PSP (e.g. Stripe, Adyen, PayPal) | Aggregator that bundles gateway + processor + sub-merchant acquiring into one API, so merchants don't contract each party separately. |

Interchange (paid to the issuer) + network assessment (paid to the network) + acquirer markup = the merchant discount rate the merchant ultimately pays.

## The Flow: Authorization → Capture → Clearing → Settlement

These are four distinct phases. Conflating authorization with settlement is the most common mental-model bug.

- **Authorization**: a real-time check. "Is this card valid and are funds/credit available?" The issuer places a *hold* on the cardholder's available balance. **No money moves.** An auth can be reversed or simply expire (typically 5-7 days) if never captured.
- **Capture**: the merchant tells the network "make this authorized amount real, ship it for collection." Often deferred until goods ship. Capture amount can be ≤ auth amount (partial capture).
- **Clearing**: batched, end-of-day exchange of captured transactions between acquirer and issuer through the network. Calculates net positions and interchange. This is bookkeeping, not cash.
- **Settlement**: actual movement of funds. The network instructs the banks; the issuer pays the acquirer (net of interchange), the acquirer credits the merchant (net of its fees). Usually T+1 to T+2.

```
Cardholder  Merchant   Gateway/PSP   Acquirer    Network     Issuer
    |  tap     |            |            |           |           |
    |--------->|            |            |           |           |
    |          |--card----->|            |           |           |
    |          |            |--authReq-->|--------->|---------->|
    |          |            |            |           |  approve/ |
    |          |            |            |           |  hold $$  |
    |          |            |<--approved-(authResp)<-|<----------|
    |<--receipt|<-----------|            |           |           |
    .          .   ...later (goods ship)...          .           .
    |          |--capture-->|----------->|--clearing batch------>|   (T+0 EOD)
    |          |            |            |           |           |
    |          |            |   <==== settlement: issuer -> acquirer -> merchant ====>  (T+1/T+2)
```

Refunds and chargebacks run *backwards* through this same chain after settlement, as new transactions.

## Ledger Design: Double-Entry Bookkeeping

Model money as an append-only log of immutable entries, never as a mutable `balance` column you increment. A mutable balance loses history, races under concurrency, and cannot answer "how did we get here?"

**Double-entry rule**: every transaction touches at least two accounts, and the sum of debits equals the sum of credits. Money is never created or destroyed, only moved between accounts. This invariant — `SUM(debits) == SUM(credits)` across the whole ledger at all times — is your strongest correctness check.

Each account has a normal side. For asset/expense accounts a debit increases the balance; for liability/equity/revenue accounts a credit increases it. A customer's wallet is a *liability* to you (you owe them their money), so a credit increases what you owe.

```text
# Customer tops up $50 into their wallet via card. Amounts in minor units (cents).
transaction_id: txn_8f21  type: topup  currency: USD  created_at: 2026-06-29T10:00:00Z
  entry  account=cash_in_transit   direction=DEBIT   amount=5000
  entry  account=customer:42:wallet direction=CREDIT  amount=5000
# invariant: 5000 debit == 5000 credit  ✓
```

Entries are immutable. To correct a mistake you post a *reversing* transaction, never an UPDATE or DELETE. The balance of any account is **derived**: `balance = SUM(credits) - SUM(debits)` (sign per account type) over its entries. For performance, snapshot a running balance per account periodically and replay only entries since the snapshot; the ledger remains the source of truth.

Model pending money with intermediate accounts: an authorized-but-uncaptured charge lives in a `pending`/`in_transit` account and only moves to a settled account on capture/settlement, so your ledger mirrors reality at every phase.

## Money Representation

- **Never use floats.** `0.1 + 0.2 != 0.3` in IEEE-754; rounding drift accumulates and fails reconciliation. Store integer **minor units** (cents, satoshis) or a fixed-precision decimal type. `$50.00 → 5000`.
- Track **currency alongside every amount**; an integer `5000` is meaningless without `USD`. Use a `Money{amount: int64, currency: string}` value object and forbid arithmetic across currencies.
- Respect each currency's exponent: USD/EUR have 2 minor digits, JPY has 0, BHD/KWD have 3. Hardcoding "×100" is a bug.
- Define and store an explicit **rounding policy** for FX, splits, and fees (e.g. banker's rounding); allocate remainders deterministically (largest-remainder method) so split amounts sum exactly to the total.

## Idempotency

Networks retry. A client that times out doesn't know if the charge went through, so it retries — and without protection you double-charge.

- The client generates a unique **idempotency key** (UUID) per logical operation and sends it on every retry of that operation.
- The server stores the key with the request fingerprint and the response. First request: process and persist result under the key. Retries with the same key: return the **stored** response without re-executing.
- Scope keys per endpoint/account and give them a TTL (e.g. 24h). Reject a reused key whose request body differs (it's a client bug, not a retry).
- Persist the key in the **same transaction** as the side effect, so "did it happen?" and "what was the result?" can never disagree. This is how you get effectively-exactly-once semantics on top of at-least-once delivery.

## Reconciliation

Your ledger is your belief about money; the processor/bank statement is ground truth. Reconciliation continuously proves they agree.

- Ingest the processor's settlement report / bank statement (often a daily file). For each external line, match to an internal ledger transaction by amount + currency + reference/external ID + date window.
- Classify outcomes: **matched**, **missing internally** (money the bank says moved but we never recorded — under-recording), **missing externally** (we recorded it but it never settled — possibly stuck/failed), **amount mismatch** (fees, FX, partial capture).
- Auto-match the high-confidence majority; route exceptions to a queue for human/automated investigation with a clear audit trail. Track an **unreconciled aging** metric — exceptions older than N days are an incident.
- Expect and model fees: the merchant nets less than the gross charge, so book interchange/processor fees as their own ledger entries rather than treating the delta as a mismatch.

## Resilient Payment System Principles

1. **Idempotency everywhere** — every mutating call carries a key so retries are safe.
2. **Async processing** — accept the request durably, do slow downstream work (capture, payout, email) via a queue; never make the user wait on a flaky third party.
3. **Continuous reconciliation** — assume your view drifts from the bank's and detect it daily, not at audit time.
4. **Retries with exponential backoff + jitter** — but only on idempotent, retriable errors; respect `Retry-After`; cap attempts.
5. **Circuit breakers** — stop hammering a failing processor; fail fast and shed load to protect the system, then probe for recovery.
6. **Audit trail / immutability** — append-only events and ledger entries; every state change is reconstructable and tamper-evident.
7. **Consistency over availability for money** — when in doubt, refuse rather than risk a wrong balance; a declined payment is recoverable, a double-spend is not.
8. **Observability** — structured logs, metrics, and traces tied to a transaction id; alert on auth success rate, latency, and unreconciled balances.
9. **Graceful degradation** — if non-critical paths (receipts, analytics) fail, the core charge still succeeds; degrade features, not correctness.
10. **Outage handling via queues / failover** — buffer requests during provider outages and drain on recovery; route to a backup PSP where possible; surface honest "pending" states instead of fabricating success.

## Consistency, Sagas, and the Outbox

**Why strong consistency (CP):** money has a single correct value. Under a network partition, payments choose consistency — refuse or hold the operation — over serving a possibly-stale balance. Eventual consistency is fine for a "like" count, not for an account balance. Within a service, wrap ledger writes in a serializable/`SELECT ... FOR UPDATE` transaction so concurrent debits can't both succeed against insufficient funds.

**Saga pattern** for multi-step workflows that span services (reserve funds → charge card → credit merchant → notify): you can't hold one ACID transaction across services, so model the flow as a sequence of local transactions, each with a **compensating action** (e.g. *refund* compensates *charge*). On failure, run compensations in reverse to unwind. Sagas give atomicity-of-outcome without distributed locks; design compensations to be idempotent and remember some steps (a sent email) can only be compensated, not undone.

**Outbox pattern** for reliable events: writing to your DB and publishing to a broker are two systems — a crash between them loses or duplicates events (dual-write problem). Instead, in the *same* DB transaction as the state change, insert the event into an `outbox` table. A separate relay polls (or tails the change log/CDC) the outbox and publishes to the broker, marking rows sent. This guarantees the event is published **iff** the state change committed — at-least-once, deduplicated downstream by event id.

## Webhooks from PSPs

PSPs notify you of async outcomes (payment succeeded, dispute opened) via webhooks. Treat them as untrusted, at-least-once delivery.

- **Verify signatures**: PSPs sign the raw body with a shared secret (HMAC) and a timestamp. Recompute over the *exact raw bytes* before parsing, compare in constant time, and reject stale timestamps to block replay. Never trust an unsigned webhook.
- **Idempotent handling**: the same event will arrive more than once (retries, redeliveries). Dedupe on the event id and make handlers safe to run repeatedly.
- **Respond fast, process async**: acknowledge with 2xx immediately and enqueue the work; a slow handler triggers the PSP's retry storm.
- **Don't trust webhook order or completeness**: events can arrive out of order or be missed entirely. Reconcile against the PSP API as source of truth; treat the webhook as a low-latency hint, not the system of record.

## Compliance & Security

- **PCI DSS** governs handling of cardholder data. The cheapest path to compliance is to never touch raw card data: use the PSP's hosted fields / client-side tokenization so the PAN goes browser→PSP, and your servers only ever see a token. This shrinks your PCI scope dramatically (SAQ-A territory).
- **Tokenization / vaulting**: the PSP stores the real PAN in a hardened vault and hands you an opaque token to charge later. **Never store the raw PAN, and never store the CVV at all** (PCI forbids post-authorization CVV storage). If you must hold card data, encrypt at rest, segment networks, and restrict access.
- **3-D Secure / SCA**: shifts the cardholder authentication (and often fraud liability) to the issuer via an extra challenge (biometric/OTP). EU **PSD2 SCA** mandates strong customer authentication (two of: something you have/know/are) for many transactions, with defined exemptions (low-value, recurring, merchant-initiated).
- Enforce least privilege, encrypt in transit (TLS) and at rest, log access to sensitive data, and rotate secrets.

## Common Failure Modes

- **Partial failures**: card charged but your DB write or downstream step failed. The outbox + ledger-in-same-transaction patterns close most of these gaps; the rest are caught by reconciliation.
- **Timeouts — "did the charge go through?"**: the dangerous ambiguity. Never blindly retry a non-idempotent charge. Retry *with the same idempotency key*, or query the PSP by your reference id to learn the true outcome before acting. Record an intent before calling out, so an orphaned charge can be matched back.
- **Refunds**: a new transaction moving money back to the cardholder; post reversing ledger entries, don't mutate the original.
- **Chargebacks / disputes**: the cardholder disputes via their issuer; funds are pulled back from the merchant (often plus a fee) pending evidence. Model the dispute lifecycle (opened → evidence submitted → won/lost) as ledger entries and reserve against expected losses. Track chargeback rate — networks penalize merchants above thresholds.
- **Double-capture / double-refund**: guard every mutation with idempotency keys and state checks (a captured auth can't be captured again).
- **Stuck/expired authorizations**: auths you never capture expire and silently free the hold; reconcile pending auths and either capture or explicitly reverse them to release the cardholder's funds promptly.
