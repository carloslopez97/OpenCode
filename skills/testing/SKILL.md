# Skill: Testing Strategy

## Overview

A comprehensive testing strategy ensures code quality, prevents regressions, and enables confident refactoring. Different test types serve different purposes.

## Testing Pyramid

```
        /\
       /E2E\        Few, slow, expensive
      /------\
     /Integration\  Medium quantity
    /--------------
   /   Unit Tests  Many, fast, cheap
  /----------------
```

- **Unit Tests**: 70% — Test individual functions/classes
- **Integration Tests**: 20% — Test component interactions
- **E2E Tests**: 10% — Test complete user journeys

## Test Types

### Unit Tests

Purpose: Verify individual building blocks in isolation

```typescript
describe('Money', () => {
  describe('add', () => {
    it('should add two money values with same currency', () => {
      const ten = new Money(10, 'USD');
      const twenty = new Money(20, 'USD');
      
      const result = ten.add(twenty);
      
      expect(result.amount).toBe(30);
    });
    
    it('should throw when adding different currencies', () => {
      const usd = new Money(10, 'USD');
      const eur = new Money(10, 'EUR');
      
      expect(() => usd.add(eur)).toThrow(CurrencyMismatchError);
    });
  });
});
```

### Integration Tests

Purpose: Verify components work together

```typescript
describe('OrderRepository', () => {
  let repository: OrderRepository;
  let testDb: TestDatabase;
  
  beforeEach(async () => {
    testDb = await setupTestDatabase();
    repository = new OrderRepository(testDb.connection);
  });
  
  it('should persist and retrieve order', async () => {
    const order = new Order({
      id: new OrderId('order-1'),
      customerId: new CustomerId('cust-1'),
      items: [new LineItem('Product A', 100)],
      status: OrderStatus.Pending
    });
    
    await repository.save(order);
    const found = await repository.findById(order.id);
    
    expect(found?.id).toEqual(order.id);
    expect(found?.status).toBe(OrderStatus.Pending);
  });
});
```

### E2E Tests

Purpose: Verify complete user workflows

```typescript
import { test, expect } from '@playwright/test';

test('customer can place order', async ({ page }) => {
  await page.goto('/orders/new');
  
  // Add item to cart
  await page.click('[data-testid="add-product-A"]');
  
  // Fill shipping info
  await page.fill('[name="address"]', '123 Main St');
  
  // Submit order
  await page.click('[data-testid="submit-order"]');
  
  // Verify confirmation
  await expect(page.locator('[data-testid="order-confirmed"]')).toBeVisible();
  await expect(page.locator('[data-testid="order-number"]')).toContainText('ORD-');
});
```

## Test Organization

### By Layer

```
tests/
├── unit/
│   ├── domain/
│   │   ├── entities/
│   │   │   └── Order.test.ts
│   │   └── valueObjects/
│   │       └── Money.test.ts
│   ├── application/
│   │   └── useCases/
│   │       └── CreateOrderUseCase.test.ts
│   └── infrastructure/
│       └── persistence/
│           └── OrderRepository.test.ts
├── integration/
│   ├── api/
│   │   └── orders.test.ts
│   └── services/
│       └── PaymentService.test.ts
└── e2e/
    ├── checkout/
    │   └── placeOrder.test.ts
    └── user/
        └── registration.test.ts
```

### By Feature

```
tests/
├── features/
│   ├── orders/
│   │   ├── create/
│   │   ├── update/
│   │   └── cancel/
│   └── products/
│       ├── list/
│       └── detail/
└── smoke/
    └── criticalPaths.test.ts
```

## Writing Guidelines

### AAA Pattern

```typescript
describe('calculateTotal', () => {
  it('should sum item prices', () => {
    // Arrange
    const items = [
      new LineItem('Item 1', 100),
      new LineItem('Item 2', 200)
    ];
    
    // Act
    const total = calculateTotal(items);
    
    // Assert
    expect(total).toBe(300);
  });
});
```

### Test Data Builders

