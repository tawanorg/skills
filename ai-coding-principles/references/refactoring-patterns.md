# Refactoring Patterns

Safe refactoring techniques for legacy code transformation.

## When to Refactor

### Refactor When

- Adding a feature is harder than it should be
- The same bug keeps appearing
- Code is difficult to understand
- Tests are hard to write
- You have test coverage to catch regressions

### Don't Refactor When

- No tests exist (write tests first)
- You don't understand what the code does
- It's working and won't be touched again
- You're on a tight deadline (make a note instead)
- As part of an unrelated change

## The Golden Rule

**Make the change easy, then make the easy change.**

```typescript
// If adding a feature is hard, first refactor to make it easy
// Then add the feature in the now-clean code

// Step 1: Refactor (separate PR)
// Step 2: Add feature (easy now)
```

## Safe Refactoring Process

### 1. Ensure Test Coverage

```typescript
// Before refactoring, verify behavior is captured
describe('OrderProcessor', () => {
  it('calculates total with tax', () => {
    const result = processor.calculate(items);
    expect(result.total).toBe(110); // 100 + 10% tax
  });

  it('applies discount before tax', () => {
    const result = processor.calculate(items, { discount: 10 });
    expect(result.total).toBe(99); // (100-10) * 1.1
  });

  // Add characterization tests for unknown behavior
  it('handles empty items', () => {
    const result = processor.calculate([]);
    expect(result.total).toBe(0); // Document current behavior
  });
});
```

### 2. Small, Incremental Changes

```typescript
// BAD: Big bang refactor
// Rewrite entire module in one commit

// GOOD: Series of small changes
// Commit 1: Extract method
// Commit 2: Rename variable
// Commit 3: Move function to new file
// Commit 4: Update callers
// Each commit: tests pass
```

### 3. Separate Refactoring from Features

```typescript
// BAD: One PR with refactoring + new feature
// Hard to review, hard to revert

// GOOD: Two PRs
// PR 1: Refactoring only (behavior unchanged)
// PR 2: New feature (on clean code)
```

## Common Refactoring Patterns

### Extract Function

```typescript
// BEFORE
function processOrder(order: Order) {
  // Validate
  if (!order.items.length) throw new Error('Empty order');
  if (!order.customer) throw new Error('No customer');
  if (order.total < 0) throw new Error('Invalid total');

  // Calculate
  let subtotal = 0;
  for (const item of order.items) {
    subtotal += item.price * item.quantity;
  }
  const tax = subtotal * 0.1;
  const total = subtotal + tax;

  // Save
  db.orders.create({ ...order, subtotal, tax, total });
}

// AFTER
function processOrder(order: Order) {
  validateOrder(order);
  const totals = calculateTotals(order.items);
  saveOrder(order, totals);
}

function validateOrder(order: Order) {
  if (!order.items.length) throw new Error('Empty order');
  if (!order.customer) throw new Error('No customer');
  if (order.total < 0) throw new Error('Invalid total');
}

function calculateTotals(items: OrderItem[]) {
  const subtotal = items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  const tax = subtotal * 0.1;
  return { subtotal, tax, total: subtotal + tax };
}

function saveOrder(order: Order, totals: OrderTotals) {
  db.orders.create({ ...order, ...totals });
}
```

### Replace Conditional with Polymorphism

```typescript
// BEFORE
function calculateShipping(order: Order): number {
  switch (order.shippingMethod) {
    case 'standard':
      return order.weight * 0.5;
    case 'express':
      return order.weight * 1.5 + 10;
    case 'overnight':
      return order.weight * 3 + 25;
    default:
      throw new Error('Unknown shipping method');
  }
}

// AFTER
interface ShippingCalculator {
  calculate(order: Order): number;
}

class StandardShipping implements ShippingCalculator {
  calculate(order: Order): number {
    return order.weight * 0.5;
  }
}

class ExpressShipping implements ShippingCalculator {
  calculate(order: Order): number {
    return order.weight * 1.5 + 10;
  }
}

class OvernightShipping implements ShippingCalculator {
  calculate(order: Order): number {
    return order.weight * 3 + 25;
  }
}

const shippingCalculators: Record<string, ShippingCalculator> = {
  standard: new StandardShipping(),
  express: new ExpressShipping(),
  overnight: new OvernightShipping(),
};

function calculateShipping(order: Order): number {
  const calculator = shippingCalculators[order.shippingMethod];
  if (!calculator) throw new Error('Unknown shipping method');
  return calculator.calculate(order);
}
```

### Introduce Parameter Object

