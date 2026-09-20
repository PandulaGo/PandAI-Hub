---
description: Update the cross-session/machine continuity ledger at the end of any coding session
agent: doc-agent
---
Do Responsibility 2 only (the context ledger) — inspect git status/log/diff
for what changed in this session and append a new entry to the top of
./Docs/context-ledger.md. Do not touch the architecture reference doc
for this command. The ledger is repo-wide, not tied to any single
project slug, so no slug is needed here. If $ARGUMENTS is given, treat
it as a free-text note on what this session focused on, not a slug.
