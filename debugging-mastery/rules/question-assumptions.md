---
title: Question Assumptions
impact: high
tags: [debugging, mindset, methodology]
---

# Question Assumptions

**The bug is often in what you're certain is correct.**

## The Rule

When stuck:
1. List what you're assuming is correct
2. Verify each assumption
3. The bug is often hiding behind false certainty

## Why This Matters

```
Most difficult bugs survive because:
- We're sure the data is correct (it isn't)
- We're sure that function works (it doesn't)
- We're sure config is right (it isn't)
- We're sure it's not X (it is X)

The longer you're stuck, the more likely
the bug is in something you haven't questioned.
```

## The Assumption Audit

```markdown
When stuck for more than 30 minutes, list:

"I am certain that:"
1. The database has the correct data
2. The API is returning valid responses
3. The configuration is correct
4. The input is properly formatted
5. The user has the right permissions

Then verify EACH ONE. The bug is likely in this list.
```

## Incorrect Approach

```typescript
// Bug: User profile not loading

// Developer's assumptions:
// - "The API definitely works, I tested it yesterday"
// - "The data is definitely in the database, I saw it"
// - "The frontend code is definitely right, I just reviewed it"

// Developer spends 4 hours on frontend code
// Never checks API or database
// Bug was: Database query had typo in column name
```

## Correct Approach

```typescript
// Bug: User profile not loading

// Step 1: List assumptions
// - API returns correct data
// - Database has the data
// - Frontend parses response correctly
// - Auth token is valid

// Step 2: Verify each one

// Verify: API returns correct data
curl https://api.example.com/users/123
// Response: { "user": null }  ← Data missing!

// Found it: API is returning null
// Assumption "API works" was wrong

// Step 3: Trace why API returns null
// Database query has typo: "user_ids" should be "user_id"

// Bug found in 15 minutes by questioning assumptions
```

## Common False Assumptions

| Assumption | Reality Check |
|------------|---------------|
| "The config is correct" | Print it. Check environment. |
| "The data exists" | Query the database directly. |
| "The API works" | Call it manually with curl. |
| "The cache is fresh" | Clear cache and retry. |
| "The right version is deployed" | Check actual deployed code. |
| "Permissions are correct" | Test with the actual user. |
| "The error message is accurate" | Trace to see what really happened. |
| "It worked in staging" | Compare environments exactly. |
| "Nobody else changed anything" | Check git log, deployments. |
| "The third-party API is fine" | Check their status page, call directly. |

## The Certainty Trap

```
Certainty level vs. likelihood of checking:

"I'm 100% sure it's not X" → 0% chance of checking X
"I'm pretty sure it's not X" → 10% chance of checking X
"It might be X" → 50% chance of checking X
"It's probably X" → 90% chance of checking X

The inverse relationship:
The more certain you are, the less likely you'll check.
The less likely you check, the more likely the bug hides there.

→ High certainty = Good hiding spot for bugs
```

## Assumption Verification Checklist

```markdown
## Things I'm "Sure" Are Correct

### Data
- [ ] I verified the input data is what I expect
- [ ] I verified the database contains expected records
- [ ] I verified the data types are correct
- [ ] I verified timestamps/timezones are correct

### API/Integration
- [ ] I called the API directly (curl/Postman)
- [ ] I verified the request format is correct
- [ ] I verified the response is what I expect
- [ ] I verified auth tokens are valid and not expired

### Configuration
- [ ] I printed the actual config values
- [ ] I verified environment variables are set
- [ ] I verified the right config file is loaded
- [ ] I verified values aren't being overridden

### Code
- [ ] I verified this code path is actually executed
- [ ] I verified the function is called with expected args
- [ ] I verified the return value is what I expect
- [ ] I verified no exceptions are being swallowed

### Environment
- [ ] I verified the correct code version is deployed
- [ ] I verified dependencies are correct versions
- [ ] I verified no cached/stale code
- [ ] I verified I'm testing in the right environment
```

## The Fresh Eyes Test

```markdown
If you've been stuck for an hour:

1. Take a break (physically walk away)

2. Come back and pretend you know nothing
   - "What if the API is broken?"
   - "What if the data is wrong?"
   - "What if I'm testing the wrong thing?"

3. Explain to someone else
   - "It can't be X because..."
   - Often, you'll realize you haven't verified X

4. Ask someone to check your assumptions
   - "I'm sure the config is right, but can you verify?"
   - Fresh eyes see what you've become blind to
```

## The Beginner's Mind

```
Expert: "It can't be the network, networking always works"
        → Doesn't check network
        → Bug was network timeout

Beginner: "I don't know if the network works"
          → Checks network
          → Finds bug immediately

Approach debugging like a beginner:
- Question everything
- Verify everything
- Assume nothing
```
