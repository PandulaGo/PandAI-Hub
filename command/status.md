---
description: Show current SDLC pipeline status and optionally advance to next stage
agent: coordinator
---
Show the current state of the SDLC pipeline and optionally advance.

## Usage

```
/status                    # Show current state
/status advance            # Advance to next stage
/status redo               # Re-run current stage
/status history            # Show full gate history
```

## Step 1 — Determine the project slug

Same as `/start` — git remote origin name, falling back to folder name.

UNLESS $ARGUMENTS explicitly names a different project slug to check.

## Step 2 — Read state

Read `Docs/sdlc/<slug>/state.json` and report:

```
Project: habit-tracker
Current Stage: 2/3 — Implementation
Gate History:
  ✅ Analysis — PASS (2026-01-15 10:30)
  🔄 Implementation — IN PROGRESS
  ⬜ Verification — NOT STARTED

Artifacts:
  ✅ analysis.md (2.3 KB)
  🔄 dev-notes.md (in progress)
  ⬜ test-report.md (not started)

Last Updated: 2026-01-15 11:45
```

## Step 3 — Handle actions

### If `advance`:
1. Check current stage
2. Run next stage's agent
3. Run gate (agent-based for Analysis, automated for others)
4. Stop and report result — do NOT proceed further automatically

### If `redo`:
1. Check current stage
2. Re-run current stage's agent
3. Run gate again
4. Stop and report result

### If `history`:
Show full gate history from state.json with timestamps

### If no action:
Just show status (Step 2)

## Rules

- Always report which stage is active
- Always show which artifacts exist
- Never auto-advance without explicit `advance` command
- Report gate history (pass/fail with timestamps)
