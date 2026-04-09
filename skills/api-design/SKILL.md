# Skill: API Design

## Overview

API design shapes how developers interact with your system. Well-designed APIs are intuitive, consistent, and evolve gracefully.

## Principles

### 1. Resource-Oriented

Design around resources (nouns), not actions (verbs).

```typescript
// BAD - Action-oriented
POST /createOrder
POST /updateOrderStatus
GET /getOrdersByCustomer

// GOOD - Resource-oriented
POST   /orders              // Create order
PATCH  /orders/{id}         // Update order
GET    /orders/{id}         // Get order
GET    /customers/{id}/orders // Get customer's orders
```

### 2. Consistent

Same patterns everywhere.

```typescript
// Consistent response format
{
  "data": { /* ... */ },
  "meta": { /* pagination, etc. */ }
}

// Error format
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": []
  }
}
```

### 3. Predictable

Use standard HTTP semantics.

| Method | Purpose | Idempotent |
|--------|---------|------------|
| GET | Retrieve | Yes |
| POST | Create | No |
| PUT | Replace | Yes |
| PATCH | Partial update | No |
| DELETE | Remove | Yes |

### 4. Versioned

Plan for evolution.

```typescript
// URL versioning
GET /v1/orders
GET /v2/orders

// Header versioning
Accept: application/vnd.myapp.v2+json
```

## REST API Design

### URL Structure

```
/resources
/resources/{id}
/resources/{id}/subresources
```

```typescript
// Collections
GET    /orders          // List orders
POST   /orders          // Create order

// Single resource
GET    /orders/123      // Get order
PATCH  /orders/123      // Update order
DELETE /orders/123      // Delete order

// Related resources
GET    /orders/123/items    // Get order items
POST   /orders/123/items   // Add item to order
```

### Query Parameters

```typescript
// Filtering
GET /orders?status=pending&customerId=456

// Sorting
GET /orders?sort=-createdAt,total

// Pagination
GET /orders?page=2&limit=20

// Fields selection
GET /orders?fields=id,status,total

// Include relations
GET /orders?include=customer,items
```

### Request Body

```typescript
// Create
POST /orders
{
  "customerId": "cust-123",
  "items": [
    { "productId": "prod-1", "quantity": 2 }
  ],
  "shippingAddress": {
    "street": "123 Main St",
    "city": "Boston",
    "state": "MA",
    "zip": "02101"
  }
}

// Update (partial)
PATCH /orders/123
{
  "status": "cancelled",
  "cancellationReason": "Customer request"
}
```

### Response Format

```typescript
// Single resource
GET /orders/123
{
  "data": {
    "id": "ord-123",
    "status": "pending",
    "total": { "amount": 100, "currency": "USD" },
    "createdAt": "2024-01-15T10:30:00Z",
    "links": {
      "self": "/orders/123",
      "customer": "/customers/cust-123",
      "items": "/orders/123/items"
    }
  }
}

// Collection
GET /orders
{
  "data": [
    { "id": "ord-123", "status": "pending", "total": 100 },
    { "id": "ord-124", "status": "completed", "total": 250 }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  },
  "links": {
    "self": "/orders?page=1",
    "next": "/orders?page=2"
  }
}
```

### Status Codes

```typescript
// Success
200 OK              // GET, PUT, PATCH success
201 Created         // POST create success
204 No Content      // DELETE success

// Client Errors
400 Bad Request     // Validation failed
401 Unauthorized    // Not authenticated
403 Forbidden       // Not authorized
404 Not Found       // Resource doesn't exist
409 Conflict        // Business rule violation
422 Unprocessable   // Validation errors

// Server Errors
500 Internal Error  // Server failure
503 Unavailable      // Service unavailable
```

## GraphQL API Design

### Schema Definition

```graphql
type Query {
  order(id: ID!): Order
  orders(
    filter: OrderFilter
    sort: OrderSort
    pagination: Pagination
  ): OrderConnection!
  customer(id: ID!): Customer
}

type Mutation {
  createOrder(input: CreateOrderInput!): CreateOrderPayload!
  updateOrder(id: ID!, input: UpdateOrderInput!): UpdateOrderPayload!
  cancelOrder(id: ID!, reason: String): CancelOrderPayload!
}

type Order {
  id: ID!
  status: OrderStatus!
  total: Money!
  customer: Customer!
  items: [OrderItem!]!
  createdAt: DateTime!
}

type OrderItem {
  product: Product!
  quantity: Int!
  price: Money!
}

input OrderFilter {
  status: OrderStatus
  customerId: ID
  dateRange: DateRange
}

type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
}
```

### Queries

```graphql
# Simple query
query GetOrder {
  order(id: "ord-123") {
    id
    status
    total {
      amount
      currency
    }
  }
}

# With nested data
query GetOrderWithDetails {
  order(id: "ord-123") {
    id
    customer {
      name
      email
    }
    items {
      product {
        name
      }
      quantity
      price {
        amount
      }
    }
  }
}
```

### Mutations

```graphql
mutation CreateOrder {
  createOrder(input: {
    customerId: "cust-123"
    items: [
      { productId: "prod-1", quantity: 2 }
    ]
    shippingAddress: {
      street: "123 Main St"
      city: "Boston"
      state: "MA"
      zip: "02101"
    }
  }) {
    order {
      id
      status
    }
    errors {
      field
      message
    }
  }
}
```

## Error Handling

### REST Errors

```typescript
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request body contains invalid data",
    "details": [
      {
        "field": "items[0].quantity",
        "message": "Must be greater than 0"
      },
      {
        "field": "shippingAddress.zip",
        "message": "Invalid ZIP code format"
      }
    ],
    "requestId": "req-abc-123"
  }
}
```

### GraphQL Errors

```json
{
  "errors": [
    {
      "message": "Quantity must be greater than 0",
      "locations": [{ "line": 3, "column": 10 }],
      "path": ["createOrder", "items", 0, "quantity"],
      "extensions": {
        "code": "VALIDATION_ERROR",
        "field": "quantity"
      }
    }
  ],
  "data": {
    "createOrder": null
  }
}
```

## Versioning Strategy

### URL Versioning (REST)

```typescript
// v1
GET /v1/orders
Response: { id: "123", total: 100 }

// v2
GET /v2/orders
Response: { id: "123", total: { amount: 100, currency: "USD" } }
```

### Header Versioning (GraphQL)

```http
Accept: application/vnd.myapp.v2+json
```

### Deprecation

```typescript
// Add deprecation notice in responses
{
  "warning": "The 'total' field will be changed to object type in v2",
  "data": {
    "total": 100
  }
}

// Document deprecation timeline
```

## Documentation

### OpenAPI Example

```yaml
paths:
  /orders:
    post:
      summary: Create a new order
      tags: [Orders]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        201:
          description: Order created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        400:
          $ref: '#/components/responses/ValidationError'
```

## Summary

- **Resource-oriented** — Nouns, not verbs
- **Consistent** — Same patterns everywhere
- **HTTP semantics** — Proper methods and codes
- **Versioned** — Plan for changes
- **Well-documented** — OpenAPI/Swagger
- **Error handling** — Clear, actionable errors