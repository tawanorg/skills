---
title: Depend on Abstractions, Not Concretions
impact: HIGH
impactDescription: enables testing and flexibility
tags: architecture, solid, dependency-injection
---

## Depend on Abstractions, Not Concretions

High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Incorrect:**

```typescript
// Direct dependency on concrete implementation
class OrderService {
  private emailService = new SmtpEmailService();  // Tight coupling
  private db = new PostgresDatabase();           // Can't swap or mock

  async placeOrder(order: Order) {
    await this.db.save(order);
    await this.emailService.send(order.userEmail, 'Order confirmed');
  }
}
```

**Correct:**

```typescript
// Depend on interfaces, inject implementations
interface EmailService {
  send(to: string, message: string): Promise<void>;
}

interface OrderRepository {
  save(order: Order): Promise<void>;
}

class OrderService {
  constructor(
    private emailService: EmailService,      // Injected abstraction
    private orderRepository: OrderRepository  // Injected abstraction
  ) {}

  async placeOrder(order: Order) {
    await this.orderRepository.save(order);
    await this.emailService.send(order.userEmail, 'Order confirmed');
  }
}

// Easy to test with mocks
const mockEmail: EmailService = { send: jest.fn() };
const service = new OrderService(mockEmail, mockRepo);
```

**Why:** Concrete dependencies make code untestable and impossible to change without breaking things.

Reference: `references/architecture-patterns.md`
