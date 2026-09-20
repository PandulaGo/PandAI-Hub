---
description: Manually advance the current project one SDLC stage instead of running full-auto
agent: orchestrator
---
Determine the slug the same way `/sdlc-init` does — git remote origin
name, falling back to the git top-level folder name, falling back to
the current directory name — UNLESS $ARGUMENTS explicitly names a
different project slug to check instead.

Advance that project by exactly one stage: run the next stage's agent,
then its gatekeeper, then stop and report the result — do not proceed
further automatically, regardless of pass/fail.
