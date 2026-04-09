---
description: Register an existing project in the OpenCode system
agent: orchestrator
subtask: true
---

# Register Project Workflow

You are guiding the user through registering an existing project in the OpenCode system.

---

## Step 1: Path Input

Ask the user for the project path:

```
📁 Please enter the project path (absolute or relative):
```

Wait for user input.

---

## Step 2: Validate Path Exists

Use bash to check if the path exists:

```bash
if [ -d "$PATH" ]; then echo "EXISTS"; else echo "NOT_FOUND"; fi
```

- **If NOT_FOUND**: 
  ```
  ❌ Path does not exist. Please enter a valid project path:
  ```
  Return to Step 1.

- **If EXISTS**: Continue to Step 3.

---

## Step 3: Check if Already Registered

Read `./context/registry.md` to check if this project is already registered.

- **If path matches existing project**:
  ```
  ⚠️ This project is already registered as "{slug}" at {existing_path}.
  
  Options:
  1. Update existing context
  2. Provide different path
  3. Cancel
  ```
  
  - **Option 1**: Load existing context file and allow editing → Continue to Step 6
  - **Option 2**: Return to Step 1
  - **Option 3**: End workflow with cancellation message

- **If NOT registered**: Continue to Step 4.

---

## Step 4: Project Analysis

Analyze the project directory to detect the tech stack and configuration.

### Step 4.1: Detect Package Manager & Project Type

Check for common project files:

```bash
# Check for package.json (Node.js/npm)
if [ -f "$PATH/package.json" ]; then
  echo "DETECTED: package.json"
  cat "$PATH/package.json"
fi

# Check for pyproject.toml (Python/Poetry)
if [ -f "$PATH/pyproject.toml" ]; then
  echo "DETECTED: pyproject.toml"
  cat "$PATH/pyproject.toml"
fi

# Check for Cargo.toml (Rust)
if [ -f "$PATH/Cargo.toml" ]; then
  echo "DETECTED: Cargo.toml"
  cat "$PATH/Cargo.toml"
fi

# Check for go.mod (Go)
if [ -f "$PATH/go.mod" ]; then
  echo "DETECTED: go.mod"
  cat "$PATH/go.mod"
fi

# Check for *.csproj (C#/.NET)
if [ -f "$PATH"/*.csproj ]; then
  echo "DETECTED: .csproj"
  ls "$PATH"/*.csproj
fi

# Check for pom.xml or build.gradle (Java)
if [ -f "$PATH/pom.xml" ]; then
  echo "DETECTED: pom.xml"
  cat "$PATH/pom.xml"
fi
if [ -f "$PATH/build.gradle" ]; then
  echo "DETECTED: build.gradle"
  cat "$PATH/build.gradle"
fi
```

### Step 4.2: Detect Frameworks & Libraries

Check for framework-specific files:

```bash
# React/Vue/Angular
if [ -f "$PATH/package.json" ]; then
  grep -E '"react"|"react-dom"' "$PATH/package.json" && echo "FRAMEWORK: React"
  grep -E '"vue"' "$PATH/package.json" && echo "FRAMEWORK: Vue"
  grep -E '"@angular/core"' "$PATH/package.json" && echo "FRAMEWORK: Angular"
fi

# Next.js
if [ -f "$PATH/package.json" ] && grep -q '"next"' "$PATH/package.json"; then
  echo "FRAMEWORK: Next.js"
fi

# Express
if [ -f "$PATH/package.json" ] && grep -q '"express"' "$PATH/package.json"; then
  echo "FRAMEWORK: Express"
fi

# Django
if [ -f "$PATH/manage.py" ] || [ -f "$PATH/Dockerfile" ] && grep -q "django" "$PATH/Dockerfile"; then
  echo "FRAMEWORK: Django"
fi

# FastAPI
if [ -f "$PATH/main.py" ] && grep -q "fastapi" "$PATH/main.py"; then
  echo "FRAMEWORK: FastAPI"
fi

# .NET
if [ -f "$PATH"/*.csproj ]; then
  echo "FRAMEWORK: .NET"
fi
```

### Step 4.3: Detect Configuration Files

```bash
# TypeScript
if [ -f "$PATH/tsconfig.json" ]; then
  echo "CONFIG: TypeScript"
fi

# Vite
if [ -f "$PATH/vite.config.ts" ] || [ -f "$PATH/vite.config.js" ]; then
  echo "CONFIG: Vite"
fi

# Tailwind
if [ -f "$PATH/tailwind.config.js" ] || [ -f "$PATH/postcss.config.js" ]; then
  echo "CONFIG: Tailwind CSS"
fi

# Docker
if [ -f "$PATH/Dockerfile" ]; then
  echo "CONFIG: Docker"
fi
if [ -f "$PATH/docker-compose.yml" ]; then
  echo "CONFIG: Docker Compose"
fi
```

### Step 4.4: Detect Repository

```bash
# Git repository
if [ -d "$PATH/.git" ]; then
  echo "REPOSITORY: present"
  if [ -f "$PATH/.git/config" ]; then
    grep -A1 "remote.*origin" "$PATH/.git/config" || echo "REPO_URL: unknown"
  fi
fi
```

