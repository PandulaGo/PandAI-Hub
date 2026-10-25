# Changelog

All notable changes to the AI Agents Skills SDLC Framework.

---

## [3.0.0] - 2026-10-25

### Major Changes

#### Architecture Simplification
- **Agents reduced: 10 → 8** (professional role-based names)
- **Pipeline stages reduced: 6 → 3** (Analysis → Implementation → Verification)
- **Commands reduced: 9 → 4** (`/start`, `/status`, `/docs`, `/research`)
- **Artifacts reduced: 6 → 3** per project (`analysis.md`, `dev-notes.md`, `test-report.md`)

#### Agent Renames
| Old Name | New Name | Reason |
|----------|----------|--------|
| orchestrator | coordinator | Clearer pipeline coordination role |
| planner + requirements + designer | solution-architect | Merged into single analysis stage |
| developer | software-engineer | Professional title, distinguishes from qa-engineer |
| tester | qa-engineer | Professional title |
| maintainer | release-engineer | Clearer post-release role |
| gatekeeper | auditor | More professional verification role |
| doc-agent | technical-writer | Professional title |
| learn | researcher | Noun-based name, moved outside sdlc/ |

#### Hybrid Gates
- **Stage 1 (Analysis):** Agent-based gate (@auditor) — checks document quality
- **Stage 2 (Implementation):** Automated gate (lint/typecheck/build) — 0 token cost
- **Stage 3 (Verification):** Automated gate (tests + coverage) — 0 token cost
- **Token savings:** ~80% reduction in gate-related token usage
- **No gate retries** — fail fast, manual review by Tech Lead

#### Standards Restructure
- **Layered standards:** Universal (always loaded) + Stack-specific (loaded per `--stack` flag)
- **Stack validation:** Framework validates stacks exist before starting pipeline
- **Standards-first approach:** New tech stacks must be defined before use
- **5 tech stacks supported:**
  - `.NET` (ASP.NET + EF Core) — 5 standards files
  - `Node.js` (Express + TypeScript) — 4 standards files
  - `Python` (Django + FastAPI) — 5 standards files
  - `Angular` — 3 standards files
  - `React` — 3 standards files

#### Tool-Agnostic Design
- Agent prompts use plain text references (no tool-specific syntax)
- Standards files are universal markdown
- Adapters for different AI tools:
  - `adapters/opencode/opencode.jsonc`
  - `adapters/claude/CLAUDE.md`

#### Structural Changes
- `agent/SDLC Agent/` renamed to `agent/sdlc/`
- Support agents (`technical-writer`, `researcher`) moved outside `sdlc/`
- Root `opencode.jsonc` removed — use adapters instead
- All agents now use professional role-based names

### New Files
- `standards/universal/project-structure.md`
- `standards/universal/api-design.md`
- `standards/dotnet/csharp-conventions.md`
- `standards/dotnet/efcore-conventions.md`
- `standards/dotnet/aspnet-conventions.md`
- `standards/node/architecture.md`
- `standards/node/naming.md`
- `standards/node/typescript-conventions.md`
- `standards/node/express-conventions.md`
- `standards/python/architecture.md`
- `standards/python/naming.md`
- `standards/python/python-conventions.md`
- `standards/python/django-conventions.md`
- `standards/python/fastapi-conventions.md`
- `standards/angular/architecture.md`
- `standards/angular/naming.md`
- `standards/angular/angular-conventions.md`
- `standards/react/architecture.md`
- `standards/react/naming.md`
- `standards/react/react-conventions.md`
- `agent/sdlc/coordinator.md`
- `agent/sdlc/solution-architect.md`
- `agent/sdlc/software-engineer.md`
- `agent/sdlc/qa-engineer.md`
- `agent/sdlc/release-engineer.md`
- `agent/sdlc/auditor.md`
- `agent/technical-writer.md`
- `agent/researcher.md`
- `command/start.md`
- `command/status.md`
- `command/docs.md`
- `command/research.md`
- `templates/analysis-output-template.md`
- `adapters/opencode/opencode.jsonc`
- `adapters/claude/CLAUDE.md`

### Removed Files
- `agent/SDLC Agent/` directory (replaced by `agent/sdlc/`)
- `agent/doc-agent.md` (moved to `agent/technical-writer.md`)
- `agent/learn.md` (moved to `agent/researcher.md`)
- `command/sdlc-init.md`, `sdlc-brief.md`, `sdlc-start.md`, `sdlc-next.md`, `sdlc-status.md`
- `command/docs-sync.md`, `wrap-session.md`, `promote-standard.md`, `standards.md`
- Root `opencode.jsonc` (use adapters instead)

### Migration Guide

#### For Existing Projects
1. Copy adapter config to your project root:
   ```bash
   cp adapters/opencode/opencode.jsonc /your/project/opencode.jsonc
   ```
2. Update any references to old agent names in your workflows
3. Run `/start` to re-initialize with new structure

#### For New Projects
1. Clone this repository
2. Copy adapter to your project root
3. Run `/start --stack <stacks> <idea>`

---

## [2.1.0] - Previous

### Qwen/DeepSeek Usage Reduction
- Consolidated gatekeeper agents (deepseek + qwen → single big-pickle)
- Input trimming strategy for stage agents
- Model-specific optimization for lighter stages

---

## [2.0.0] - Previous

### Multi-Agent Pipeline
- Introduced 6-stage pipeline (Planning → Requirements → Design → Development → Testing → Maintenance)
- Added verification gates between stages
- Multiple specialized agents
- State tracking via state.json

---

## [1.0.0] - Initial

### Single-Agent Setup
- Initial SDLC framework
- Basic documentation templates
- Single agent handling all stages
