# Project Context: {project-slug}

> Replace `{project-slug}` with actual project identifier (kebab-case)

## Overview

- **Name**: Project Name
- **Description**: Brief description of what the project does
- **Path**: /absolute/path/to/project
- **Status**: active
- **Created**: YYYY-MM-DD
- **Last Updated**: YYYY-MM-DD

## Goals

What this project aims to achieve:
1. Goal 1
2. Goal 2
3. Goal 3

## Requirements

### Functional Requirements

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-01 | Requirement description | high | pending |
| FR-02 | Another requirement | medium | pending |

### Non-Functional Requirements

- Performance: < 100ms response time
- Availability: 99.9% uptime
- Security: OWASP top 10 compliant

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React, TypeScript |
| Backend | Node.js, Express |
| Database | PostgreSQL |
| Cache | Redis |
| Infrastructure | Docker, AWS |

## Architecture

### High-Level Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │────▶│   Backend   │────▶│  Database   │
└─────────────┘     └─────────────┘     └─────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │   Cache     │
                   └─────────────┘
```

### Architecture Pattern

Clean Architecture with:
- Domain Layer (entities, value objects, repository interfaces)
- Application Layer (use cases, services)
- Infrastructure Layer (persistence, external services)
- Presentation Layer (API controllers, UI components)

## Folder Structure

```
src/
├── domain/
│   ├── entities/
│   ├── valueObjects/
│   ├── repositories/  # Interfaces only
│   └── services/
├── application/
│   ├── useCases/
│   ├── interfaces/
│   ├── dtos/
│   └── services/
├── infrastructure/
│   ├── persistence/
│   ├── external/
│   └── config/
└── presentation/
    ├── controllers/
    └── handlers/
```

## Constraints

- Budget: $X
- Timeline: X months
- Team size: X developers
- Regulatory: GDPR compliant

## Open Questions

| ID | Question | Status | Decision |
|----|----------|--------|----------|
| Q-01 | Should we use GraphQL or REST? | open | TBD |
| Q-02 | Which cloud provider? | open | TBD |

## Decisions

| ID | Decision | Date | Rationale |
|----|----------|------|------------|
| D-01 | Use TypeScript | 2024-01-01 | Type safety, better DX |
| D-02 | Use PostgreSQL | 2024-01-01 | ACID compliance needed |

## Team

| Role | Name | Contact |
|------|------|---------|
| Lead Developer | | |
| Product Owner | | |

## Notes

Any additional context, decisions, or notes go here.

---

*This file is maintained by the orchestrator. Updates are made as decisions are finalized and requirements evolve.*