### Step 4.5: Detect Scripts & Commands

From `package.json` scripts section, detect:
- `dev` or `start` → Development command
- `build` → Build command
- `test` → Test command
- `lint` → Lint command

### Step 4.6: Generate Analysis Summary

Compile all detected information:

| Category | Detected Values |
|----------|-----------------|
| **Languages** | JavaScript, TypeScript, Python, Rust, Go, C#, Java |
| **Frameworks** | React, Next.js, Express, Django, FastAPI, .NET, Spring |
| **Package Managers** | npm, yarn, pnpm, pip, poetry, cargo, go, maven |
| **Project Type** | web-app, backend-api, cli-tool, library, full-stack |
| **Tools** | TypeScript, Vite, Tailwind, Docker, ESLint, Prettier |
| **Repository** | Yes/No, URL (if available) |
| **Commands** | dev, build, test (if detected) |

---

## Step 5: Follow-up Questions

Present analysis and ask for clarification.

### 5.1: Confirm or Correct Analysis

```
🔍 Project Analysis Complete:

- **Name**: {detected_name}
- **Type**: {detected_type}
- **Languages**: {languages}
- **Frameworks**: {frameworks}
- **Package Manager**: {package_manager}
- **Repository**: {repo_status}
- **Commands**: {detected_commands}

Is this correct? (yes/no/edit)
```

- **yes**: Continue to Step 6
- **no**: Ask which fields to correct
- **edit**: Allow user to modify specific fields

### 5.2: Ask for Missing Information

If description is unclear or missing:
```
📝 Please provide a brief description of what this project does:
```

If usage is unclear:
```
🎯 What is the intended usage of this project?
   - Web application
   - Backend API
   - CLI tool
   - Library/package
   - Other: _______
```

If setup instructions are needed:
```
⚙️ Are there any special setup instructions? (e.g., environment variables, dependencies)
```

---

## Step 6: Generate & Confirm Preview

### 6.1: Build Project Context

Generate the project context file based on the template:

```markdown
# {name} - Project Context

> Created: {created_date}
> Updated: {updated_date}
> Status: active

## Overview

- **Name**: {name}
- **Path**: {path}
- **Type**: {project_type}
- **Description**: {description}
- **Repository**: {repository}
- **Repository URL**: {repo_url}
- **Created**: {created_date}
- **Last Updated**: {updated_date}

## Technologies

{detected_technologies_list}

## Goals

1. 

## Requirements

### Functional Requirements

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-01 | | high | pending |

### Non-Functional Requirements

- Performance:
- Availability:
- Security:

## Tech Stack

| Layer | Technology |
|-------|------------|
{tech_stack_table}

## Commands

| Command | Action |
|---------|--------|
| dev | Start development server |
| build | Build for production |
| test | Run tests |

## Architecture

### High-Level Architecture

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
│   ├── repositories/
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

- Budget: 
- Timeline: 
- Team size: 
- Regulatory: 

## Open Questions

| ID | Question | Status | Decision |
|----|----------|--------|----------|
| Q-01 | | open | TBD |

## Decisions

| ID | Decision | Date | Rationale |
|----|----------|------|------------|

## Team

| Role | Name | Contact |
|------|------|---------|
| Lead Developer | | |
| Product Owner | | |

## Notes

{setup_instructions}

---

*This file is maintained by the orchestrator. Updates are made as decisions are finalized and requirements evolve.*
```

### 6.2: Show Preview

```
📋 Project Context Preview:
```

Display the generated markdown.

### 6.3: Confirmation Options

```
Actions:
1. ✅ Confirm and save
2. ✏️ Edit specific fields
3. ❌ Cancel

Enter your choice:
```

- **1**: Proceed to Step 7
- **2**: Ask which field to edit, then regenerate preview
- **3**: End workflow with cancellation message

---

## Step 7: Save

### 7.1: Determine if New or Update

- **If updating existing**: Overwrite `./context/projects/{slug}.md`
- **If new**: Create `./context/projects/{slug}.md`

### 7.2: Update Registry

If new project, append to `./context/registry.md`:

```yaml
- slug: {slug}
  name: {name}
  description: {description}
  path: {path}
  status: active
  created: {created_date}
  tech_stack:
    {tech_stack_items}
```

### 7.3: Confirm Success

```
✅ Project registered successfully!

- **Name**: {name}
- **Slug**: {slug}
- **Path**: {path}
- **Type**: {project_type}
- **Technologies**: {technologies_list}

Files updated:
- ./context/projects/{slug}.md
- ./context/registry.md
```

---

## Error Handling

| Error | Message | Recovery |
|-------|---------|----------|
| Path not found | ❌ Path does not exist | Ask for new path |
| No project files detected | ⚠️ Could not detect project type. Please describe manually. | Ask for details |
| Already registered | ⚠️ Project already registered | Offer update or new path |
| Permission denied | ❌ Cannot write to context directory | Report error |
| Invalid YAML in registry | ❌ Registry file is corrupted | Offer to repair |

---

## End Workflow

Ask the user:
1. Start working on this project
2. Add more details to the project context
3. Register another project
4. Exit
