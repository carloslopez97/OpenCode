# Project Registry

## Registered Projects

This file tracks all projects managed by the orchestrator system.

### Format

```yaml
- slug: project-slug
  name: Project Name
  description: Brief description
  path: /absolute/path/to/project
  status: active|archived
  created: YYYY-MM-DD
  tech_stack: []
```

### Projects

<!-- Add new projects below -->

Use this template to add projects:

```yaml
- slug: example-project
  name: Example Project
  description: An example project
  path: /absolute/path/to/example-project
  status: active
  created: 2024-01-01
  tech_stack:
    - TypeScript
    - Node.js
    - PostgreSQL
    - React
```

---

## Adding New Projects

1. Create context file: `./context/projects/{slug}.md`
2. Add entry to this registry (include the absolute `path` field)
3. Orchestrator will load context automatically

## Project Lifecycle

1. **Creating** - Project being set up
2. **Active** - In development/maintenance
3. **Archived** - No longer active

## Switching Projects

The orchestrator will:

1. Read this registry to identify projects
2. Ask user to select project if unclear
3. Load project-specific context
