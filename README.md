# OpenCode Configuration

This repository defines an opinionated OpenCode setup for orchestrating multi-agent software delivery. The configuration is centered on one orchestrator, a set of specialist agents, reusable skills, explicit rule packs, and isolated per-project context.

The source of truth is [`opencode.yaml`](./opencode.yaml). The supporting prompt files, rules, skills, and project context live alongside it in this repository.

## Overview

The configuration is designed around a few core ideas:

- The `orchestrator` is the default entry point and coordinates delivery end to end.
- Specialist agents own clearly separated domains such as architecture, backend, frontend, testing, security, and performance.
- Skills provide reusable guidance that can be attached to multiple agents.
- Rules act as quality gates and behavioral constraints.
- Every project has isolated context so decisions do not leak across workstreams.
- Task contracts and response formats are standardized to make delegation safer and easier to validate.

## Repository Structure

```text
.
|-- opencode.yaml
|-- README.md
|-- agents/
|-- commands/
|-- context/
|   |-- registry.md
|   `-- projects/
|-- modes/
|-- rules/
`-- skills/
```

Key folders:

- [`modes/`](./modes) contains mode-specific behavior, including the orchestrator prompt.
- [`agents/`](./agents) contains specialist agent definitions.
- [`skills/`](./skills) contains reusable domain guidance packaged as `SKILL.md` files.
- [`rules/`](./rules) contains shared engineering constraints and review criteria.
- [`context/`](./context) contains the project registry and per-project context documents.
- [`commands/`](./commands) contains command-oriented workflow helpers.

## Configuration Layout

`opencode.yaml` is organized into these major sections:

1. `modes`
2. `agents`
3. `skills`
4. `rules`
5. `context`
6. `orchestrator`
7. `task_contract`
8. `agent_response`
9. `failure_handling`
10. `hard_constraints`
11. `notes`

This structure makes the system readable from top to bottom: who can act, what guidance they receive, what context they load, how work is delegated, and how outcomes are validated.

## Modes

The default mode is `orchestrator`.

Available modes currently include:

- `orchestrator`
- `architect`
- `backend-developer`
- `frontend-developer`
- `reviewer`
- `tester`
- `devops`
- `product-manager`
- `security-engineer`
- `data-engineer`
- `performance-optimizer`

The orchestrator mode is the only one with a shared base rule set attached directly in the `modes.available` section:

- [`architecture-rules.md`](./rules/architecture-rules.md)
- [`coding-rules.md`](./rules/coding-rules.md)
- [`testing-rules.md`](./rules/testing-rules.md)
- [`review-rules.md`](./rules/review-rules.md)

That makes the orchestrator responsible for coordinating delivery with architectural, coding, testing, and review awareness from the start.

## Agents

The configuration defines ten specialist agents. Each agent declares:

- `name`
- `description`
- `mode`
- `responsibilities`
- `rules`
- `context`

### Agent Roster

| Agent | Primary Purpose |
|------|-----------------|
| `architect` | Technical direction, boundaries, interfaces, tradeoffs |
| `backend-developer` | Backend implementation, integrations, reliability |
| `frontend-developer` | UI delivery, state handling, accessibility |
| `reviewer` | Severity-based review and merge-readiness assessment |
| `tester` | Risk-based testing and confidence reporting |
| `devops` | CI/CD, infrastructure, operations, release safety |
| `product-manager` | Scope framing, prioritization, acceptance definition |
| `security-engineer` | Threat review, hardening, auth and data exposure checks |
| `data-engineer` | Data models, pipelines, and data quality |
| `performance-optimizer` | Bottleneck analysis and performance verification |

### Agent Design Principles

The agents are intentionally specialized:

- Ownership is separated to reduce overlap and ambiguity.
- High-risk work can be escalated to dedicated specialists instead of being handled generically.
- Agents inherit only the rules and skills that fit their role.
- Every agent loads shared registry context plus project-specific context through `./context/projects/{project-slug}.md`.

## Skills

Skills are reusable capability packs that can be attached to multiple agents without duplicating prompt logic.

The current configuration defines these skills:

| Skill | Purpose |
|------|---------|
| `clean-architecture` | Layering, boundaries, dependency direction |
| `code-quality` | Maintainability, readability, reviewability |
| `testing` | Test strategy and implementation patterns |
| `api-design` | Contract quality and API design discipline |
| `naming-conventions` | Consistent naming for clarity |

Each skill declares:

- `name`
- `description`
- `rules`
- `available_for`

This allows the same guidance to be shared across multiple modes and agents while keeping the overall configuration modular.

## Rules

Rules are formalized quality constraints that agents must follow. The repository currently defines four rule packs:

| Rule | Intent |
|------|--------|
| `architecture-rules` | Modularity, dependency direction, design integrity |
| `coding-rules` | Implementation discipline and maintainability |
| `testing-rules` | Verification expectations and confidence standards |
| `review-rules` | Review quality gates and severity-based assessment |

Each rule entry specifies:

- `name`
- `description`
- `file`
- `applies_to`

This gives you one place to see which roles are governed by which standards.

## Context Model

Context handling is one of the strongest parts of this configuration.

### Always-Loaded Context

The following file is always loaded:

- [`registry.md`](./context/registry.md)

This acts as the shared project directory for project discovery and selection.

### Project Context

Project-specific context is loaded from:

- `./context/projects/{project-slug}.md`

This is required for all configured roles, including the orchestrator and every specialist agent.

### Context Isolation

The configuration explicitly enables isolation:

