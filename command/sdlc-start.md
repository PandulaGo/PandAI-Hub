---
description: Start a new project through the full SDLC pipeline
agent: orchestrator
---
If Docs/sdlc/<slug>/00-idea.md already exists (e.g. created by
`/sdlc-brief`), use it as-is and ignore $ARGUMENTS entirely — it's
already in the structured format @planner works from most efficiently.

Otherwise: if $ARGUMENTS references an existing file, read that file's
contents as the idea. If it's neither an existing structured idea nor a
file reference, treat $ARGUMENTS itself as raw idea text — this still
works, it just gives @planner more to infer (and costs more tokens)
than running `/sdlc-brief` first would have. Mention that option to the
user once, briefly, the first time this happens:

$ARGUMENTS
