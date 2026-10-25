---
description: Sync all documentation (architecture docs + session ledger)
agent: technical-writer
---
Sync all project documentation in one command.

## Usage

```
/docs              # Sync architecture + session ledger
/docs arch         # Sync architecture doc only
/docs session      # Update session ledger only
```

## Responsibility 1 — Architecture & API Reference

Sync `Docs/architecture-and-api-reference.md` against the current source code:

1. Inspect the ACTUAL codebase (directory tree, models, routes, controllers)
2. Update the architecture doc to reflect what IS, not what was planned
3. If it diverges from `Docs/sdlc/<slug>/analysis.md`, note the divergence explicitly
4. Do not restructure the template — fill it out as designed

## Responsibility 2 — Session & Machine Continuity Ledger

Update `Docs/context-ledger.md`:

1. Run `git status`, `git log -n 5`, and `git diff` to see what changed this session
2. Append a new session block to the TOP of the ledger
3. Never delete or rewrite past entries
4. Include:
   - What was done this session
   - Files modified/created
   - Current state (working/blocked/complete)
   - Next actions (for machine switching)

## Rules

- If $ARGUMENTS is "arch", do Responsibility 1 only
- If $ARGUMENTS is "session", do Responsibility 2 only
- If no $ARGUMENTS, do both
- The ledger is repo-wide, not tied to any single project slug
- If nothing meaningful changed, say so plainly
- Never touch application source code
