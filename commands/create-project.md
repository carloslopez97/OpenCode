---
description: Register a new project in the system
agent: orchestrator
subtask: true
---

# Create Project Workflow

You are guiding the user through registering a new project in the OpenCode system.

## Input Parsing

Parse the following input to extract any provided fields:

```
$ARGUMENTS
```

Extract what you can and identify missing fields.

---

## Step 1: Input Collection

Collect missing information through interactive prompts. Ask for each missing field:

### Required Fields

**Project Name**
- Ask: "What is the project name?"
- Validate: Cannot be empty

**Project Path**
- Ask: "What is the absolute path to the project directory?"
- Validate: Must be a valid path (user will confirm it exists)

**Technologies**
- Ask: "What technologies will this project use? (comma-separated list)"
- Validate: Cannot be empty, split into array
- Examples: React, TypeScript, .NET, PostgreSQL, Node.js

### Optional Fields

**Repository**
- Ask: "Will this project have a Git repository? (yes/no)"
- Normalize: "yes" → true, "no" → false, y → true, n → false

**Repository Name** (only if repository = true)
- Ask: "What is the repository name (e.g., owner/repo)?"
- Only ask if repository is true/yes

**Description**
- Ask: "Brief description of the project (optional):"

**Document**
- Ask: "Is there an existing document or file to reference for this project? Provide the path if so:"

---

## Step 2: Validation

Validate all collected data:

| Field | Validation |
|-------|------------|
| name | Not empty, trimmed |
| path | Not empty, trimmed |
| technologies | Array, not empty |
| repository | Boolean (true/false) |
| repository_name | If repository=true, not empty |
| description | Optional, trimmed if provided |
| document | Optional, trimmed if provided |

Generate the project slug:
- Convert name to kebab-case
- Example: "My Project" → "my-project"

---

## Step 3: Context Registration

Create the project context file and update registry.

### Generate Project Context File

Create file: `./context/projects/{slug}.md`

```markdown
# {name} - Project Context

> Created: {current_date}
> Status: active

## Overview

- **Name**: {name}
- **Path**: {path}
- **Description**: {description}
- **Repository**: {repository}
- **Repository Name**: {repository_name}
- **Created**: {current_date}
- **Last Updated**: {current_date}

## Technologies

{technologies_list}

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

{document_reference}

---

*This file is maintained by the orchestrator. Updates are made as decisions are finalized and requirements evolve.*
```

### Update Registry

Append to `./context/registry.md`:

```yaml
- slug: {slug}
  name: {name}
  description: {description}
  path: {path}
  status: active
  created: {current_date}
  tech_stack:
    {tech_stack_items}
```

---

## Step 4: Confirmation

Present the results to the user:

```
✅ Project registered successfully!

- **Name**: {name}
- **Slug**: {slug}
- **Path**: {path}
- **Technologies**: {technologies_list}
- **Repository**: {repository}
{document_info}
{reference_info}

Files created/updated:
- ./context/projects/{slug}.md
- ./context/registry.md
```

Ask the user if they want to:
1. Start working on this project
2. Add more details to the project context
3. Create another project

---

## Step 5: Project Creation

Now actually create the project on disk using CLI tools based on the selected technologies.

### Step 5.1: Validate Path

First, ensure the parent directory exists and is writable:

```bash
# Get parent directory from path
PARENT_DIR=$(dirname "{path}")
mkdir -p "$PARENT_DIR" 2>/dev/null || echo "ERROR: Cannot create parent directory"
```

If error, report to user and abort.

### Step 5.2: Map Technologies to CLI Commands

Based on the selected technologies, determine which scaffolding commands to run:

| Technology | CLI Command | Notes |
|------------|-------------|-------|
| React | `npm create vite@latest . -- --template react-ts` | Uses Vite for modern React |
| Vue | `npm create vue@latest . --default` | Default options |
| Angular | `npx @angular/cli new . --standalone --routing --style=scss` | Modern Angular |
| Next.js | `npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"` | Full-featured Next.js |
| Node.js / Express | `npx express-generator . --no-view` | Express with no template |
| .NET | `dotnet new webapi -o . --framework net9.0` | .NET 9 Web API |
| Python / Django | `django-admin startproject .` | Run from within project folder |
| Python / FastAPI | Manual | Create requirements.txt manually |
| Go | `go mod init {slug}` | Initialize Go module |
| Rust | `cargo init --lib .` | Rust library template |
| PostgreSQL | docker-compose.yml | Create with docker-compose |
| MongoDB | docker-compose.yml | Create with docker-compose |

