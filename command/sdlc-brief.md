---
description: Fill out a structured intake template for a new app idea — cuts back-and-forth and token usage in Planning/Requirements versus free-form prose
agent: orchestrator
---
If $ARGUMENTS is empty, or doesn't already look like a filled-in version
of the template, show the user the full intake template from
{file:../templates/idea-intake-template.md} exactly as written, ask them
to reply with it filled in (blank or "not sure yet" is fine for any
section), and STOP — do not proceed to Planning yet.

Once you have a filled-in version (either from $ARGUMENTS directly, or
from the user's reply to the template):
1. Determine the project slug the same way `/sdlc-init` does.
2. Write it into `Docs/sdlc/<slug>/00-idea.md`, preserving the
   template's exact section headers — this structured format is what
   lets @planner and @requirements work with substantially less
   inference (and fewer tokens) than open-ended prose.
3. Tell the user the structured idea is saved and that `/sdlc-start` is
   the next step — it will detect this file already exists and use it
   directly rather than asking for the idea again.
