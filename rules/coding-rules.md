# Coding Rules

## Purpose

These rules keep implementation readable, reliable, and easy to review.

## Core Rules

### 1. Prefer Clarity Over Cleverness

- choose intent-revealing names
- prefer straightforward control flow
- avoid abstractions that save little and hide a lot

### 2. Keep Units Focused

- functions, classes, and modules should have one clear purpose
- split logic that mixes validation, persistence, orchestration, and presentation
- avoid generic catch-all helpers when a domain-specific name would be clearer

### 3. Handle Errors Explicitly

- do not swallow errors
- validate inputs at boundaries
- return or raise failures in a way that callers can act on
- log operationally significant failures without leaking sensitive data

### 4. Remove Magic Values

- use named constants for important business or control values
- keep environment-specific values in configuration
- document non-obvious defaults through naming or a short comment

### 5. Respect Type and Contract Boundaries

- use concrete types where they improve safety and reviewability
- avoid `any` unless there is no better boundary representation
- make interface expectations explicit at system boundaries

### 6. Keep Code Reviewable

- keep diffs narrow and intentional
- add concise comments only where the why is otherwise unclear
- avoid unnecessary churn in unrelated files

### 7. Be Performance-Aware by Default

- avoid obviously wasteful loops, queries, or re-renders
- note deferred hotspots instead of hiding them
- optimize only when there is evidence or a clear bottleneck risk

## Anti-Patterns

Avoid:

- hidden side effects
- large mixed-responsibility functions
- broad utility modules with vague ownership
- silent defaults for important behavior
- duplication that will clearly diverge
- premature abstraction that adds maintenance cost

## Enforcement

- linting and type checks should catch the mechanical issues
- code review should focus on correctness, clarity, and maintainability
- agents should call out any important verification or design gaps they could not close