```typescript
// BEFORE
function searchUsers(
  name?: string,
  email?: string,
  role?: string,
  createdAfter?: Date,
  createdBefore?: Date,
  isActive?: boolean,
  limit?: number,
  offset?: number
) {
  // ...
}

// AFTER
interface UserSearchParams {
  name?: string;
  email?: string;
  role?: string;
  createdAfter?: Date;
  createdBefore?: Date;
  isActive?: boolean;
  pagination?: {
    limit: number;
    offset: number;
  };
}

function searchUsers(params: UserSearchParams) {
  // ...
}
```

### Replace Magic Numbers/Strings

```typescript
// BEFORE
if (user.loginAttempts > 5) {
  user.status = 'locked';
  setTimeout(unlockUser, 900000); // What is 900000?
}

// AFTER
const MAX_LOGIN_ATTEMPTS = 5;
const LOCKOUT_DURATION_MS = 15 * 60 * 1000; // 15 minutes

const UserStatus = {
  ACTIVE: 'active',
  LOCKED: 'locked',
} as const;

if (user.loginAttempts > MAX_LOGIN_ATTEMPTS) {
  user.status = UserStatus.LOCKED;
  setTimeout(unlockUser, LOCKOUT_DURATION_MS);
}
```

### Decompose Conditional

```typescript
// BEFORE
function getPrice(date: Date, quantity: number): number {
  if (date.getMonth() >= 5 && date.getMonth() <= 8 &&
      date.getDay() !== 0 && date.getDay() !== 6 &&
      quantity >= 10) {
    return basePrice * 0.8;
  }
  return basePrice;
}

// AFTER
function getPrice(date: Date, quantity: number): number {
  if (isSummerWeekday(date) && isBulkOrder(quantity)) {
    return applyBulkSummerDiscount(basePrice);
  }
  return basePrice;
}

function isSummerWeekday(date: Date): boolean {
  const month = date.getMonth();
  const day = date.getDay();
  const isSummer = month >= 5 && month <= 8;
  const isWeekday = day !== 0 && day !== 6;
  return isSummer && isWeekday;
}

function isBulkOrder(quantity: number): boolean {
  return quantity >= 10;
}

function applyBulkSummerDiscount(price: number): number {
  return price * 0.8;
}
```

## Legacy Code Strategies

### Strangler Fig Pattern

```typescript
// Gradually replace old system with new

// Phase 1: Route some traffic to new
const useNewSystem = featureFlags.isEnabled('new-order-processor');

async function processOrder(order: Order) {
  if (useNewSystem) {
    return newOrderProcessor.process(order);
  }
  return legacyOrderProcessor.process(order);
}

// Phase 2: Increase percentage over time
// Phase 3: Remove old system when 100% migrated
```

### Sprout Method

```typescript
// Add new functionality in clean code, call from legacy

class LegacyOrderProcessor {
  process(order: Order) {
    // ... lots of legacy code ...

    // Instead of modifying legacy code, sprout new method
    const discount = this.calculateLoyaltyDiscount(order.customer);
    order.total -= discount;

    // ... more legacy code ...
  }

  // New, clean, tested method
  private calculateLoyaltyDiscount(customer: Customer): number {
    if (!customer.loyaltyTier) return 0;

    const discountRates = {
      silver: 0.05,
      gold: 0.10,
      platinum: 0.15,
    };

    return order.subtotal * (discountRates[customer.loyaltyTier] || 0);
  }
}
```

### Wrap Method

```typescript
// BEFORE: Need to add logging to existing method
function processPayment(payment: Payment) {
  // existing logic
}

// AFTER: Wrap with new behavior
function processPayment(payment: Payment) {
  logPaymentAttempt(payment);
  processPaymentOriginal(payment);
  logPaymentSuccess(payment);
}

function processPaymentOriginal(payment: Payment) {
  // existing logic, unchanged
}
```

## Refactoring Checklist

### Before Starting

- [ ] Tests exist and pass
- [ ] I understand what the code does
- [ ] I have a clear goal for the refactoring
- [ ] This is a separate PR from feature work

### During Refactoring

- [ ] Making small, incremental changes
- [ ] Running tests after each change
- [ ] Committing frequently
- [ ] Not changing behavior

### After Refactoring

- [ ] All tests pass
- [ ] Code is more readable/maintainable
- [ ] No behavior changes (unless that was the goal)
- [ ] PR is reviewable (not too large)

## When Refactoring Goes Wrong

### Signs to Stop

- Tests start failing unexpectedly
- You don't understand why something works
- The refactoring is taking much longer than expected
- You're making "one more change" repeatedly

### Recovery

```bash
# If things go wrong, git is your friend
git stash              # Save current work
git checkout .         # Discard changes
git stash pop          # Get work back if needed

# Or reset to last known good state
git reset --hard HEAD~3  # Undo last 3 commits
```

### Prevention

- Commit frequently
- Keep changes small
- Write tests first
- Know when to stop
