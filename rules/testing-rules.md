# Testing Rules

## Purpose

These rules keep testing focused on confidence, regression prevention, and maintainability.

## Core Rules

### 1. Match Test Depth to Risk

- use unit tests for deterministic business logic
- use integration tests for contracts, wiring, and persistence behavior
- use end-to-end tests only for critical user or system journeys
- do not add broad test suites when a narrower layer can prove the behavior safely

### 2. Test Behaviors, Not Internals

- each test should prove one behavior, not one assertion
- multiple assertions are acceptable when they validate the same behavior
- avoid testing private state, implementation details, or incidental call order unless that behavior is the contract

### 3. Cover Critical Paths and Edge Cases

Always consider:

- happy path behavior
- boundary conditions
- failure paths
- previously broken behavior that needs regression coverage

### 4. Keep Tests Deterministic

- no arbitrary sleeps or timing-dependent expectations
- isolate shared state between tests
- mock only the boundaries needed to keep the test reliable and focused
- prefer real integrations over mocks when the contract itself is the risk

### 5. Keep Tests Maintainable

- use descriptive names that explain the behavior being verified
- prefer builders or fixtures for complex setup
- keep setup proportional to the behavior under test
- remove redundant tests that do not increase confidence

## Coverage Guidance

Coverage is a signal, not the goal.

Recommended targets:

| Layer | Target |
|-------|--------|
| Domain | 85%+ |
| Application | 75%+ |
| Infrastructure | 50%+ |
| Overall | 65%+ |

Use risk and change impact to decide where deeper coverage is required.

## Verification Expectations

For behavior-changing work, testing should make clear:

- what behavior is now proven
- what test level was used and why
- what meaningful gaps still remain

## Anti-Patterns

Avoid:

- one-test-per-assertion fragmentation
- asserting implementation details instead of outcomes
- excessive mocking that hides integration problems
- flaky timing or environment-coupled tests
- chasing coverage numbers with low-value tests

## Enforcement

- CI should enforce execution of the relevant test suites
- reviewers should assess test adequacy based on risk, not only coverage numbers
- agents must call out residual verification gaps explicitly
