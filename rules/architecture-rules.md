# Architecture Rules

## Purpose

These rules preserve clear boundaries, dependency direction, and modular change.

## Core Rules

### 1. Keep Dependencies Pointing Inward

- domain code must not depend on infrastructure or presentation concerns
- application code may depend on domain
- infrastructure adapts to domain and application contracts
- presentation should call application-level interfaces, not business internals directly

### 2. Keep Boundaries Explicit

- define clear module ownership
- use interfaces or contracts at external boundaries
- do not hide cross-boundary coupling inside convenience abstractions

### 3. Keep Business Logic Out of Framework Edges

- controllers, handlers, and transport adapters should stay thin
- orchestration belongs in use cases or application services
- business rules belong in domain or application layers

### 4. Avoid Oversized Units

- classes, modules, and services should have one clear reason to change
- split code that mixes unrelated responsibilities
- avoid central coordinators that accumulate unrelated behavior

### 5. Design for Replaceable Infrastructure

- persistence, messaging, caching, and third-party integrations should be swappable behind stable contracts
- infrastructure details must not leak into core domain decisions

### 6. Make Risky Coupling Visible

Call out explicitly when a change introduces:

- new runtime dependencies
- data migration requirements
- cross-module write coordination
- rollout or rollback complexity

## Review Checks

Before considering a design or implementation acceptable, verify:

- dependency direction is still correct
- ownership boundaries remain clear
- important contracts are explicit and testable
- the change can be delivered incrementally
- infrastructure concerns have not leaked into core logic

## Anti-Patterns

Avoid:

- domain code importing framework or persistence libraries
- controllers containing business rules
- direct database access from use-case orchestration without an abstraction boundary
- shared utility layers that become hidden coupling points
- module designs that require coordinated edits everywhere
