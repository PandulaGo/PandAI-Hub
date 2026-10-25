# AI Agents Skills — Multi-Agent SDLC Framework

**Version 3.0.0** — see [CHANGELOG.md](./CHANGELOG.md) for the full version history.

A simplified, tool-agnostic multi-agent SDLC framework that runs software ideas through a 3-stage pipeline (Analysis → Implementation → Verification) with hybrid verification gates and consistent standards across all projects.

## Why This Framework Exists

This framework solves **consistency** across projects:
- Naming, architecture, code style, testing, and git conventions are defined once in `standards/` and enforced automatically
- Every project produces the same documentation structure
- Every idea moves through the same disciplined pipeline process
- Tech stack standards are validated before starting — no more ad hoc setups

## Architecture Overview

### 8 Agents (down from 10)

| Agent | Role | Location |
|-------|------|----------|
| `coordinator` | Drives the pipeline, coordinates stages | `agent/sdlc/` |
| `solution-architect` | Analysis (plan + requirements + design) | `agent/sdlc/` |
| `software-engineer` | Implementation | `agent/sdlc/` |
| `qa-engineer` | Testing & validation | `agent/sdlc/` |
| `release-engineer` | Post-release maintenance | `agent/sdlc/` |
| `auditor` | Quality gate verification | `agent/sdlc/` |
| `technical-writer` | Documentation sync | `agent/` |
| `researcher` | Research assistant | `agent/` |

### 3-Stage Pipeline (down from 6)

```
/start --stack dotnet,angular <idea>
         ↓
@coordinator validates stacks, creates scaffold
         ↓
Stage 1: @solution-architect → @auditor gate → Tech Lead review
         ↓
Stage 2: @software-engineer → Automated gate (lint/typecheck/build) → Tech Lead review
         ↓
Stage 3: @qa-engineer → Automated gate (test coverage) → Tech Lead review
         ↓
@technical-writer syncs docs → DONE
         ↓
Maintenance: @release-engineer (ongoing)
```

### Hybrid Gates (saves tokens)

| Stage | Gate Type | Token Cost |
|-------|-----------|-----------|
| Analysis | @auditor agent check | 1 gatekeeper call |
| Implementation | Automated (lint/typecheck/build) | 0 tokens |
| Verification | Automated (tests + coverage) | 0 tokens |

### 4 Commands (down from 9)

| Command | Agent | Description |
|---------|-------|-------------|
| `/start --stack <stacks> <idea>` | @coordinator | Initialize + brief + start pipeline |
| `/status [advance\|redo]` | @coordinator | Show pipeline status, optionally advance |
| `/docs [arch\|session]` | @technical-writer | Sync architecture + session ledger |
| `/research <topic>` | @researcher | Research topics, view/promote standards |

## Standards Structure

### Layered Standards (universal + stack-specific)

```
standards/
├── universal/              # Always loaded (5 files)
│   ├── project-structure.md   # Backend/, Frontend/, Tests/, etc.
│   ├── code-style.md          # Small functions, early returns, etc.
│   ├── testing.md             # AAA pattern, coverage expectations
│   ├── git.md                 # Conventional commits, branch naming
│   └── api-design.md          # REST conventions, response format
│
├── dotnet/                 # .NET projects (5 files)
│   ├── architecture.md        # Clean Architecture
│   ├── naming.md              # PascalCase, C# conventions
│   ├── csharp-conventions.md  # Primary constructors, records, async
│   ├── efcore-conventions.md  # Repository pattern, Unit of Work
│   └── aspnet-conventions.md  # Controllers, middleware, DI
│
├── node/                   # Node.js projects (4 files)
│   ├── architecture.md        # Layered architecture
│   ├── naming.md              # camelCase, kebab-case files
│   ├── typescript-conventions.md
│   └── express-conventions.md
│
├── python/                 # Python projects (5 files)
│   ├── architecture.md        # Service + Repository pattern
│   ├── naming.md              # snake_case, PEP 8
│   ├── python-conventions.md  # Type hints, async, error handling
│   ├── django-conventions.md
│   └── fastapi-conventions.md
│
├── angular/                # Angular projects (3 files)
│   ├── architecture.md        # Feature-based modules
│   ├── naming.md              # kebab-case files
│   └── angular-conventions.md # Standalone components, signals
│
└── react/                  # React projects (3 files)
    ├── architecture.md        # Feature-based structure
    ├── naming.md              # PascalCase components
    └── react-conventions.md   # Hooks, state management
```

### Stack Validation

The framework validates tech stacks before starting:

```
/start --stack dotnet,angular Build a user management system
→ ✅ Both stacks found, proceeding...

/start --stack vue,svelte Build a blog
→ ❌ ERROR: No standards defined for: vue, svelte
→ To add support: Create standards/vue/ with architecture.md, naming.md, etc.
```

This forces a **standards-first approach** — new tech stacks must be defined before use.

## Project Structure (Consistent Across All Projects)

