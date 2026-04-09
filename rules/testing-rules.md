# Testing Rules

## Purpose

These rules ensure comprehensive, maintainable test coverage.

## Rule 1: Test Coverage Targets

**Requirement**: Minimum coverage levels.

| Layer | Target |
|-------|--------|
| Domain | 90%+ |
| Application | 80%+ |
| Infrastructure | 60%+ |
| Overall | 70%+ |

**Implementation**:
- Run coverage reports on every build
- Set coverage thresholds in CI
- Flag coverage drops

## Rule 2: Test Naming

**Requirement**: Descriptive, intent-revealing names.

**Implementation**:
```typescript
// BAD
test('test1', () => { });
test('create', () => { });

// GOOD - Describes expected behavior
test('should throw ValidationError when email format is invalid', () => { });
test('should create order with pending status when valid', () => { });
test('should reject order when inventory insufficient', () => { });
```

## Rule 3: One Assertion Per Test

**Requirement**: Each test verifies one behavior.

**Implementation**:
```typescript
// BAD - Multiple assertions
test('should create order', () => {
  const order = createOrder(validData);
  expect(order.id).toBeDefined();
  expect(order.status).toBe('pending');
  expect(order.items.length).toBe(1);
  expect(order.total).toBe(100);
});

// GOOD - Separate tests
test('should generate unique id', () => { /* ... */ });
test('should set status to pending', () => { /* ... */ });
test('should include items in order', () => { /* ... */ });
test('should calculate total from items', () => { /* ... */ });
```

## Rule 4: AAA Pattern

**Requirement**: Structure tests with Arrange-Act-Assert.

**Implementation**:
```typescript
describe('calculateTotal', () => {
  it('should sum item prices', () => {
    // Arrange
    const items = [
      new OrderItem('Item 1', 100),
      new OrderItem('Item 2', 200)
    ];
    
    // Act
    const total = calculateTotal(items);
    
    // Assert
    expect(total).toBe(300);
  });
});
```

## Rule 5: Test Data Builders

**Requirement**: Use builders for complex test data.

**Implementation**:
```typescript
// BAD - Inline data
const order = {
  id: 'order-1',
  customerId: 'cust-1',
  items: [
    { productId: 'prod-1', quantity: 2, price: 50 }
  ],
  status: 'pending'
};

// GOOD - Builder pattern
class OrderBuilder {
  private data = {
    id: 'order-1',
    customerId: 'cust-1',
    items: [],
    status: OrderStatus.Pending
  };
  
  withItems(items: OrderItem[]): this {
    this.data.items = items;
    return this;
  }
  
  withStatus(status: OrderStatus): this {
    this.data.status = status;
    return this;
  }
  
  build(): Order {
    return new Order(this.data);
  }
}

const order = new OrderBuilder()
  .withItems([new OrderItem('prod-1', 2, 50)])
  .withStatus(OrderStatus.Pending)
  .build();
```

## Rule 6: Test Doubles Strategy

**Requirement**: Appropriate mocking per test type.

**Implementation**:
```typescript
// Unit tests - Mock everything
const mockRepo = {
  save: jest.fn().mockResolvedValue(undefined),
  findById: jest.fn().mockResolvedValue(order)
};

// Integration tests - Mock external services only
const repository = new OrderRepository(realDb);
jest.mock('../external/PaymentService', () => ({
  PaymentService: jest.fn().mockImplementation(() => ({
    process: jest.fn().mockResolvedValue({ success: true })
  }))
}));

// E2E tests - No mocks
// Uses real backend, real DB, real services
```

## Rule 7: Test Behavior, Not Implementation

**Requirement**: Test observable behavior, not internal state.

**Implementation**:
```typescript
// BAD - Tests implementation details
it('should set processed flag', () => {
  const processor = new OrderProcessor();
  processor.process(order);
  expect((processor as any).processed).toBe(true);
});

// GOOD - Tests behavior
it('should mark order as processed', () => {
  const result = processor.process(order);
  expect(result.status).toBe(OrderStatus.Processed);
});
```

## Rule 8: Edge Cases

**Requirement**: Cover happy path AND edge cases.

**Implementation**:
```typescript
describe('calculateDiscount', () => {
  it('should apply 10% discount for orders over $100', () => { /* ... */ });
  it('should apply 20% discount for orders over $500', () => { /* ... */ });
  
  // Edge cases
  it('should return zero for $0 order', () => { /* ... */ });
  it('should return zero for negative amount', () => { /* ... */ });
  it('should handle maximum discount correctly', () => { /* ... */ });
});
```

## Rule 9: Test Isolation

**Requirement**: Tests must not depend on each other.

**Implementation**:
- Each test sets up its own data
- Clean up after tests
- No shared state between tests

```typescript
beforeEach(async () => {
  await testDb.cleanup();
});

afterEach(async () => {
  await testDb.cleanup();
});
```

## Rule 10: Fast Tests

**Requirement**: Tests must be fast.

**Implementation**:
- Unit tests: < 100ms
- Integration tests: < 500ms
- No sleep() in tests
- Use test databases, not real ones

```typescript
// BAD - Slow test
test('should process order', async () => {
  await processOrder(order);
  await new Promise(r => setTimeout(r, 1000)); // Artificial delay
  expect(getOrderStatus(order.id)).toBe('processed');
});

// GOOD - Fast test
test('should process order', async () => {
  await processOrder(order);
  const status = await getOrderStatus(order.id);
  expect(status).toBe('processed');
});
```

## Rule 11: Maintainable Tests

**Requirement**: Tests must be as maintainable as production code.

**Implementation**:
- Extract common test utilities
- Use constants
- Group related tests with describe blocks
- Keep tests focused

## Enforcement

- Coverage thresholds in CI
- Test execution time limits
- Code review for test quality
- Flaky test detection