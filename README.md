# Multi-Agent Development System

A production-ready orchestrator-driven multi-agent system for managing multiple software projects with isolated contexts and structured collaboration.

## Overview

This system implements a clean architecture approach to multi-agent software development where:

- **Orchestrator** manages everything and delegates to agents
- **Agents** execute specific tasks (implementation, testing, review, etc.)
- **Skills** provide domain-specific knowledge
- **Rules** enforce standards and prevent anti-patterns
- **Context** maintains project-specific information

## Quick Start

### 1. Project Detection

The orchestrator automatically detects which project you're working on:
1. Reads `C:\Users\carlo\.config\opencode/context/registry.md` to list registered projects
2. Infers project from your prompt
3. Asks "Which project does this belong to?" if unclear
4. Creates new project if requested

### 2. Context Management

Once a project is identified:
- Orchestrator loads `C:\Users\carlo\.config\opencode/context/projects/{project-slug}.md`
- This is the single source of truth for all decisions
- Context is updated after each task completion

### 3. Task Delegation

When you request work:
1. Orchestrator analyzes requirements
2. Decomposes into atomic tasks
3. Delegates to appropriate agent
4. Validates response
5. Updates context

## File Structure

```
.opencode/
├── modes/
│   └── orchestrator.md          # Orchestrator mode definition
├── agents/
│   ├── architect.md             # System design
│   ├── backend-developer.md     # Backend implementation
│   ├── frontend-developer.md    # Frontend implementation
│   ├── reviewer.md              # Code quality review
│   ├── tester.md                # Testing
│   └── devops.md                # CI/CD & deployment
├── skills/
│   ├── clean-architecture/
│   ├── code-quality/
│   ├── testing/
│   ├── api-design/
│   └── naming-conventions/
├── rules/
│   ├── architecture-rules.md
│   ├── coding-rules.md
│   ├── testing-rules.md
│   └── review-rules.md
├── context/
│   ├── registry.md              # Project registry
│   └── projects/
│       └── template.md          # Project context template
└── opencode.yaml                # Main configuration
```

## Adding a New Project

1. Create project context file:
   ```
   C:\Users\carlo\.config\opencode/context/projects/my-project.md
   ```

2. Add to registry:
   ```yaml
   - slug: my-project
     name: My Project
     description: Description
     status: active
     created: YYYY-MM-DD
     tech_stack: [TypeScript, Node.js]
   ```

3. Orchestrator will automatically detect and use it

## Task Contract Format

```json
{
  "task_id": "unique-id",
  "project": "project-slug",
  "assigned_to": "backend-developer",
  "task_type": "implementation",
  "priority": "high",
  "context": {
    "summary": "Implement order creation API",
    "requirements": ["POST /orders endpoint", "validate input"],
    "constraints": ["use existing repository pattern"],
    "related_files": ["src/domain/entities/Order.ts"]
  },
  "input": {
    "description": "Create order endpoint",
    "artifacts": ["api-spec.yaml"]
  },
  "expected_output": {
    "type": "code",
    "format": "TypeScript",
    "definition_of_done": ["endpoint works", "tests pass"]
  },
  "acceptance_criteria": [
    "Returns 201 on success",
    "Validates input",
    "Returns 400 on invalid input"
  ]
}
```

## Agent Selection

| Task | Agent |
|------|-------|
| System design, patterns | architect |
| Backend APIs, services | backend-developer |
| UI, components, state | frontend-developer |
| Code review, quality | reviewer |
| Tests | tester |
| CI/CD, deployment | devops |

## Hard Constraints

- ✅ Orchestrator NEVER writes implementation code
- ✅ Agents NEVER modify rules
- ✅ No overlapping responsibilities
- ✅ Everything modular
- ✅ Context isolation enforced

## Configuration

Main configuration is in `opencode.yaml`:
- Mode definitions
- Agent registration
- Skill mappings
- Rule loading
- Context management

## Customization

### Adding Skills

1. Create `C:\Users\carlo\.config\opencode/skills/{skill-name}/SKILL.md`
2. Add to `opencode.yaml` skills section

### Adding Rules

1. Create `C:\Users\carlo\.config\opencode/rules/{rule-name}.md`
2. Reference in `opencode.yaml`

### Adding Agents

1. Create `C:\Users\carlo\.config\opencode/agents/{agent-name}.md`
2. Register in `opencode.yaml`

## Best Practices

1. **Always identify project first** - Don't assume, check registry
2. **Keep contexts isolated** - Never mix project contexts
3. **Use task contracts** - Structured delegation prevents ambiguity
4. **Update context after each task** - Document decisions and learnings
5. **Enforce rules consistently** - Prevent technical debt

## System Flow

```
User Request
     │
     ▼
┌─────────────────┐
│  Orchestrator   │
│ (identifies     │
│  project)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Load Project    │
│ Context         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Analyze &       │
│ Decompose      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Delegate to     │
│ Agent        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Agent        │
│ Executes        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Validate        │
│ Response        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Update Context  │
│ & Continue      │
└─────────────────┘
```

---

*This system enables structured, scalable multi-agent collaboration while maintaining clean separation of concerns.*