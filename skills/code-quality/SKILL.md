# Skill: Code Quality

## Overview

Code quality determines maintainability, readability, and reliability. High-quality code is easier to understand, extend, and debug.

## Principles

### 1. Readability
Code is read more than written. Optimize for human understanding.

```typescript
// BAD
const x = d.map(i => i.t).filter(i => i > 0);

// GOOD - Descriptive names and clear intent
const activeUsers = users
  .map(user => user.name)
  .filter(name => name.length > 0);
```

### 2. Simplicity
Prefer simple solutions. Complex code is error-prone.

```typescript
// BAD - Over-engineered
class OrderProcessor {
  private static instance: OrderProcessor;
  
  private constructor(private strategy: ProcessingStrategy) {}
  
  static getInstance(strategy?: ProcessingStrategy): OrderProcessor {
    if (!OrderProcessor.instance) {
      OrderProcessor.instance = new OrderProcessor(
        strategy || new DefaultStrategy()
      );
    }
    return OrderProcessor.instance;
  }
}

// GOOD - Simple and direct
function processOrder(order: Order): ProcessedOrder {
  return {
    id: order.id,
    status: calculateStatus(order),
    total: calculateTotal(order.items)
  };
}
```

### 3. Single Responsibility
Each piece of code should do one thing well.

```typescript
// BAD - Multiple responsibilities
class User {
  constructor(
    public name: string,
    public email: string,
    public passwordHash: string
  ) {}
  
  validate(): boolean { /* ... */ }
  save(): void { /* ... */ }
  sendEmail(): void { /* ... */ }
  generateReport(): void { /* ... */ }
}

// GOOD - Separated concerns
class User {
  constructor(
    public readonly id: UserId,
    public readonly name: string,
    public readonly email: Email
  ) {}
}

class UserValidator {
  validate(user: User): ValidationResult { /* ... */ }
}

class UserRepository {
  save(user: User): Promise<void> { /* ... */ }
}
```

### 4. DRY (Don't Repeat Yourself)
Extract common patterns into reusable abstractions.

```typescript
// BAD - Repeated logic
function createOrderValidation(order: Order): ValidationResult {
  if (!order.items || order.items.length === 0) {
    return { valid: false, error: 'No items' };
  }
  if (!order.customerId) {
    return { valid: false, error: 'No customer' };
  }
  if (!order.shippingAddress) {
    return { valid: false, error: 'No address' };
  }
  return { valid: true };
}

function createShipmentValidation(shipment: Shipment): ValidationResult {
  if (!shipment.items || shipment.items.length === 0) {
    return { valid: false, error: 'No items' };
  }
  if (!shipment.customerId) {
    return { valid: false, error: 'No customer' };
  }
  if (!shipment.shippingAddress) {
    return { valid: false, error: 'No address' };
  }
  return { valid: true };
}

// GOOD - Extracted validator
function validateRequiredFields<T>(data: T, fields: (keyof T)[]): ValidationResult {
  for (const field of fields) {
    if (!data[field]) {
      return { valid: false, error: `Missing ${String(field)}` };
    }
  }
  return { valid: true };
}

// Usage
const orderValidation = validateRequiredFields(order, ['items', 'customerId', 'shippingAddress']);
const shipmentValidation = validateRequiredFields(shipment, ['items', 'customerId', 'shippingAddress']);
```

### 5. YAGNI (You Aren't Gonna Need It)
Don't add functionality until necessary.

```typescript
// BAD - Premature abstraction
interface IProcessor<T extends HasId> {
  process(item: T): Result;
  processBatch(items: T[]): Result[];
  validate(item: T): boolean;
  rollback(item: T): void;
  getHistory(itemId: string): History[];
}

class OrderProcessor implements IProcessor<Order> {
  process(item: Order): Result { /* ... */ }
  processBatch(items: Order[]): Result[] { /* ... */ }
  validate(item: Order): boolean { return true; }
  rollback(item: Order): void { /* Not needed yet */ }
  getHistory(itemId: string): History[] { /* Not needed yet */ }
}

// GOOD - Start simple, add when needed
function processOrder(order: Order): Result {
  // Implementation
}
```

## Practices

### Naming

| Bad | Good |
|-----|------|
| `data`, `temp`, `x` | `userData`, `temporaryFile`, `index` |
| `processData()` | `calculateOrderTotal()` |
| `get()` | `getUserById()`, `fetchActiveOrders()` |
| `Manager`, `Handler` | `PaymentProcessor`, `RequestHandler` |

### Functions

- Small (20 lines max)
- Few parameters (3 or fewer)
- No side effects where possible
- Clear intent

```typescript
// BAD
function process(o: any): any {
  // 50 lines of mixed logic
}

// GOOD - Focused functions
function calculateOrderTotal(items: LineItem[]): Money {
  return items.reduce((total, item) => total.add(item.price), Money.zero());
}

function validateOrder(order: Order): ValidationResult {
  if (order.items.length === 0) {
    return { valid: false, error: 'Order has no items' };
  }
  return { valid: true };
}

function createOrder(order: Order): CreatedOrder {
  validateOrder(order);
  return { ...order, id: generateId(), status: 'created' };
}
```

### Error Handling

```typescript
// BAD - Swallowed errors
try {
  await processOrder(order);
} catch (e) {
  // Nothing
}

// GOOD - Explicit handling
try {
  await processOrder(order);
} catch (error) {
  if (error instanceof OrderValidationError) {
    logger.warn('Order validation failed', { orderId: order.id, errors: error.errors });
    throw new ApiError(400, 'Invalid order', error.errors);
  }
  logger.error('Order processing failed', { orderId: order.id, error });
  throw new ApiError(500, 'Internal server error');
}
```

### Comments

- Explain WHY, not WHAT
- Document non-obvious decisions
- Keep up to date or remove

```typescript
// BAD
// Loop through items
for (const item of items) {
  // Check if valid
  if (isValid(item)) {
    // Add to result
    result.push(item);
  }
}

// GOOD
// We filter out inactive items because they cannot be processed
// This aligns with business rule: Only active items are billable
const activeItems = items.filter(item => item.status === ItemStatus.Active);
```

### Code Organization

```typescript
// BAD - Random order
const db = connect();
function getUser(id: string) { /* ... */ }
const config = loadConfig();
function processOrder(order: Order) { /* ... */ }
class UserService { }

// GOOD - Logical grouping
// 1. Imports
// 2. Constants and config
// 3. Interfaces and types
// 4. Domain models
// 5. Services and use cases
// 6. Controllers/handlers
// 7. Initialization
```

## Code Smells

### Long Functions
- Extract to smaller functions
- Each function does one thing

### Primitive Obsession
- Use value objects instead of primitives

### Feature Envy
- If a function uses many fields of another class, it belongs there

### God Classes
- Split into focused classes

### Shotgun Surgery
- One change requires changes in many places

### Parallel Inheritance
- Avoid duplicate hierarchies

## Tools & Practices

1. **Linting** — Catch style issues early (ESLint, TSLint)
2. **Formatting** — Consistent style (Prettier)
3. **Type Checking** — TypeScript, Flow
4. **Code Review** — Peer review before merge
5. **Static Analysis** — SonarQube, CodeClimate

## Summary

High-quality code:
- Is readable and self-documenting
- Has clear intent
- Is simple, not complex
- Is testable
- Is maintainable

Write code for the next person (who might be you in 6 months).