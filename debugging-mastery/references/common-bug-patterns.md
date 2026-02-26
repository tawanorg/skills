# Common Bug Patterns

The usual suspects. When debugging, check these first.

## Off-By-One Errors

```typescript
// Pattern: Wrong boundary condition

// Bug: Missing last element
const items = ['a', 'b', 'c'];
for (let i = 0; i < items.length - 1; i++) {  // Wrong: < length - 1
  process(items[i]);
}
// Processes: a, b (misses 'c')

// Fix:
for (let i = 0; i < items.length; i++) {  // Correct: < length
  process(items[i]);
}

// Bug: Array index out of bounds
const last = items[items.length];  // Wrong: index === length is out of bounds
// Fix:
const last = items[items.length - 1];

// Bug: Fence post error
// "Build a 100-meter fence with posts every 10 meters"
const posts = 100 / 10;  // Wrong: 10 posts
const posts = 100 / 10 + 1;  // Correct: 11 posts (need one at each end)
```

## Null/Undefined References

```typescript
// Pattern: Accessing property of null/undefined

// Bug: Deep access without checks
const theme = user.profile.settings.theme;
// Crashes if user, profile, or settings is undefined

// Fix: Optional chaining
const theme = user?.profile?.settings?.theme ?? 'default';

// Bug: Array method on undefined
const items = data.items.map(i => i.name);
// Crashes if data.items is undefined

// Fix: Default to empty array
const items = (data.items ?? []).map(i => i.name);

// Bug: Destructuring undefined
const { name, email } = getUser();  // Crashes if getUser returns undefined

// Fix: Default value
const { name, email } = getUser() ?? {};
```

## Race Conditions

```typescript
// Pattern: Timing-dependent bugs

// Bug: Check-then-act race
async function withdraw(accountId: string, amount: number) {
  const balance = await getBalance(accountId);  // Read
  if (balance >= amount) {                      // Check
    // Another request could withdraw here!
    await setBalance(accountId, balance - amount);  // Act
  }
}

// Fix: Atomic operations or locks
async function withdraw(accountId: string, amount: number) {
  return await db.transaction(async (tx) => {
    const account = await tx.accounts
      .where({ id: accountId })
      .forUpdate()  // Lock row
      .first();

    if (account.balance >= amount) {
      await tx.accounts
        .where({ id: accountId })
        .update({ balance: account.balance - amount });
      return true;
    }
    return false;
  });
}

// Bug: Missing await
async function processOrders(orders) {
  orders.forEach(order => {
    processOrder(order);  // Missing await! Runs in parallel unexpectedly
  });
  console.log('All done');  // Logs before processing finishes
}

// Fix: Use for...of or Promise.all
async function processOrders(orders) {
  for (const order of orders) {
    await processOrder(order);  // Sequential
  }
  // Or
  await Promise.all(orders.map(o => processOrder(o)));  // Parallel
  console.log('All done');
}
```

## State Mutation Bugs

```typescript
// Pattern: Unexpected object mutation

// Bug: Mutating shared object
function applyDiscount(order) {
  order.total = order.total * 0.9;  // Mutates original!
  return order;
}

const order = { total: 100 };
const discounted = applyDiscount(order);
console.log(order.total);  // 90 - original was mutated!

// Fix: Return new object
function applyDiscount(order) {
  return {
    ...order,
    total: order.total * 0.9
  };
}

// Bug: Array mutation
function addItem(items, newItem) {
  items.push(newItem);  // Mutates original!
  return items;
}

// Fix: Return new array
function addItem(items, newItem) {
  return [...items, newItem];
}
```

## Type Coercion Bugs

```typescript
// Pattern: JavaScript's loose typing

// Bug: String concatenation instead of addition
function addNumbers(a, b) {
  return a + b;
}
addNumbers('5', 3);  // Returns '53' not 8

// Fix: Explicit conversion
function addNumbers(a: number, b: number) {
  return Number(a) + Number(b);
}

// Bug: Loose equality
if (userId == null) {  // True for both null AND undefined
  // ...
}

// Fix: Strict equality
if (userId === null) {  // Only true for null
  // ...
}
if (userId === undefined) {  // Only true for undefined
  // ...
}

// Bug: Boolean coercion
const items = [];
if (items) {  // True! Empty array is truthy
  console.log('Has items');
}

// Fix: Check length
if (items.length > 0) {
  console.log('Has items');
}
```

## Async/Await Bugs