```
project-root/
├── Backend/                    # Backend application code
├── Frontend/                   # Frontend application code
├── Tests/                      # Unit, Integration, E2E
├── Scripts/                    # Build, deploy, utility scripts
├── Configurations/             # Environment configs
│   ├── .env
│   ├── appsettings.backend.json
│   └── appsettings.frontend.json
└── Docs/                       # Documentation
    ├── architecture-and-api-reference.md
    ├── context-ledger.md
    └── sdlc/
        └── <project-slug>/
            ├── analysis.md          # Combined plan + requirements + design
            ├── dev-notes.md         # Implementation notes
            ├── test-report.md       # Test results
            └── state.json           # Pipeline state
```

## Quick Start

### 1. Install
```bash
git clone <this-repo>
cd "AI Agents Skills"
```

### 2. Configure Your AI Tool

Copy the appropriate adapter to your project root:

**For OpenCode:**
```bash
cp adapters/opencode/opencode.jsonc /path/to/your/project/opencode.jsonc
```

**For Claude Code:**
```bash
cp adapters/claude/CLAUDE.md /path/to/your/project/CLAUDE.md
```

### 3. Start a Project
```
/start --stack dotnet,angular Build a habit tracking app with daily check-ins
```

### 4. Follow the Pipeline
The coordinator will:
1. Validate tech stacks
2. Create project scaffold
3. Run @solution-architect for analysis
4. Run @auditor for quality gate
5. Pause for your review
6. Continue through Implementation and Verification
7. Sync documentation when complete

## File Reference

### Agents (`agent/`)
| File | Agent | Purpose |
|------|-------|---------|
| `sdlc/coordinator.md` | @coordinator | Pipeline driver |
| `sdlc/solution-architect.md` | @solution-architect | Analysis stage |
| `sdlc/software-engineer.md` | @software-engineer | Implementation |
| `sdlc/qa-engineer.md` | @qa-engineer | Testing |
| `sdlc/release-engineer.md` | @release-engineer | Maintenance |
| `sdlc/auditor.md` | @auditor | Quality gate |
| `technical-writer.md` | @technical-writer | Documentation |
| `researcher.md` | @researcher | Research |

### Commands (`command/`)
| File | Command | Purpose |
|------|---------|---------|
| `start.md` | `/start` | Initialize + start pipeline |
| `status.md` | `/status` | Show/advance pipeline |
| `docs.md` | `/docs` | Sync documentation |
| `research.md` | `/research` | Research topics |

### Standards (`standards/`)
| Folder | Stacks | Files |
|--------|--------|-------|
| `universal/` | All | 5 files |
| `dotnet/` | .NET + EF Core + ASP.NET | 5 files |
| `node/` | Node.js + Express + TypeScript | 4 files |
| `python/` | Python + Django + FastAPI | 5 files |
| `angular/` | Angular | 3 files |
| `react/` | React | 3 files |

### Templates (`templates/`)
| File | Purpose |
|------|---------|
| `idea-intake-template.md` | Structured project intake form |
| `analysis-output-template.md` | Template for @solution-architect output |
| `architecture-and-other-details.md` | As-built architecture template |
| `context-ledger.md` | Session continuity template |
| `scaffold/` | Project initialization files |

### Adapters (`adapters/`)
| Folder | AI Tool | File |
|--------|---------|------|
| `opencode/` | OpenCode | `opencode.jsonc` |
| `claude/` | Claude Code | `CLAUDE.md` |

## Supported Stack Combinations

| Backend | Frontend | Command |
|---------|----------|---------|
| .NET (ASP.NET + EF Core) | Angular | `--stack dotnet,angular` |
| .NET (ASP.NET + EF Core) | React | `--stack dotnet,react` |
| Node.js (Express + TypeScript) | Angular | `--stack node,angular` |
| Node.js (Express + TypeScript) | React | `--stack node,react` |
| Python (Django) | Angular | `--stack python,django,angular` |
| Python (Django) | React | `--stack python,django,react` |
| Python (FastAPI) | Angular | `--stack python,fastapi,angular` |
| Python (FastAPI) | React | `--stack python,fastapi,react` |

## Contributing New Standards

To add a new tech stack:

1. Create `standards/<stack>/` folder
2. Add required files:
   - `architecture.md` — Project structure and layering
   - `naming.md` — Naming conventions
   - `<framework>-conventions.md` — Framework-specific patterns
3. Update `AGENTS.md` with the new stack
4. Test with `/start --stack <stack> <idea>`

## Version History

See [CHANGELOG.md](./CHANGELOG.md) for details.

| Version | Key Changes |
|---------|-------------|
| 3.0.0 | Simplified to 8 agents, 3 stages, hybrid gates, tool-agnostic |
| 2.1.0 | Qwen/DeepSeek usage reduction |
| 2.0.0 | Multi-agent pipeline |
| 1.0.0 | Initial single-agent setup |

## License

MIT
