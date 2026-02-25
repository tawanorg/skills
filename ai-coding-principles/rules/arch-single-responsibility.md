---
title: Single Responsibility Principle
impact: HIGH
impactDescription: prevents god classes and tangled dependencies
tags: architecture, solid, design
---

## Single Responsibility Principle

Each class or module should have one reason to change. Don't mix unrelated responsibilities.

**Incorrect:**

```typescript
// God class with multiple responsibilities
class UserService {
  async createUser(data: UserData) { /* database logic */ }
  async sendWelcomeEmail(user: User) { /* email logic */ }
  async generateReport(users: User[]) { /* reporting logic */ }
  async validateCreditCard(card: string) { /* payment logic */ }
  async uploadAvatar(file: File) { /* file storage logic */ }
}
```

**Correct:**

```typescript
// Each class has one responsibility
class UserService {
  constructor(private userRepository: UserRepository, private eventBus: EventBus) {}

  async createUser(data: UserData): Promise<User> {
    const user = await this.userRepository.create(data);
    this.eventBus.publish(new UserCreatedEvent(user));  // Others handle side effects
    return user;
  }
}

class EmailService {
  async sendWelcomeEmail(user: User) { /* only email logic */ }
}

class ReportService {
  async generateUserReport(users: User[]) { /* only report logic */ }
}

class PaymentService {
  async validateCard(card: string) { /* only payment logic */ }
}
```

**Why:** Mixed responsibilities create classes that change for unrelated reasons and are hard to test.

Reference: `references/architecture-patterns.md`