### Step 5.3: Build Command List

Compile the list of commands to execute based on technologies:

```
Technologies: {technologies_list}

Commands to execute:
{command_list}
```

### Step 5.4: Show Planned Structure

Display what will be created:

```
📦 Project structure to create at {path}:

{folder_structure}

Scaffolding commands:
{cli_commands}
```

### Step 5.5: Ask Confirmation

```
Proceed with project creation? (yes/no/edit):
```

- **yes**: Continue to Step 5.6
- **no**: Skip project creation, go to Step 4 completion
- **edit**: Ask which element to modify, then regenerate

### Step 5.6: Execute Scaffolding

#### 5.6.1: Navigate to Project Directory

```bash
cd "{path}" || echo "ERROR: Cannot navigate to path"
```

#### 5.6.2: Run CLI Commands

Execute each scaffolding command in sequence:

```bash
# Example for React + TypeScript
npm create vite@latest . -- --template react-ts

# Example for .NET
dotnet new webapi -o . --framework net9.0
```

Handle errors gracefully:
- If command fails, ask user if they want to continue with manual setup
- Log errors but continue with other commands if possible

#### 5.6.3: Apply Clean Architecture Folders

Create the Clean Architecture folder structure inside `src/`:

```bash
mkdir -p src/domain/entities
mkdir -p src/domain/valueObjects
mkdir -p src/domain/repositories
mkdir -p src/domain/services
mkdir -p src/application/useCases
mkdir -p src/application/interfaces
mkdir -p src/application/dtos
mkdir -p src/application/services
mkdir -p src/infrastructure/persistence
mkdir -p src/infrastructure/external
mkdir -p src/infrastructure/config
mkdir -p src/presentation/controllers
mkdir -p src/presentation/handlers
```

#### 5.6.4: Create Configuration Files

Create standard configuration files:

**`.env.example`**:
```
DATABASE_URL=postgresql://user:password@localhost:5432/db
PORT=3000
NODE_ENV=development
```

**`docker-compose.yml`** (if database selected):
```yaml
version: '3.8'
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
volumes:
  postgres_data:
```

#### 5.6.5: Initialize Git (if repository = true)

```bash
git init

# Create .gitignore
cat > .gitignore << EOF
node_modules/
dist/
.env
*.log
.DS_Store
EOF

# Create initial commit
git add .
git commit -m "Initial commit: $(date +%Y-%m-%d)"

# Add remote if repository_name provided
if [ -n "{repository_name}" ]; then
  git remote add origin "https://github.com/{repository_name}.git"
fi
```

#### 5.6.6: Install Dependencies (if applicable)

```bash
# For Node.js projects
npm install

# For Python projects (if using virtualenv)
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
```

### Step 5.7: Show Created Files

After creation, list what was created:

```
✅ Project created successfully!

📁 Files created at {path}:

$ ls -la

$ ls -la src/
```

### Step 5.8: Handle Errors

If any step fails:

```
⚠️ Some errors occurred during project creation:

{error_list}

Options:
1. Retry
2. Continue with manual setup
3. Cancel and roll back
```

- **1**: Retry the failed command
- **2**: Continue, skipping failed commands
- **3**: Clean up created files and abort

### Step 5.9: Final Confirmation

```
🎉 Project ready!

- **Name**: {name}
- **Path**: {path}
- **Slug**: {slug}
- **Technologies**: {technologies_list}
- **Git**: {repository_initialized}

Next steps:
1. cd {path} && npm run dev  (to start development)
2. Edit ./context/projects/{slug}.md to add more details
3. Run /register-project if you want to re-analyze later

Would you like to:
- Start development server?
- Add more configuration?
- Exit?
```

---

## Complete Workflow Summary

```
Step 1: Input Collection
   └── Collect: name, path, technologies, repository, description

Step 2: Validation
   └── Validate fields, generate slug

Step 3: Context Registration
   └── Create: ./context/projects/{slug}.md
   └── Update: ./context/registry.md

Step 4: Confirmation
   └── Show summary, ask for confirmation

Step 5: Project Creation (NEW)
   └── 5.1: Validate path
   └── 5.2: Map technologies to CLI commands
   └── 5.3: Build command list
   └── 5.4: Show planned structure
   └── 5.5: Ask confirmation
   └── 5.6: Execute scaffolding
   └── 5.7: Show created files
   └── 5.8: Handle errors
   └── 5.9: Final confirmation
```
