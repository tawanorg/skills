# Architecture Patterns

SOLID principles and architectural patterns for enterprise-grade code.

## SOLID Principles

### S - Single Responsibility

**Each class/module should have one reason to change.**

```typescript
// BAD: Multiple responsibilities
class UserService {
  createUser(data: UserData) { /* ... */ }
  sendWelcomeEmail(user: User) { /* ... */ }
  generateReport(users: User[]) { /* ... */ }
  validateCreditCard(card: string) { /* ... */ }
}

// GOOD: Single responsibility each
class UserService {
  constructor(
    private userRepository: UserRepository,
    private eventBus: EventBus
  ) {}

  async createUser(data: UserData): Promise<User> {
    const user = await this.userRepository.create(data);
    this.eventBus.publish(new UserCreatedEvent(user));
    return user;
  }
}

class EmailService {
  async sendWelcomeEmail(user: User) { /* ... */ }
}

class ReportService {
  async generateUserReport(users: User[]) { /* ... */ }
}
```

### O - Open/Closed

**Open for extension, closed for modification.**

```typescript
// BAD: Requires modification to add new payment types
class PaymentProcessor {
  process(payment: Payment) {
    if (payment.type === 'credit') {
      // process credit
    } else if (payment.type === 'debit') {
      // process debit
    } else if (payment.type === 'crypto') {
      // had to modify this class to add crypto
    }
  }
}

// GOOD: Extend without modifying
interface PaymentStrategy {
  process(payment: Payment): Promise<PaymentResult>;
}

class CreditCardStrategy implements PaymentStrategy {
  async process(payment: Payment) { /* ... */ }
}

class CryptoStrategy implements PaymentStrategy {
  async process(payment: Payment) { /* ... */ }
}

class PaymentProcessor {
  constructor(private strategies: Map<string, PaymentStrategy>) {}

  async process(payment: Payment) {
    const strategy = this.strategies.get(payment.type);
    if (!strategy) throw new Error(`Unknown payment type: ${payment.type}`);
    return strategy.process(payment);
  }
}
```

### L - Liskov Substitution

**Subtypes must be substitutable for their base types.**

```typescript
// BAD: Square violates Rectangle contract
class Rectangle {
  constructor(protected width: number, protected height: number) {}

  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  area() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; } // Violates LSP
  setHeight(h: number) { this.width = this.height = h; }
}

// GOOD: Separate abstractions
interface Shape {
  area(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  area() { return this.width * this.height; }
}

class Square implements Shape {
  constructor(private side: number) {}
  area() { return this.side * this.side; }
}
```

### I - Interface Segregation

**Clients shouldn't depend on interfaces they don't use.**

```typescript
// BAD: Fat interface
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
  attendMeeting(): void;
}

// Robots can't eat or sleep
class Robot implements Worker {
  work() { /* ... */ }
  eat() { throw new Error('Robots cannot eat'); } // Violation
  sleep() { throw new Error('Robots cannot sleep'); }
  attendMeeting() { /* ... */ }
}

// GOOD: Segregated interfaces
interface Workable {
  work(): void;
}

interface Feedable {
  eat(): void;
}

interface Restable {
  sleep(): void;
}

class Human implements Workable, Feedable, Restable {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

class Robot implements Workable {
  work() { /* ... */ }
}
```

### D - Dependency Inversion

**Depend on abstractions, not concretions.**

```typescript
// BAD: Direct dependency on implementation
class OrderService {
  private emailService = new SmtpEmailService(); // Tight coupling

  async placeOrder(order: Order) {
    await this.saveOrder(order);
    await this.emailService.send(order.userEmail, 'Order confirmed');
  }
}

// GOOD: Depend on abstraction
interface EmailService {
  send(to: string, message: string): Promise<void>;
}

class OrderService {
  constructor(private emailService: EmailService) {} // Injected

  async placeOrder(order: Order) {
    await this.saveOrder(order);
    await this.emailService.send(order.userEmail, 'Order confirmed');
  }
}

// Easy to test with mock
class MockEmailService implements EmailService {
  sent: Array<{ to: string; message: string }> = [];
  async send(to: string, message: string) {
    this.sent.push({ to, message });
  }
}
```

## Layered Architecture

### Clean Architecture Layers

```
┌─────────────────────────────────────┐
│           Presentation              │  Controllers, Views, DTOs
├─────────────────────────────────────┤
│           Application               │  Use Cases, Services
├─────────────────────────────────────┤
│             Domain                  │  Entities, Value Objects, Domain Services
├─────────────────────────────────────┤
│          Infrastructure             │  Repositories, External APIs, Database
└─────────────────────────────────────┘

Dependencies point INWARD only.
Domain knows nothing about infrastructure.
```

### Layer Responsibilities

| Layer | Contains | Depends On |
|-------|----------|------------|
| Domain | Entities, Value Objects, Domain Events | Nothing |
| Application | Use Cases, DTOs, Interfaces | Domain |
| Infrastructure | Repositories, API Clients, ORM | Domain, Application |
| Presentation | Controllers, Validators | Application |

### Example Structure

```
src/
├── domain/
│   ├── entities/
│   │   └── User.ts
│   ├── value-objects/
│   │   └── Email.ts
│   └── events/
│       └── UserCreated.ts
├── application/
│   ├── use-cases/
│   │   └── CreateUser.ts
│   ├── interfaces/
│   │   └── UserRepository.ts
│   └── dtos/
│       └── CreateUserDto.ts
├── infrastructure/
│   ├── repositories/
│   │   └── PostgresUserRepository.ts
│   └── services/
│       └── SendGridEmailService.ts
└── presentation/
    └── controllers/
        └── UserController.ts
```

## Domain-Driven Design Essentials

### Entities vs Value Objects

```typescript
// Entity: Has identity, mutable
class User {
  constructor(
    public readonly id: UserId,  // Identity
    private email: Email,
    private name: string
  ) {}

  changeName(newName: string) {
    this.name = newName;
  }
}

// Value Object: No identity, immutable, equality by value
class Email {
  private constructor(private readonly value: string) {}

  static create(email: string): Email {
    if (!this.isValid(email)) {
      throw new InvalidEmailError(email);
    }
    return new Email(email.toLowerCase());
  }

  private static isValid(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }

  toString(): string {
    return this.value;
  }
}
```

### Aggregates

```typescript
// Aggregate Root: Order controls its OrderItems
class Order {
  private items: OrderItem[] = [];

  addItem(product: Product, quantity: number) {
    const existing = this.items.find(i => i.productId === product.id);
    if (existing) {
      existing.increaseQuantity(quantity);
    } else {
      this.items.push(new OrderItem(product.id, product.price, quantity));
    }
  }

  removeItem(productId: ProductId) {
    this.items = this.items.filter(i => i.productId !== productId);
  }

  get total(): Money {
    return this.items.reduce(
      (sum, item) => sum.add(item.subtotal),
      Money.zero()
    );
  }
}

// Never modify OrderItem directly from outside Order
```

## Architecture Checklist

### Before Writing Code

- [ ] Which layer does this code belong to?
- [ ] What are the dependencies? Do they point inward?
- [ ] Is there an existing pattern for similar functionality?
- [ ] Does this violate any SOLID principles?

### Code Placement

| If you're writing... | It belongs in... |
|---------------------|------------------|
| Business rules | Domain |
| Orchestration logic | Application (Use Case) |
| Database queries | Infrastructure (Repository) |
| HTTP handling | Presentation (Controller) |
| External API calls | Infrastructure (Service) |