```typescript
class OrderBuilder {
  private data = {
    id: 'order-1',
    customerId: 'cust-1',
    items: [],
    status: OrderStatus.Pending
  };
  
  withId(id: string): this {
    this.data.id = id;
    return this;
  }
  
  withItems(items: LineItem[]): this {
    this.data.items = items;
    return this;
  }
  
  build(): Order {
    return new Order(this.data);
  }
}

// Usage
const order = new OrderBuilder()
  .withId('order-123')
  .withItems([new LineItem('Product', 100)])
  .build();
```

### Descriptive Names

```typescript
// BAD
test('test1', () => { /* ... */ });
test('create', () => { /* ... */ });

// GOOD
test('should throw ValidationError when email is invalid', () => { /* ... */ });
test('should create order with pending status', () => { /* ... */ });
```

## Test Coverage

### What to Cover

- Business logic (domain layer)
- Use cases and application services
- Edge cases and error paths
- Critical user flows

### What NOT to Cover

- Generated code
- Trivial getters/setters
- Simple data transformations
- Third-party libraries

### Targets

| Layer | Target |
|-------|--------|
| Domain | 90%+ |
| Application | 80%+ |
| Infrastructure | 60%+ |
| Overall | 70%+ |

## Mocking Strategy

### Unit Tests — Mock Everything

```typescript
const mockOrderRepo = {
  save: jest.fn().mockResolvedValue(undefined),
  findById: jest.fn().mockResolvedValue(null)
};

const useCase = new CreateOrderUseCase(mockOrderRepo, mockEventBus);
```

### Integration Tests — Mock External Services

```typescript
// Real DB, mocked external API
const repository = new OrderRepository(realDb);
const mockPaymentApi = jest.fn().mockResolvedValue({ success: true });
```

### E2E Tests — No Mocks

```typescript
// Real everything
test('full checkout flow', async ({ page }) => {
  // Uses real backend, real DB, real services
});
```

## Anti-Patterns

### 1. Testing Implementation Details

```typescript
// BAD - Test internal state
it('should set internal flag to true', () => {
  const processor = new OrderProcessor();
  processor.process(order);
  expect((processor as any).processed).toBe(true);
});

// GOOD - Test observable behavior
it('should mark order as processed', () => {
  const processor = new OrderProcessor();
  const result = processor.process(order);
  
  expect(result.status).toBe(OrderStatus.Processed);
});
```

### 2. Multiple Assertions on Unrelated Things

```typescript
// BAD
it('should create order', () => {
  const order = createOrder({ items: [...] });
  
  expect(order.id).toBeDefined();
  expect(order.createdAt).toBeNow();
  expect(order.items.length).toBe(1);
  expect(order.total.amount).toBe(100);
  expect(order.status).toBe(OrderStatus.Pending);
});

// GOOD - Separate tests
it('should generate unique id', () => { /* ... */ });
it('should set createdAt to current time', () => { /* ... */ });
it('should calculate total from items', () => { /* ... */ });
```

### 3. Flaky Tests

```typescript
// BAD - Time-dependent
it('should process within 100ms', () => {
  const start = Date.now();
  process(data);
  expect(Date.now() - start).toBeLessThan(100);
});

// GOOD - Deterministic
it('should process large dataset', () => {
  const result = process(largeDataset);
  expect(result).toMatchSnapshot();
});
```

## Test-Driven Development (TDD)

1. **Red** — Write failing test
2. **Green** — Write minimal code to pass
3. **Refactor** — Improve code while keeping tests passing

```typescript
// Step 1: Red - Write failing test
describe('Money', () => {
  it('should format as currency string', () => {
    const money = new Money(1234.56, 'USD');
    expect(money.format()).toBe('$1,234.56');
  });
});

// Step 2: Green - Minimal implementation
class Money {
  format(): string {
    return `$${this.amount}`;
  }
}

// Step 3: Refactor - Proper formatting
class Money {
  format(): string {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: this.currency
    }).format(this.amount);
  }
}
```

## Summary

- **Unit tests** — Fast, isolated, many
- **Integration tests** — Test interactions
- **E2E tests** — Verify user journeys
- **AAA pattern** — Arrange, Act, Assert
- **Descriptive names** — Clear intent
- **Good coverage** — Focus on business logic
- **No mocks in E2E** — Test real behavior