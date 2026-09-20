---
description: Manually trigger an as-built architecture/API doc sync for a project
agent: doc-agent
---
Determine the slug the same way `/sdlc-init` does — git remote origin
name, falling back to the git top-level folder name, falling back to
the current directory name — UNLESS $ARGUMENTS explicitly names a
different project slug to check instead.

Do Responsibility 1 only (architecture-and-api-reference.md) — sync it
against the current source code, noting any deviation from
Docs/sdlc/<slug>/03-design.md. Do not touch the context ledger for this
command.
