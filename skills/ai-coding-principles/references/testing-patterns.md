# Testing Patterns

Practical testing strategies for AI coding agents.

## Test Classification

### Unit Tests

**When:** Pure functions, isolated logic, utilities

```typescript
// Function to test
function calculateDiscount(price: number, discountPercent: number): number {
  if (discountPercent < 0 || discountPercent > 100) {
    throw new Error('Invalid discount percentage');
  }
  return price * (1 - discountPercent / 100);
}

// Unit tests
describe('calculateDiscount', () => {
  it('applies percentage discount correctly', () => {
    expect(calculateDiscount(100, 20)).toBe(80);
  });

  it('handles zero discount', () => {
    expect(calculateDiscount(100, 0)).toBe(100);
  });

  it('handles full discount', () => {
    expect(calculateDiscount(100, 100)).toBe(0);
  });

  it('rejects invalid percentages', () => {
    expect(() => calculateDiscount(100, -10)).toThrow();
    expect(() => calculateDiscount(100, 150)).toThrow();
  });
});
```

### Integration Tests

**When:** API endpoints, database operations, service interactions

```typescript
describe('POST /api/users', () => {
  beforeEach(async () => {
    await db.users.deleteMany();
  });

  it('creates a new user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'Test User' });

    expect(response.status).toBe(201);
    expect(response.body.email).toBe('test@example.com');

    // Verify database state
    const user = await db.users.findOne({ email: 'test@example.com' });
    expect(user).toBeDefined();
  });

  it('rejects duplicate emails', async () => {
    await db.users.create({ email: 'test@example.com', name: 'Existing' });

    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'Duplicate' });

    expect(response.status).toBe(409);
  });
});
```

### E2E Tests

**When:** Critical user journeys only

```typescript
describe('Checkout Flow', () => {
  it('completes purchase successfully', async () => {
    await page.goto('/products');
    await page.click('[data-testid="product-1"]');
    await page.click('[data-testid="add-to-cart"]');
    await page.click('[data-testid="checkout"]');

    await page.fill('[name="email"]', 'buyer@example.com');
    await page.fill('[name="card"]', '4242424242424242');
    await page.click('[data-testid="pay"]');

    await expect(page.locator('[data-testid="success"]')).toBeVisible();
  });
});
```

## Test Naming

### Pattern: should_expectedBehavior_when_condition

```typescript
describe('UserService', () => {
  it('should return user when valid id provided', () => {});
  it('should throw NotFoundError when user does not exist', () => {});
  it('should hash password when creating user', () => {});
});
```

### Alternative: given_when_then

```typescript
describe('UserService.createUser', () => {
  it('given valid data, when creating user, then returns created user', () => {});
  it('given duplicate email, when creating user, then throws ConflictError', () => {});
});
```

## Mocking Strategies

### Mock External Dependencies

```typescript
// Mock external API
jest.mock('./stripe', () => ({
  createCharge: jest.fn().mockResolvedValue({ id: 'ch_123' }),
}));

// Test
it('processes payment', async () => {
  const result = await processOrder(order);

  expect(stripe.createCharge).toHaveBeenCalledWith({
    amount: order.total,
    currency: 'usd',
  });
  expect(result.status).toBe('paid');
});
```

### Don't Mock What You Own

```typescript
// BAD: Mocking your own repository
jest.mock('./userRepository');
userRepository.findById.mockResolvedValue(mockUser);

// GOOD: Use test database
beforeEach(async () => {
  await db.users.create(testUser);
});

it('fetches user by id', async () => {
  const user = await userService.getById(testUser.id);
  expect(user.email).toBe(testUser.email);
});
```

## Test Data

### Factories

```typescript
function createTestUser(overrides: Partial<User> = {}): User {
  return {
    id: randomUUID(),
    email: `test-${randomUUID()}@example.com`,
    name: 'Test User',
    createdAt: new Date(),
    ...overrides,
  };
}

// Usage
const user = createTestUser({ name: 'Custom Name' });
```

### Fixtures

```typescript
const fixtures = {
  validUser: {
    email: 'valid@example.com',
    name: 'Valid User',
    password: 'SecurePass123!',
  },
  invalidEmails: ['not-an-email', '@missing.local', 'spaces in@email.com'],
};
```

## What Not to Test

### Framework Code

```typescript
// DON'T test that Express routes work
// DON'T test that React renders components
// DON'T test that ORM saves to database

// DO test YOUR logic using these tools
```

### Trivial Code

```typescript
// DON'T test simple getters/setters
class User {
  getName() { return this.name; }  // Don't test this
}

// DO test getters with logic
class User {
  getDisplayName() {  // Test this
    return this.nickname || this.name || 'Anonymous';
  }
}
```

## Testing Checklist

### For New Features

- [ ] Happy path covered
- [ ] Error cases covered
- [ ] Edge cases covered (empty, null, boundary values)
- [ ] Tests are independent
- [ ] Tests clean up after themselves

### For Bug Fixes

- [ ] Test reproduces the bug (fails before fix)
- [ ] Test passes after fix
- [ ] Related edge cases covered
