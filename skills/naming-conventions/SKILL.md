# Skill: Naming Conventions

## Overview

Good names are one of the most important aspects of readable code. Names should reveal intent — what the code does and why.

## Principles

### 1. Intent-Revealing

The name should answer why the code exists.

```typescript
// BAD - What is this?
let d: number;
let temp: any;
function process(): void { }

// GOOD - Clear intent
let elapsedDays: number;
let pendingOrder: Order;
function calculateOrderTotal(): void { }
```

### 2. No Disinformation

Don't mislead readers.

```typescript
// BAD - Misleading
const accountList = { "a1": Account }; // It's a map, not a list

// GOOD - Accurate
const accountsById = new Map<string, Account>();
const accountIds = ['a1', 'a2']; // This IS a list
```

### 3. Searchable

Long, descriptive names beat short, ambiguous ones.

```typescript
// BAD - Hard to search
const x = 7;
if (x > 0) { }

// GOOD - Easy to search
const MAX_RETRY_ATTEMPTS = 7;
if (retryCount > MAX_RETRY_ATTEMPTS) { }
```

### 4. Meaningful Distinctions

Different names for different things.

```typescript
// BAD - No distinction
function getData(): Data { }
function getDataById(id: string): Data { }

// GOOD - Clear distinction
function fetchAllData(): Data[] { }
function fetchDataById(id: string): Data { }
```

## Naming by Type

### Variables

```typescript
// Use nouns
const user: User = /* ... */;
const orderTotal: Money = /* ... */;
const isActive: boolean = /* ... */;

// Boolean - prefix with is, has, should, can
const isValid: boolean;
const hasPermission: boolean;
const shouldRetry: boolean;

// Collections - plural or collective
const users: User[] = /* ... */;
const orderItems: OrderItem[] = /* ... */;
```

### Functions

```typescript
// Use verbs or verb phrases
function createOrder(): Order { }
function validateInput(): boolean { }
function calculateTotal(): Money { }

// Get/Set for accessors
function getUserId(): string { }
function setUserName(name: string): void { }

// Query/Mutation for more complex operations
function fetchOrders(): Promise<Order[]> { }
function processPayment(): Promise<PaymentResult> { }
```

### Classes

```typescript
// Use nouns or noun phrases
class OrderProcessor { }
class UserRepository { }
class PaymentService { }

// Avoid generic names
// BAD: Manager, Handler, Helper, Processor (too vague)
// GOOD: Specific purpose
```

### Interfaces

```typescript
// Use nouns or adjectives
interface Repository { }
interface Comparable { }
interface OrderRepository { }
interface CreateOrderInput { }

// Prefix with I for interfaces (TypeScript convention)
interface IRepository { }
interface IOrderRepository { }
// OR use suffix
interface RepositoryInterface { }
interface OrderRepositoryInterface { }
```

### Constants

```typescript
// SCREAMING_SNAKE_CASE for values
const MAX_RETRY_ATTEMPTS = 3;
const DEFAULT_PAGE_SIZE = 20;
const HTTP_STATUS_OK = 200;

// Use meaningful names, not magic numbers
// BAD: if (x > 10) { }
// GOOD: if (attempts > MAX_RETRY_ATTEMPTS) { }
```

### Enums

```typescript
// PascalCase for enum and values
enum OrderStatus {
  Pending = 'pending',
  Confirmed = 'confirmed',
  Shipped = 'shipped',
  Delivered = 'delivered',
  Cancelled = 'cancelled'
}

// TypeScript enums can also use const
const enum PaymentMethod {
  CreditCard = 'credit_card',
  PayPal = 'paypal',
  BankTransfer = 'bank_transfer'
}
```

## Domain-Specific Naming

### Entities

```typescript
class Order {
  readonly id: OrderId;
  private _status: OrderStatus;
  private _items: OrderItem[];
  
  get status(): OrderStatus { return this._status; }
  get items(): ReadonlyArray<OrderItem> { return this._items; }
}
```

### Value Objects

```typescript
class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) { }
}

class Address {
  constructor(
    public readonly street: string,
    public readonly city: string,
    public readonly state: string,
    public readonly zipCode: string
  ) { }
}
```

### Use Cases

```typescript
// [Action][Target]UseCase
class CreateOrderUseCase { }
class CancelOrderUseCase { }
class UpdateInventoryUseCase { }
```

### Repositories

```typescript
// [Entity]Repository
interface IOrderRepository {
  findById(id: OrderId): Promise<Order>;
  save(order: Order): Promise<void>;
  findByCustomer(customerId: CustomerId): Promise<Order[]>;
}

class OrderRepository implements IOrderRepository { }
```

### DTOs

```typescript
// [Action][Target]DTO
class CreateOrderDTO { }
class UpdateOrderDTO { }
class OrderResponseDTO { }
```

## Anti-Patterns

### 1. Single Letters (Except Loops)

```typescript
// BAD
const a = orders.filter(o => o.status === 'active');

// GOOD - Even for short-lived variables
const activeOrders = orders.filter(order => order.status === OrderStatus.Active);

// Acceptable for well-known patterns
for (let i = 0; i < items.length; i++) { }
for (const item of items) { }
```

### 2. Hungarian Notation

```typescript
// Unnecessary with TypeScript
// BAD
const strName: string = 'John';
const bIsActive: boolean = true;

// GOOD
const name: string = 'John';
const isActive: boolean = true;
```

### 3. Noise Words

```typescript
// BAD
const orderInfo: Order; // Info is meaningless
const userData: User;
function processData(): void;

// GOOD
const order: Order;
const user: User;
function processOrder(): void;
```

### 4. Inconsistent Naming

```typescript
// BAD
getUserById()
fetchOrderByID()
retrieveCustomer() // Mixed conventions

// GOOD - Consistent
getUserById()
getOrderById()
getCustomerById()
```

## File Naming

```typescript
// PascalCase for files (TypeScript/React)
Order.ts
OrderProcessor.ts
useOrders.ts
OrderList.tsx

// Kebab-case for config files
tsconfig.json
webpack.config.js

// Descriptive, reflects content
// BAD: utils.ts, helpers.ts, lib.ts
// GOOD: dateUtils.ts, validationHelpers.ts
```

## Summary

- **Reveal intent** — Name explains why
- **Be accurate** — No disinformation
- **Be searchable** — Long > short ambiguous
- **Make distinctions** — Different names for different things
- **Be consistent** — Same patterns everywhere
- **Avoid noise** — No "Info", "Data", "Manager"
- **Use domain terms** — Speak the language of the business