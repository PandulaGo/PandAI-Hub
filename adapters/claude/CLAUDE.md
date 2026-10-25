# AI Agents Skills — Claude Code Configuration

This file configures Claude Code to use the AI Agents Skills SDLC framework.
Copy this file to your project root as `CLAUDE.md`.

## Agent Configuration

### SDLC Pipeline Agents

#### @coordinator
Coordinates the SDLC pipeline through Analysis, Implementation, and Verification stages.
- Mode: Primary agent
- Reads: `agent/sdlc/coordinator.md`
- Never writes code — delegates to stage agents

#### @solution-architect
Analyzes requirements and designs solutions (merged planner + requirements + designer).
- Mode: Subagent
- Reads: `agent/sdlc/solution-architect.md`
- Output: `analysis.md` (combined plan + requirements + design)
- Model: Use your strongest model for architecture decisions

#### @software-engineer
Implements the application per the approved design.
- Mode: Subagent
- Reads: `agent/sdlc/software-engineer.md`
- Output: Source code + `dev-notes.md`
- Must follow all standards in `standards/`

#### @qa-engineer
Tests the implementation against requirements and design.
- Mode: Subagent
- Reads: `agent/sdlc/qa-engineer.md`
- Output: `test-report.md`
- Reports bugs, does not fix them

#### @release-engineer
Handles ongoing changes, bug fixes, and maintenance after initial release.
- Mode: Subagent
- Reads: `agent/sdlc/release-engineer.md`
- Follows same standards as @software-engineer
- Escalates major changes back to pipeline

#### @auditor
Strict verification gate between SDLC stages.
- Mode: Subagent
- Reads: `agent/sdlc/auditor.md`
- Returns GATE: PASS or GATE: FAIL with numbered issues
- Never softens FAIL into PASS

### Support Agents

#### @technical-writer
Maintains as-built architecture docs and session continuity ledger.
- Mode: Primary agent
- Reads: `agent/technical-writer.md`
- Two responsibilities:
  1. Architecture & API reference (after pipeline stages)
  2. Session continuity ledger (via /docs)

#### @researcher
Research and learning assistant.
- Mode: Primary agent
- Reads: `agent/researcher.md`
- No write access, no standards loaded
- Uses webfetch for research
- Can promote learnings to standards via /research promote

## Standards

### Universal Standards (loaded for every project)
- `standards/universal/project-structure.md`
- `standards/universal/code-style.md`
- `standards/universal/testing.md`
- `standards/universal/git.md`
- `standards/universal/api-design.md`

### Stack-Specific Standards (loaded based on --stack flag)
- **dotnet/**: architecture, naming, csharp-conventions, efcore-conventions, aspnet-conventions
- **node/**: architecture, naming, typescript-conventions, express-conventions
- **python/**: architecture, naming, python-conventions, django-conventions, fastapi-conventions
- **angular/**: architecture, naming, angular-conventions
- **react/**: architecture, naming, react-conventions

## Commands

| Command | Agent | Description |
|---------|-------|-------------|
| `/start --stack <stacks> <idea>` | @coordinator | Initialize + brief + start pipeline |
| `/status [advance\|redo\|history]` | @coordinator | Show pipeline status, optionally advance |
| `/docs [arch\|session]` | @technical-writer | Sync architecture + session ledger |
| `/research <topic>` | @researcher | Research topics, view/promote standards |

## Workflow

1. Run `/start --stack dotnet,angular Build a user management system`
2. @coordinator validates stacks, creates scaffold, starts pipeline
3. @solution-architect produces `analysis.md`
4. @auditor verifies analysis → Tech Lead reviews
5. @software-engineer implements code + `dev-notes.md`
6. Automated gate (lint/typecheck/build) → Tech Lead reviews
7. @qa-engineer produces `test-report.md`
8. Automated gate (tests + coverage) → Tech Lead reviews
9. @technical-writer syncs documentation
10. Pipeline complete — @release-engineer handles ongoing changes
