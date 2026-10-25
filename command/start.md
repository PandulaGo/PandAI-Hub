---
description: Initialize project, capture idea, and start the SDLC pipeline
agent: coordinator
---
Start a new SDLC project. This single command replaces init + brief + start.

## Usage

```
/start --stack <stacks> <idea or description>
```

### Examples
```
/start --stack dotnet,angular Build a user management system with login, registration, and role-based access control
/start --stack node,react Create a REST API for order management with CRUD operations
/start --stack python,fastapi,react Build a blog platform with posts, comments, and user authentication
/start --stack dotnet,angular (then paste the structured intake template)
```

## Stack Validation

Before doing anything, validate the requested stacks:

1. Parse `--stack` argument (comma-separated list)
2. Check each stack exists in `standards/` folder:
   - `standards/dotnet/` → ✅ .NET
   - `standards/node/` → ✅ Node.js
   - `standards/python/` → ✅ Python
   - `standards/angular/` → ✅ Angular
   - `standards/react/` → ✅ React
3. If any stack is missing:
   ```
   ERROR: No standards defined for: [missing-stack]
   
   To add support:
   1. Create standards/[missing-stack]/ folder
   2. Add required files:
      - architecture.md
      - naming.md
      - [framework]-conventions.md
   3. Retry /start
   
   Available stacks: dotnet, node, python, angular, react
   ```
4. Never proceed without valid stacks

## Step 1 — Determine the project slug

1. `git rev-parse --is-inside-work-tree`
   - If fails: use current directory name, warn user
2. If git repo: `git remote get-url origin`
   - If succeeds: take last path segment, strip `.git`
   - If fails: `git rev-parse --show-toplevel`, use basename
3. Sanitize: lowercase, replace spaces/underscores with hyphens, strip non-alphanumeric
4. Report: `Detected project slug: habit-tracker (from git remote origin)`

## Step 2 — Check for existing project (idempotency)

If `Docs/sdlc/<slug>/` already exists:
- If `state.json` stage is `"not_started"`: resume from Analysis
- If `state.json` stage is anything else: report current stage, ask before proceeding
- Do NOT overwrite existing artifacts

## Step 3 — Create project scaffold (if not exists)

```
Backend/                (+ .gitkeep)
Frontend/               (+ .gitkeep)
Tests/                  (+ .gitkeep)
Scripts/                (+ .gitkeep)
Configurations/
  .env
  appsettings.backend.json
  appsettings.frontend.json
Docs/
  architecture-and-api-reference.md   (seed from template, unfilled)
  context-ledger.md                   (seed from template, unfilled)
  sdlc/
    <slug>/
      00-idea.md
      state.json
```

## Step 4 — Capture the idea

If $ARGUMENTS is empty or doesn't look like a filled-in template:
1. Show the user the structured intake template from `templates/idea-intake-template.md`
2. Ask them to reply with it filled in
3. STOP — do not proceed to pipeline yet

Once you have a filled-in version:
1. Write it into `Docs/sdlc/<slug>/00-idea.md`
2. Update `state.json`: `{ "slug": "<slug>", "stage": "analysis", "history": [] }`

## Step 5 — Start the pipeline

1. Load standards for requested stacks
2. Call @solution-architect with idea + standards
3. Follow the pipeline flow defined in `agent/sdlc/coordinator.md`
4. Pause for Tech Lead review after each gate

## Rules

- Always validate stacks first
- Never skip the structured intake (unless idea is already in `00-idea.md`)
- Never overwrite existing artifacts without confirmation
- Always report slug detection result
- Always pause for Tech Lead approval after each stage
