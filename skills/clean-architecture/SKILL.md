# Skill: Clean Architecture

## Overview

Clean architecture separates business rules from frameworks and delivery mechanisms. Dependencies should point toward stable policy, not outward toward tooling details.

## Core Principles

### 1. Stable Core

- domain concepts should not depend on frameworks, databases, or transport layers
- business rules should remain understandable without infrastructure context

### 2. Explicit Boundaries

- application code should orchestrate use cases and workflows
- infrastructure should implement adapters for persistence, messaging, and external services
- presentation should translate external input into application-level actions

### 3. Replaceable Edges

- databases, APIs, queues, and UI frameworks are tools, not the core model
- use contracts at boundaries so infrastructure can change without rewriting business logic

### 4. Testable Design

- core behavior should be testable without external systems
- integration tests should verify adapters and wiring where the contracts matter

## Layer Model

```text
domain <- application <- infrastructure <- presentation
```

Typical responsibilities:

- `domain`: entities, value objects, domain services, core invariants
- `application`: use cases, orchestration, workflow rules, ports
- `infrastructure`: persistence, external clients, configuration, runtime adapters
- `presentation`: controllers, handlers, UI adapters, transport mapping

## Good Boundary Signals

A design is usually on the right track when:

- the domain model reads like the business language
- controllers and handlers stay thin
- persistence logic is isolated behind contracts
- infrastructure can be swapped with minimal change to core logic

## Common Failure Modes

Avoid:

- domain entities importing framework or ORM types
- controllers implementing business rules directly
- use cases constructing infrastructure clients directly
- shared utility layers becoming hidden coupling points
- “simple shortcuts” that leak persistence or transport concerns inward

## Practical Guidance

When designing or reviewing a change:

1. identify the stable business rule
2. identify the boundary where outside systems enter
3. place orchestration in the application layer
4. keep adapters at the edge
5. verify dependency direction still points inward

## Summary

Clean architecture is mainly about protecting the core of the system from churn at the edges. Keep dependencies pointed inward, boundaries explicit, and infrastructure replaceable.
