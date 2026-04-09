# Skill: Clean Architecture

## Overview

Clean Architecture is a software design philosophy that emphasizes separation of concerns and dependency direction. The core principle is that code dependencies should only point inward — toward higher-level policies.

## Core Principles

### 1. Independence of Frameworks
- Architecture should not depend on the existence of any particular framework
- Frameworks are tools, not constraints
- Allows testing without infrastructure

### 2. Independence of UI
- Business logic should be independent of UI
- UI can change without changing business rules
- Different UIs can use same logic

### 3. Independence of Database
- Business rules should not depend on database
- Repository pattern abstracts data access
- Swappable persistence

### 4. Testability
- Business rules can be tested without UI, DB, or external systems
- Test dependencies point inward

## Layer Structure

```
src/
├── domain/           # Enterprise business rules
│   ├── entities/     # Business objects
│   ├── valueObjects/ # Immutable objects
│   ├── repositories/ # Repository interfaces
│   └── services/     # Domain services
│
├── application/      # Application business rules
│   ├── useCases/     # Application-specific business rules
│   ├── interfaces/  # Port interfaces
│   ├── dtos/         # Data transfer objects
│   └── services/     # Application services
│
├── infrastructure/   # Frameworks & drivers
│   ├── persistence/  # Database implementations
│   ├── external/     # External service clients
│   ├── config/       # Configuration
│   └── api/          # API implementations
│
└── presentation/     # Interface adapters
    ├── controllers/  # API controllers
    ├── views/        # UI components
    └── handlers/     # Event handlers
```

## Dependency Rule

```
domain ← application ← infrastructure ← presentation
```

- Domain: Has NO dependencies
- Application: Depends on domain
- Infrastructure: Depends on application and domain
- Presentation: Depends on application

## Key Patterns

### Repository Pattern
```typescript
// Domain defines interface (no implementation)
interface IOrderRepository {
  findById(id: string): Promise<Order>;
  save(order: Order): Promise<void>;
}

// Infrastructure implements
class OrderRepository implements IOrderRepository {
  async findById(id: string): Promise<Order> {
    // DB implementation
  }
}
```

### Use Case Pattern
```typescript
class CreateOrderUseCase {
  constructor(
    private orderRepo: IOrderRepository,
    private eventBus: IEventBus
  ) {}

  async execute(dto: CreateOrderDTO): Promise<Order> {
    // Orchestrate domain logic
    // Use repositories
    // Publish events
  }
}
```

### Dependency Injection
```typescript
// Composition root
const orderRepo = new OrderRepository(dbConnection);
const eventBus = new RabbitMQEventBus();
const createOrderUseCase = new CreateOrderUseCase(orderRepo, eventBus);
```

## Anti-Patterns

### 1. Anemic Domain Model
```typescript
// BAD: Just data, no behavior
class Order {
  id: string;
  items: OrderItem[];
  total: number;
}

// GOOD: Behavior with data
class Order {
  private _items: OrderItem[] = [];
  
  get items() { return [...this._items]; }
  
  addItem(item: OrderItem): void {
    this._items.push(item);
    this.recalculateTotal();
  }
  
  private recalculateTotal(): void {
    this._total = this._items.reduce((sum, i) => sum + i.price, 0);
  }
}
```

### 2. Anemia in Application Layer
```typescript
// BAD: Just delegation
class CreateOrderController {
  async handle(req, res) {
    const orderRepo = new DatabaseOrderRepository();
    await orderRepo.save(req.body);
    res.json({ success: true });
  }
}

// GOOD: Use case orchestration
class CreateOrderController {
  constructor(private createOrderUseCase: CreateOrderUseCase) {}
  
  async handle(req, res) {
    const order = await this.createOrderUseCase.execute(req.body);
    res.json(order);
  }
}
```

### 3. Infrastructure in Domain
```typescript
// BAD: Domain depends on ORM
import { Entity, Column } from 'typeorm';

@Entity()
export class Order {
  @Column()
  id: string;
}

// GOOD: Plain domain object
export class Order {
  id: string;
  items: OrderItem[];
  
  constructor(data: Partial<Order>) {
    Object.assign(this, data);
  }
}
```

### 4. Concrete Dependencies in Use Cases
```typescript
// BAD: Direct database import
class CreateOrderUseCase {
  async execute(dto) {
    const db = new Database();
    await db.orders.insert(dto);
  }
}

// GOOD: Interface dependency
class CreateOrderUseCase {
  constructor(private orderRepo: IOrderRepository) {}
  
  async execute(dto) {
    await this.orderRepo.save(dto);
  }
}
```

## Examples

### Value Object
```typescript
class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {}
  
  add(other: Money): Money {
    if (other.currency !== this.currency) {
      throw new CurrencyMismatchError();
    }
    return new Money(this.amount + other.amount, this.currency);
  }
}
```

### Entity with Identity
```typescript
class Order {
  public readonly id: OrderId;
  private _status: OrderStatus;
  
  constructor(id: OrderId, items: OrderItem[]) {
    this.id = id;
    this._items = items;
    this._status = OrderStatus.PENDING;
  }
  
  get status(): OrderStatus {
    return this._status;
  }
  
  confirm(): void {
    if (this._status !== OrderStatus.PENDING) {
      throw new InvalidStateError('Cannot confirm non-pending order');
    }
    this._status = OrderStatus.CONFIRMED;
  }
}
```

## Testing with Clean Architecture

```typescript
// Unit test - no external dependencies
describe('Order', () => {
  it('should calculate total correctly', () => {
    const order = new Order(
      new OrderId('1'),
      [
        new OrderItem('Item 1', 100),
        new OrderItem('Item 2', 200)
      ]
    );
    
    expect(order.total.amount).toBe(300);
  });
});

// Integration test - real infrastructure
describe('OrderRepository', () => {
  it('should persist and retrieve order', async () => {
    const repo = new OrderRepository(testDb);
    const order = new Order(/* ... */);
    
    await repo.save(order);
    const found = await repo.findById(order.id);
    
    expect(found).toEqual(order);
  });
});
```

## Summary

Clean Architecture creates:
1. **Testable** — Business logic testable without infrastructure
2. **Maintainable** — Changes isolated to appropriate layers
3. **Independent** — Framework, UI, and DB independence
4. **Clear** — Obvious where things belong

Remember: Dependencies point inward. Nothing in an inner layer knows about outer layers.