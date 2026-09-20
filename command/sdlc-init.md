---
description: Scaffold the full project folder structure (Backend/Frontend/Docs/etc.) and the SDLC artifact skeleton for a brand-new repo
agent: orchestrator
---
Initialize this repo as an SDLC-managed project. Run once, right after
the repo is created — before `/sdlc-start`.

## Step 1 — Determine the project slug
1. `git rev-parse --is-inside-work-tree`
   - If this fails (not a git repo): use the current directory's name as
     the slug and warn the user that no git repo was detected.
2. If it is a git repo: `git remote get-url origin`
   - If this succeeds: take the last path segment of the URL, strip a
     trailing `.git`, and use that as the slug candidate. This is the
     canonical repo name even if the local folder was renamed.
   - If it fails (no remote yet): `git rev-parse --show-toplevel` and use
     the basename of that path as the slug candidate instead.
3. Sanitize the slug candidate: lowercase it, replace spaces and
   underscores with hyphens, strip any character that isn't `a-z0-9-`.
4. Report the result before doing anything else, e.g.:
   `Detected project slug: habit-tracker (from git remote origin)`

## Step 2 — Check for an existing scaffold (idempotency)
If `Docs/sdlc/<slug>/` already exists and contains any of the files
below, STOP and report that this project is already initialized — list
what's already there. Do not overwrite anything. If the user explicitly
asks to re-initialize, confirm once before proceeding.

## Step 3 — Create the folder structure
```
Backend/                (+ .gitkeep so git tracks the empty folder)
Frontend/               (+ .gitkeep)
Scripts/                (+ .gitkeep)
Tests/                  (+ .gitkeep)
Configurations/
  .env
  appsettings.backend.json
  appsettings.frontend.json
Docs/
  architecture-and-api-reference.md   (seed with the structure from
                                        templates/architecture-and-other-details.md,
                                        left unfilled — @doc-agent fills
                                        it in after Development/Testing)
  context-ledger.md                   (seed with the structure from
                                        templates/context-ledger.md,
                                        left unfilled — @doc-agent
                                        appends entries via /wrap-session)
  sdlc/
    <slug>/
      00-idea.md
      01-plan.md
      02-requirements.md
      03-design.md
      04-dev-notes.md
      05-test-report.md
      state.json
.gitignore
.dockerignore
README.md
docker-compose.yml
```

## Step 4 — File contents
- `.gitignore` ← copy verbatim from `{file:../templates/scaffold/gitignore.txt}`
- `.dockerignore` ← copy verbatim from `{file:../templates/scaffold/dockerignore.txt}`
- `Configurations/.env` ← copy verbatim from `{file:../templates/scaffold/env.txt}`
- `Configurations/appsettings.backend.json` ← copy verbatim from `{file:../templates/scaffold/appsettings.backend.json}`
- `Configurations/appsettings.frontend.json` ← copy verbatim from `{file:../templates/scaffold/appsettings.frontend.json}`
- `README.md`, `docker-compose.yml`: create empty, to be filled in later.
- `Docs/architecture-and-api-reference.md`, `Docs/context-ledger.md`:
  copy the section headers and AI-instruction blocks from the
  corresponding files under `templates/`, with all the actual content
  fields left as placeholders — do not fabricate real architecture/API
  details, there's no code to inspect yet.
- Each `0X-*.md` under `Docs/sdlc/<slug>/`: create with a single line:
  `_Not yet generated. Run /sdlc-start to populate the pipeline._`
- `Docs/sdlc/<slug>/state.json`:
  ```json
  { "slug": "<slug>", "stage": "not_started", "history": [] }
  ```

## Rules
- Never touch `Backend/` or `Frontend/` beyond creating the empty folder
  — no starter app code, no assumptions about framework/language.
- Never overwrite an existing file — if something in this list already
  exists with content, skip it and report that you skipped it.
- Finish with a short summary: the detected slug, what was created, and
  a reminder that `/sdlc-start <idea>` is the next step.
