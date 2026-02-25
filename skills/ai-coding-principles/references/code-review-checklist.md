# Code Review Checklist

Systematic review process for AI coding agents and human reviewers.

## Review Philosophy

**Code review is not about finding bugs.** Tests find bugs.

Code review is about:
- Knowledge sharing
- Maintaining code quality
- Catching design issues early
- Ensuring consistency

## Quick Review Checklist

### Correctness
- [ ] Does the code do what it's supposed to?
- [ ] Are edge cases handled?
- [ ] Are error cases handled appropriately?
- [ ] Are there any obvious bugs?

### Design
- [ ] Is the code in the right place?
- [ ] Does it follow existing patterns?
- [ ] Is it appropriately abstracted (not too much, not too little)?
- [ ] Are dependencies reasonable?

### Readability
- [ ] Can you understand what the code does?
- [ ] Are names clear and consistent?
- [ ] Is the code self-documenting?
- [ ] Are complex parts commented?

### Security
- [ ] Is user input validated?
- [ ] Are there any injection risks?
- [ ] Are secrets handled properly?
- [ ] Are permissions checked?

### Testing
- [ ] Are critical paths tested?
- [ ] Do tests verify behavior, not implementation?
- [ ] Are edge cases covered?
- [ ] Do tests pass?

## Detailed Review Areas

### 1. Functionality

```typescript
// CHECK: Does it solve the stated problem?
// The PR says "fix pagination" - does it actually fix pagination?

// CHECK: Edge cases
// What happens with empty input?
// What happens with null/undefined?
// What happens with very large input?
// What happens with concurrent access?

// CHECK: Error handling
// Are errors caught?
// Are errors logged with context?
// Does the user get appropriate feedback?
```

### 2. Design & Architecture

```typescript
// CHECK: Single Responsibility
// Does this class/function do one thing?

// BAD: Mixed responsibilities
class UserController {
  async createUser(data: UserData) {
    // Validates, creates, sends email, logs analytics...
  }
}

// CHECK: Appropriate abstraction level
// Is it solving today's problem or a hypothetical future one?

// BAD: Over-engineered for one use case
interface DataProcessor<T, U, V> {
  process(input: T, config: U): Promise<V>;
}

// GOOD: Simple and sufficient
function processOrder(order: Order): ProcessedOrder { }

// CHECK: Dependencies
// Does it depend on concrete implementations?
// Could this be tested in isolation?
```

### 3. Code Quality

```typescript
// CHECK: Naming
// Does `data` describe what this contains?
// Is `handleClick` specific enough?

// BAD
const data = await fetch('/api/users');
const handle = (e) => { };

// GOOD
const users = await fetchUsers();
const handleUserFormSubmit = (event: FormEvent) => { };

// CHECK: Complexity
// Is there deep nesting that could be flattened?
// Are there long functions that could be split?

// BAD: Deep nesting
if (user) {
  if (user.isActive) {
    if (user.hasPermission('admin')) {
      // finally do something
    }
  }
}

// GOOD: Early returns
if (!user) return;
if (!user.isActive) return;
if (!user.hasPermission('admin')) return;
// do something

// CHECK: Magic values
// Are there unexplained numbers or strings?

// BAD
if (retries > 3) { }
if (status === 'XYZ_001') { }

// GOOD
const MAX_RETRIES = 3;
if (retries > MAX_RETRIES) { }

const OrderStatus = { PENDING_APPROVAL: 'XYZ_001' } as const;
if (status === OrderStatus.PENDING_APPROVAL) { }
```

### 4. Security Review

```typescript
// CHECK: Input validation
// Is external input validated before use?

// BAD
app.get('/users/:id', (req, res) => {
  db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
});

// GOOD
app.get('/users/:id', (req, res) => {
  const id = validateUUID(req.params.id);
  db.query('SELECT * FROM users WHERE id = $1', [id]);
});

// CHECK: Authorization
// Is the user allowed to perform this action?

// BAD: No authorization check
async function deleteUser(userId: string) {
  await db.users.delete(userId);
}

// GOOD
async function deleteUser(userId: string, requestingUser: User) {
  if (!requestingUser.canDelete(userId)) {
    throw new ForbiddenError();
  }
  await db.users.delete(userId);
}

// CHECK: Data exposure
// Are we returning more than necessary?
// Are sensitive fields excluded?

// BAD
return user;  // Includes password hash, tokens, etc.

// GOOD
return {
  id: user.id,
  name: user.name,
  email: user.email,
};
```

### 5. Performance

```typescript
// CHECK: N+1 queries
// Are we making queries in a loop?

// BAD
const orders = await db.orders.findAll();
for (const order of orders) {
  order.items = await db.orderItems.findByOrderId(order.id);
}

// GOOD
const orders = await db.orders.findAll({
  include: ['items'],
});

// CHECK: Unnecessary work
// Are we computing things we don't need?
// Are we fetching data we don't use?

// BAD
const allUsers = await db.users.findAll();
const user = allUsers.find(u => u.id === id);

// GOOD
const user = await db.users.findById(id);

// CHECK: Memory usage
// Are we loading large datasets into memory?
// Should this use streaming/pagination?
```

### 6. Testing

```typescript
// CHECK: Test coverage
// Are the important paths tested?
// Not just happy path - what about errors?

// CHECK: Test quality
// Do tests verify behavior or implementation?

// BAD: Testing implementation
expect(mockDb.query).toHaveBeenCalledWith('SELECT...');

// GOOD: Testing behavior
expect(result.users).toHaveLength(2);
expect(result.users[0].name).toBe('Alice');

// CHECK: Test isolation
// Do tests depend on each other?
// Do tests clean up after themselves?
```

## Review Comments

### Good Comments

```
// Be specific
"This query lacks an index on user_id, which will cause full table scans.
Consider adding: CREATE INDEX idx_orders_user_id ON orders(user_id)"

// Explain why
"Avoid using any here because it defeats type checking.
Consider defining a proper type or using unknown."

// Offer alternatives
"Instead of nested if statements, consider using early returns:
if (!user) return null;
if (!user.isActive) return null;
return user.profile;"

// Ask questions
"What happens if the API returns an empty array here?
Should we handle that case differently?"
```

### Avoid

```
// Too vague
"This is confusing"  // What specifically is confusing?

// Style nitpicks without tooling
"Missing semicolon"  // Should be caught by linter

// Personal preference without reasoning
"I would have done it differently"  // How? Why?

// Condescending
"Obviously this is wrong"  // Just explain the issue
```

## Review Process

### Before Reviewing

1. Understand the context (read PR description, linked issues)
2. Check if tests pass
3. Review the scope (is this PR doing too much?)

### During Review

1. First pass: understand the overall change
2. Second pass: detailed line-by-line review
3. Check tests match implementation

### After Review

- Approve if good
- Request changes if blocking issues
- Comment if minor suggestions only

## Self-Review Checklist

Before submitting code for review:

- [ ] I've tested this locally
- [ ] Tests pass
- [ ] I've reviewed my own diff
- [ ] Variable names are clear
- [ ] No debugging code left in
- [ ] Error handling is appropriate
- [ ] No sensitive data exposed
- [ ] PR description explains what and why