```typescript
// Pattern: Promise handling errors

// Bug: Unhandled rejection
async function fetchData() {
  const data = await api.get('/data');  // If this fails...
  return process(data);  // ...this line never runs, error bubbles up
}
// No catch anywhere - unhandled rejection

// Fix: Handle errors
async function fetchData() {
  try {
    const data = await api.get('/data');
    return process(data);
  } catch (error) {
    logger.error('Failed to fetch data', error);
    throw error;  // Re-throw or return default
  }
}

// Bug: Parallel vs sequential confusion
async function fetchAll() {
  const a = await fetchA();  // Wait for A
  const b = await fetchB();  // Then wait for B
  const c = await fetchC();  // Then wait for C
  return [a, b, c];  // Total time: A + B + C
}

// Fix: Parallel when independent
async function fetchAll() {
  const [a, b, c] = await Promise.all([
    fetchA(),
    fetchB(),
    fetchC()
  ]);  // Total time: max(A, B, C)
  return [a, b, c];
}

// Bug: forEach with async
async function processItems(items) {
  items.forEach(async (item) => {
    await processItem(item);  // These run in parallel!
  });
  console.log('Done');  // Logs before processing finishes
}

// Fix: for...of
async function processItems(items) {
  for (const item of items) {
    await processItem(item);  // Sequential
  }
  console.log('Done');  // Logs after all processing
}
```

## Date/Time Bugs

```typescript
// Pattern: Timezone and date handling

// Bug: Local time vs UTC
const date = new Date('2024-01-15');  // Midnight UTC
console.log(date.getDate());  // Might be 14 or 15 depending on timezone!

// Fix: Be explicit
const date = new Date('2024-01-15T00:00:00Z');  // Explicit UTC
// Or use date library

// Bug: Month indexing
const date = new Date(2024, 1, 15);  // February 15, not January!
// Months are 0-indexed in JavaScript

// Fix: Use explicit methods or ISO strings
const date = new Date('2024-01-15');  // January 15

// Bug: Date comparison
const date1 = new Date('2024-01-15');
const date2 = new Date('2024-01-15');
if (date1 === date2) {  // false! Different objects
  // ...
}

// Fix: Compare timestamps
if (date1.getTime() === date2.getTime()) {  // true
  // ...
}
```

## Floating Point Bugs

```typescript
// Pattern: Floating point precision

// Bug: Floating point arithmetic
const total = 0.1 + 0.2;
console.log(total);  // 0.30000000000000004

if (total === 0.3) {  // false!
  // ...
}

// Fix: Use integers for money
const priceInCents = 10 + 20;  // 30 cents
const dollars = priceInCents / 100;  // $0.30

// Fix: Compare with epsilon
const epsilon = 0.0001;
if (Math.abs(total - 0.3) < epsilon) {  // true
  // ...
}

// Fix: Use decimal library for financial calculations
import Decimal from 'decimal.js';
const total = new Decimal('0.1').plus('0.2');
console.log(total.toString());  // '0.3'
```

## Closure Bugs

```typescript
// Pattern: Stale closures

// Bug: Loop variable capture
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 3, 3, 3 (not 0, 1, 2)

// Fix: Use let
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Logs: 0, 1, 2

// Bug: Stale state in React
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setCount(count + 1);  // count is always 0 (stale closure)
    }, 1000);
    return () => clearInterval(interval);
  }, []);  // Empty deps, count is captured at mount

  return <div>{count}</div>;  // Always shows 1
}

// Fix: Functional update
setCount(prevCount => prevCount + 1);  // Uses current value
```

## Encoding Bugs

```typescript
// Pattern: Character encoding issues

// Bug: URL encoding
const url = `https://api.example.com/search?q=${query}`;
// If query = "hello world", URL is invalid

// Fix: Encode
const url = `https://api.example.com/search?q=${encodeURIComponent(query)}`;

// Bug: JSON in HTML
const html = `<script>const data = ${JSON.stringify(userInput)};</script>`;
// If userInput contains </script>, XSS vulnerability

// Fix: Escape properly
const html = `<script>const data = ${JSON.stringify(userInput).replace(/</g, '\\u003c')};</script>`;

// Bug: Base64 padding
const decoded = atob(base64String);  // Fails if padding is wrong

// Fix: Handle padding
function safeDecode(str) {
  const padding = '='.repeat((4 - str.length % 4) % 4);
  return atob(str + padding);
}
```

## Quick Diagnostic Questions

```markdown
When debugging, ask:

## For crashes
- What was null/undefined?
- What array index was out of bounds?
- What type conversion failed?

## For wrong results
- Is it off-by-one?
- Is it a timing issue?
- Is state being mutated unexpectedly?
- Is there a type coercion issue?

## For intermittent bugs
- Is there a race condition?
- Is there stale data?
- Is there a timing dependency?

## For data issues
- Is it an encoding problem?
- Is it a timezone issue?
- Is it a floating point issue?
```
