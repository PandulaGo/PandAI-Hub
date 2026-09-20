---
description: Show the current SDLC stage and gate history for a project
agent: orchestrator
---
Determine the slug the same way `/sdlc-init` does — git remote origin
name, falling back to the git top-level folder name, falling back to
the current directory name — UNLESS $ARGUMENTS explicitly names a
different project slug to check instead.

Read Docs/sdlc/<slug>/state.json and report: current stage, full gate
history (pass/fail per attempt), and which artifact files exist so far.