- `enabled: true`
- Description: each project maintains isolated context and decisions

### Hydration Rules

When project context is loaded, the system expects it to include:

- project metadata
- technical stack
- architecture decisions
- known risks
- open questions
- pending tasks

The orchestrator rejects incomplete context if core data is missing, specifically:

- project path
- core stack information

## Orchestrator Behavior

The `orchestrator` section defines how work is planned, delegated, validated, and evolved over time.

### Project Detection

The orchestrator is configured to:

- auto-detect the project
- ask when the project is unclear
- create context when a project is missing

### Delegation Policy

Delegation is intentionally strict. The orchestrator requires:

- task contracts
- acceptance criteria
- dependency tracking
- non-goals for risky tasks
- verification evidence
- response validation

### Concurrency Controls

Parallel work is enabled, but bounded:

- `max_concurrent_tasks: 3`
- `max_tasks_per_agent: 1`
- parallelism is only allowed when dependencies are clear
- parallelism is only allowed when ownership is isolated

This is a strong signal that the system favors controlled concurrency over aggressive parallel delegation.

### Planning Behavior

The orchestrator is configured to:

- build a task DAG
- prioritize by risk and value
- force explicit assumptions
- split multi-domain work
- use specialists for high-risk domains

This moves the system closer to principal-engineer-style coordination rather than simple routing.

### Quality Gates

The configuration enables explicit gates for:

- architecture review on high-risk changes
- testing for behavior changes
- review before merge-ready claims
- observability checks for operational changes
- security review for sensitive flows
- performance review for critical paths

### Context Evolution

The orchestrator also updates project knowledge continuously by tracking:

- decisions
- risks
- learnings
- follow-up tasks

## Task Contract Format

The configuration standardizes handoffs through a required task contract.

### Required Fields

```json
[
  "task_id",
  "project",
  "assigned_to",
  "task_type",
  "priority",
  "dependencies",
  "context",
  "input",
  "expected_output",
  "acceptance_criteria"
]
```

### Recommended Context Fields

- `summary`
- `requirements`
- `constraints`
- `decisions`
- `risks`
- `assumptions`

### Recommended Input Fields

- `description`
- `artifacts`
- `non_goals`

### Recommended Expected Output Fields

- `type`
- `format`
- `definition_of_done`
- `verification_evidence`

### Supported Task Types

- `implementation`
- `design`
- `testing`
- `review`
- `devops`
- `product`
- `security`
- `data`
- `performance`

### Supported Priorities

- `low`
- `medium`
- `high`
- `critical`

## Agent Response Format

Agents are expected to return structured output as well.

### Required Response Fields

```json
[
  "task_id",
  "status",
  "summary",
  "output",
  "issues",
  "suggestions",
  "next_steps"
]
```

### Recommended Response Fields

- `verification_notes`
- `risks`
- `assumptions`

### Status Values

- `completed`
- `blocked`
- `needs_clarification`

This creates a predictable contract in both directions: the orchestrator sends structured tasks and receives structured outcomes.

## Failure Handling

The configuration describes expected orchestrator behavior for the three core task outcomes:

- `needs_clarification`: resolve ambiguity and retry with a tighter contract
- `blocked`: isolate the blocker, create an unblock task, and resume safe dependents
- `completed`: validate evidence, integrate output, and update context

There is also explicit behavior for repeated failure:

- reduce scope
- revisit ownership
- change execution strategy

## Hard Constraints

The system enforces these non-negotiable constraints:

- orchestrator never writes code
- agents never modify rules
- no overlapping responsibilities
- everything is modular
- context isolation is enforced
- evidence is required for completion claims

These constraints are the backbone of the overall operating model.

## How To Extend The Configuration

### Add a New Agent

1. Create a new prompt file in [`agents/`](./agents).
2. Add a corresponding entry under `agents` in [`opencode.yaml`](./opencode.yaml).
3. Assign responsibilities, rule packs, and required context deliberately.
4. Only introduce a new agent when it owns a genuinely distinct domain.

### Add a New Skill

1. Create `skills/<skill-name>/SKILL.md`.
2. Add the skill under `skills` in [`opencode.yaml`](./opencode.yaml).
3. Map it through `available_for` instead of duplicating guidance inside many agents.

### Add a New Rule Pack

1. Add the markdown file in [`rules/`](./rules).
2. Register it under `rules` in [`opencode.yaml`](./opencode.yaml).
3. Attach it only to the roles that should actually be governed by it.

### Add a New Project

1. Register it in [`registry.md`](./context/registry.md).
2. Create its project context file in [`context/projects/`](./context/projects).
3. Ensure the context includes project path and core stack information so hydration does not fail.

## Recommended Usage Pattern

In practice, the configuration encourages this flow:

1. The orchestrator identifies the project and loads isolated context.
2. Work is decomposed into explicit, dependency-aware task contracts.
3. Specialists are assigned only the work they uniquely own.
4. Responses are validated against acceptance criteria and evidence expectations.
5. Project context is updated with decisions, risks, learnings, and follow-up work.

## Summary

This OpenCode configuration is not just a list of agents. It is a structured operating model for software delivery with:

- strong project isolation
- explicit specialist ownership
- reusable skills
- role-based rule enforcement
- evidence-driven delegation
- bounded concurrency
- built-in handling for risk, failure, and evolving context

If you want to change behavior, start with [`opencode.yaml`](./opencode.yaml) and treat the markdown files in [`agents/`](./agents), [`rules/`](./rules), [`skills/`](./skills), and [`context/`](./context) as the implementation details behind that configuration.
