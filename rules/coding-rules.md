# Coding Rules

## Purpose

These rules ensure code quality, readability, and maintainability across all implementations.

## Rule 1: Meaningful Names

**Requirement**: All names must reveal intent.

**Implementation**:
- Variables/functions/classes use descriptive names
- No single-letter names except loop indices
- Boolean variables use `is`, `has`, `should` prefixes

**Examples**:
```typescript
// BAD
const x = orders.filter(o => o.s === 'p');

// GOOD
const pendingOrders = orders.filter(order => order.status === OrderStatus.Pending);
```

## Rule 2: Small Functions

**Requirement**: Functions should do one thing.

**Implementation**:
- Max 20 lines per function
- Max 3 parameters
- No duplication

**Anti-pattern**:
```typescript
// BAD - Does everything
function processOrder(order) {
  // Validate
  if (!order.items.length) return error;
  // Calculate
  order.total = order.items.reduce((sum, i) => sum + i.price, 0);
  // Save
  db.orders.save(order);
  // Email
  email.send(order.customer, 'Order placed');
  // Log
  logger.info('Order processed');
  // Inventory
  inventory.decrement(order.items);
}
```

**Correct**:
```typescript
// GOOD - Focused functions
function validateOrder(order): void {
  if (!order.items.length) throw new ValidationError('No items');
}

function calculateOrderTotal(order): Money {
  return order.items.reduce((total, item) => total.add(item.price), Money.zero());
}

async function processOrder(order): Promise<void> {
  validateOrder(order);
  const total = calculateOrderTotal(order);
  await saveOrder({ ...order, total });
  await notifyCustomer(order);
  await updateInventory(order.items);
}
```

## Rule 3: Error Handling

**Requirement**: All errors must be handled explicitly.

**Implementation**:
- Never swallow errors
- Use specific error types
- Log appropriately

```typescript
// BAD
try {
  await processOrder(order);
} catch (e) {
  // Silent failure
}

// GOOD
try {
  await processOrder(order);
} catch (error) {
  if (error instanceof ValidationError) {
    logger.warn('Order validation failed', { orderId: order.id, reason: error.message });
    throw new ApiError(400, 'Invalid order data', error.details);
  }
  logger.error('Order processing failed', { orderId: order.id, error });
  throw new ApiError(500, 'Internal server error');
}
```

## Rule 4: No Magic Values

**Requirement**: No hardcoded values in code.

**Implementation**:
- Use constants for all fixed values
- Use configuration for environment-specific values
- No magic numbers or strings

```typescript
// BAD
if (user.retryCount > 5) { /* ... */ }
await delay(2000);

// GOOD
const MAX_RETRY_ATTEMPTS = 5;
const RETRY_DELAY_MS = 2000;

if (user.retryCount > MAX_RETRY_ATTEMPTS) { /* ... */ }
await delay(RETRY_DELAY_MS);
```

## Rule 5: Single Responsibility

**Requirement**: Each class/function has one purpose.

**Implementation**:
- Class name should describe its responsibility
- If you can't describe what the class does without using "and", split it

```typescript
// BAD
class UserManager {
  createUser() {}
  validateUser() {}
  sendEmail() {}      // Not user's responsibility
  generateReport() {} // Not user's responsibility
}

// GOOD
class UserService {
  createUser() {}
  validateUser() {}
}

class NotificationService {
  sendEmail() {}
}

class ReportingService {
  generateReport() {}
}
```

## Rule 6: DRY (Don't Repeat Yourself)

**Requirement**: No code duplication.

**Implementation**:
- Extract common patterns
- Use inheritance or composition appropriately
- Create utilities for repeated logic

```typescript
// BAD
function validateOrder(order) {
  if (!order.customerId) throw new Error('No customer');
  if (!order.items.length) throw new Error('No items');
  if (!order.shippingAddress) throw new Error('No address');
}

function validateShipment(shipment) {
  if (!shipment.customerId) throw new Error('No customer');
  if (!shipment.items.length) throw new Error('No items');
  if (!shipment.shippingAddress) throw new Error('No address');
}

// GOOD
function validateRequiredFields<T>(data: T, fields: (keyof T)[]): void {
  for (const field of fields) {
    if (!data[field]) {
      throw new Error(`Missing required field: ${String(field)}`);
    }
  }
}

function validateOrder(order) {
  validateRequiredFields(order, ['customerId', 'items', 'shippingAddress']);
}

function validateShipment(shipment) {
  validateRequiredFields(shipment, ['customerId', 'items', 'shippingAddress']);
}
```

## Rule 7: Comments

**Requirement**: Comments explain WHY, not WHAT.

**Implementation**:
- Code should be self-documenting
- Explain non-obvious decisions
- Document business rules

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
// Only active items can be processed per business rule:
// Inactive items have pending issues that must be resolved first
const activeItems = items.filter(item => item.status === ItemStatus.Active);
```

## Rule 8: TypeScript Usage

**Requirement**: Use TypeScript features fully.

**Implementation**:
- Strict typing
- No `any` type
- Use interfaces for shapes

```typescript
// BAD
function process(data: any): any {
  return data.map((i: any) => i.value);
}

// GOOD
interface OrderItem {
  id: string;
  value: number;
}

function process(items: OrderItem[]): number[] {
  return items.map(item => item.value);
}
```

## Rule 9: Import Organization

**Requirement**: Organized imports.

**Implementation**:
```typescript
// 1. External libraries
import { Router } from 'express';
import { z } from 'zod';

// 2. Internal modules (absolute)
import { OrderService } from '@application/services';
import { Order } from '@domain/entities';

// 3. Relative imports
import { validateOrder } from '../validators';

// 4. Type imports
import type { Request, Response } from 'express';
```

## Rule 10: Async/Await

**Requirement**: Consistent async patterns.

**Implementation**:
- Use async/await over raw promises
- Handle errors with try/catch
- Don't mix patterns

```typescript
// BAD
function getData() {
  return fetch('/api').then(res => res.json());
}

// GOOD
async function getData(): Promise<Data> {
  const response = await fetch('/api');
  if (!response.ok) {
    throw new ApiError(response.status, 'Failed to fetch');
  }
  return response.json();
}
```

## Enforcement

- Linting rules enforce most of these
- Code review validates
- Automated checks in CI