# Review Rules

## Purpose

These rules define the code review process and ensure consistent quality standards.

## Reviewer Responsibilities

### 1. Pre-Review Checklist

Before reviewing, verify:
- [ ] Code compiles without errors
- [ ] All tests pass
- [ ] Linting passes
- [ ] Type checking passes
- [ ] PR description explains the change

### 2. Review Focus Areas

| Area | What to Check |
|------|---------------|
| Correctness | Does it work as intended? |
| Architecture | Clean architecture compliance |
| Code Quality | Readability, naming, structure |
| Testing | Coverage, test quality |
| Security | Vulnerabilities, data protection |
| Performance | Efficiency, optimization |

## Review Standards

### Critical Issues (Block Merge)

1. **Functional Bugs** - Code doesn't work as specified
2. **Security Vulnerabilities** - SQL injection, XSS, etc.
3. **Architecture Violations** - Domain depends on infrastructure
4. **Missing Tests** - No test coverage for new features

### High Issues (Must Fix)

1. **Code Smells** - Duplication, dead code, god classes
2. **Poor Naming** - Unclear variable/function names
3. **Error Handling** - Swallowed errors, no logging
4. **Memory Leaks** - Unclosed resources, listeners

### Medium Issues (Should Fix)

1. **Code Style** - Inconsistent formatting
2. **Missing Comments** - Non-obvious logic undocumented
3. **Magic Values** - Hardcoded numbers/strings
4. **Test Quality** - Weak assertions, flaky tests

### Low Issues (Consider Fixing)

1. **Type Safety** - Using `any` instead of specific types
2. **Minor Duplication** - Similar code that could be extracted
3. **Console Logging** - Should use proper logger

## Review Feedback Format

```markdown
## Code Review: [PR Title]

### Summary
Brief overview of what was reviewed.

### Issues Found

#### Critical
- [ ] **Issue**: Description
  - **Location**: File:Line
  - **Suggestion**: How to fix

#### High
- [ ] **Issue**: Description
  - **Location**: File:Line
  - **Suggestion**: How to fix

### Recommendations
- Suggestion 1
- Suggestion 2

### What Looks Good
- Positive observation 1
- Positive observation 2

### Decision
- [ ] Approved
- [ ] Changes Requested
- [ ] Blocked
```

## Anti-Patterns to Catch

### 1. Feature Envy
```typescript
// BAD - Using another class's data extensively
class OrderService {
  calculateTotal(order) {
    return order.items.reduce(
      (sum, i) => sum + i.price * i.quantity * (1 - i.discount),
      0
    );
  }
}

// GOOD - Move logic to Order class
class Order {
  get total() {
    return this.items.reduce((sum, i) => sum + i.subtotal, 0);
  }
}
```

### 2. Primitive Obsession
```typescript
// BAD
function sendEmail(to: string, subject: string, body: string) { }

// GOOD
function sendEmail(email: Email) { }
```

### 3. Parallel Inheritance
```typescript
// BAD - Duplicate hierarchies
class Order {}
class OrderRepository {}

class Shipment {}
class ShipmentRepository {}

// Good - Shared base or no inheritance
interface Repository<T> { }
class OrderRepository implements Repository<Order> { }
```

### 4. Message Chains
```typescript
// BAD
const city = order.customer.address.city;

// GOOD
const city = order.shippingAddress.city;
```

### 5. Switch Statements
```typescript
// BAD
switch (order.status) {
  case 'pending': /* ... */ break;
  case 'confirmed': /* ... */ break;
  case 'cancelled': /* ... */ break;
}

// GOOD - Polymorphism or strategy pattern
interface OrderStatusHandler {
  handle(order: Order): void;
}
```

## Security Review

### Must Check

- [ ] Input validation
- [ ] Authentication/authorization
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] CSRF protection
- [ ] No hardcoded secrets

### Common Vulnerabilities

```typescript
// BAD - SQL Injection
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// GOOD - Parameterized query
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [userId]);

// BAD - XSS
element.innerHTML = userInput;

// GOOD - Sanitized or textContent
element.textContent = userInput;
```

## Architecture Review

### Checkpoints

1. **Domain Independence**
   - No infrastructure imports in domain layer
   - Interfaces in domain, implementations in infrastructure

2. **Dependency Direction**
   - Dependencies point toward domain
   - No circular dependencies

3. **Single Responsibility**
   - Classes have one reason to change
   - No god classes

4. **Repository Pattern**
   - Data access through repository interfaces
   - No direct database calls in use cases

## Review Process

1. **Author** - Pre-submit self-review
2. **Automated Checks** - CI passes
3. **Reviewer** - Thorough code review
4. **Discussion** - Address feedback
5. **Approval** - Merge or request changes

## Time Limits

- Small changes: Review within 24 hours
- Medium changes: Review within 48 hours
- Large changes: Review within 72 hours

## Enforcement

- Block merges for critical issues
- Require fixes for high issues
- Track review metrics
- Regular review retrospectives