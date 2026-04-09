# Architecture Rules

## Purpose

These rules enforce clean architecture principles and prevent common architectural anti-patterns.

## Rule 1: Layer Independence

**Requirement**: Domain layer must have no external dependencies.

**Implementation**:
- Domain layer contains only business logic
- No imports from infrastructure, application, or presentation
- Use interfaces for external dependencies

**Anti-pattern**:
```typescript
// BAD - Domain imports infrastructure
import { Entity } from 'typeorm';
import { DatabaseService } from './infrastructure';

@Entity()
export class Order { } // TypeORM in domain
```

**Correct**:
```typescript
// GOOD - Plain domain object
export class Order {
  constructor(
    public readonly id: OrderId,
    public readonly items: OrderItem[]
  ) {}
}
```

## Rule 2: Dependency Direction

**Requirement**: Dependencies must point inward toward domain.

**Implementation**:
- Application layer depends on domain
- Infrastructure depends on application
- Presentation depends on application

**Anti-pattern**:
```typescript
// BAD - Infrastructure in application use case
import { MongoClient } from 'mongodb';

class CreateOrderUseCase {
  async execute(dto) {
    const db = new MongoClient().connect();
    await db.collection('orders').insertOne(dto);
  }
}
```

**Correct**:
```typescript
// GOOD - Interface dependency
class CreateOrderUseCase {
  constructor(private orderRepo: IOrderRepository) {}
  
  async execute(dto) {
    await this.orderRepo.save(dto);
  }
}
```

## Rule 3: No God Classes

**Requirement**: Classes must have single responsibility.

**Implementation**:
- Max 200 lines per class
- Max 10 methods per class
- Clear, focused purpose

**Anti-pattern**:
```typescript
// BAD - God class
class Manager {
  createUser() {}
  deleteUser() {}
  createOrder() {}
  cancelOrder() {}
  processPayment() {}
  sendEmail() {}
  generateReport() {}
  // ... 50 more methods
}
```

**Correct**:
```typescript
// GOOD - Focused classes
class UserService { createUser, deleteUser }
class OrderService { createOrder, cancelOrder }
class PaymentService { processPayment }
class NotificationService { sendEmail }
class ReportingService { generateReport }
```

## Rule 4: Repository Pattern

**Requirement**: Data access must use repository pattern.

**Implementation**:
- Domain defines repository interfaces
- Infrastructure implements repositories
- Use dependency injection

```typescript
// Domain - Interface only
interface IOrderRepository {
  findById(id: OrderId): Promise<Order>;
  save(order: Order): Promise<void>;
  findByCustomer(customerId: CustomerId): Promise<Order[]>;
}

// Infrastructure - Implementation
class OrderRepository implements IOrderRepository {
  constructor(private db: Database) {}
  
  async findById(id: OrderId): Promise<Order> {
    const row = await this.db.query('SELECT * FROM orders WHERE id = ?', [id]);
    return Order.fromRow(row);
  }
}
```

## Rule 5: Use Case Pattern

**Requirement**: Business logic orchestration through use cases.

**Implementation**:
- Each use case is a single operation
- Use case receives input, returns output
- No UI concerns in use case

```typescript
class CreateOrderUseCase {
  constructor(
    private orderRepo: IOrderRepository,
    private eventBus: IEventBus,
    private inventoryService: IInventoryService
  ) {}
  
  async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {
    await this.inventoryService.reserve(input.items);
    const order = Order.create(input);
    await this.orderRepo.save(order);
    await this.eventBus.publish(new OrderCreatedEvent(order));
    return { orderId: order.id };
  }
}
```

## Rule 6: Value Objects

**Requirement**: Use value objects for primitive obsession.

**Implementation**:
- Replace primitive types with meaningful value objects
- Immutable
- Self-validating

**Anti-pattern**:
```typescript
// BAD - Primitives everywhere
function calculateTotal(price: number, quantity: number, currency: string): number {
  // What if price is negative? What currency?
}
```

**Correct**:
```typescript
// GOOD - Value objects
class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {
    if (amount < 0) throw new InvalidMoneyError();
  }
}

function calculateTotal(price: Money, quantity: number): Money {
  return new Money(price.amount * quantity, price.currency);
}
```

## Rule 7: Module Organization

**Requirement**: Clear module boundaries.

**Implementation**:
```
src/
├── domain/           # Business logic, no dependencies
│   ├── entities/
│   ├── valueObjects/
│   ├── repositories/ # Interfaces only
│   └── services/
├── application/     # Use cases, orchestrates domain
│   ├── useCases/
│   ├── interfaces/
│   ├── dtos/
│   └── services/
├── infrastructure/  # External concerns
│   ├── persistence/
│   ├── external/
│   └── config/
└── presentation/    # API, UI
    ├── controllers/
    └── handlers/
```

## Rule 8: Cross-Layer Communication

**Requirement**: Use events for loose coupling.

**Implementation**:
- Use event bus for cross-boundary communication
- Avoid direct dependencies between modules
- Async for non-critical operations

```typescript
// Use event for order completion
class OrderService {
  async completeOrder(orderId: OrderId) {
    await this.orderRepo.updateStatus(orderId, OrderStatus.Completed);
    await this.eventBus.publish(new OrderCompletedEvent(orderId));
  }
}

// Listener in different module
class NotificationHandler {
  async handle(event: OrderCompletedEvent) {
    await this.emailService.sendOrderConfirmation(event.orderId);
  }
}
```

## Enforcement

- Code review must verify architecture compliance
- Architecture tests should validate layer structure
- Automated checks for dependency direction