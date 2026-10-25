# AGENTS.md — Global Instructions

You operate as one of 8 specialized agents in the AI Agents Skills SDLC framework.

## Agent Types

### SDLC Pipeline Agents (in `agent/sdlc/`)
- **coordinator** — Drives the 3-stage pipeline (Analysis → Implementation → Verification)
- **solution-architect** — Produces combined analysis (plan + requirements + design)
- **software-engineer** — Implements code per the approved design
- **qa-engineer** — Tests implementation against requirements
- **release-engineer** — Handles post-release maintenance and changes
- **auditor** — Strict verification gate between stages

### Support Agents (in `agent/`)
- **technical-writer** — Maintains architecture docs and session ledger
- **researcher** — Research assistant (no standards, no write access)

## Implementation vs. Research

- **Implementation agents** (all SDLC agents + technical-writer): Follow standards in `standards/` folder
- **Research agent** (researcher): Does NOT load or reference `standards/*.md`

## Standards Loading

When starting any SDLC pipeline:
1. Always load `standards/universal/*` (applies to every project)
2. Load stack-specific standards based on `--stack` flag:
   - `standards/dotnet/*` for .NET projects
   - `standards/node/*` for Node.js projects
   - `standards/python/*` for Python projects
   - `standards/angular/*` for Angular projects
   - `standards/react/*` for React projects
3. Validate all requested stacks exist before proceeding

## Pipeline Flow

```
/start --stack <stacks> <idea>
         ↓
@coordinator validates stacks, creates scaffold
         ↓
Stage 1: @solution-architect → @auditor → Tech Lead review
         ↓
Stage 2: @software-engineer → Automated gate → Tech Lead review
         ↓
Stage 3: @qa-engineer → Automated gate → Tech Lead review
         ↓
@technical-writer syncs docs → DONE
         ↓
Maintenance: @release-engineer (ongoing)
```

## Commands

| Command | Description |
|---------|-------------|
| `/start` | Initialize project and start pipeline |
| `/status` | Show current stage, optionally advance |
| `/docs` | Sync all documentation |
| `/research` | Research topics, view/promote standards |
