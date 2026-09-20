<<<<<<< HEAD
# OpenCode Multi-Agent SDLC Framework
=======
# OpenCode - AI Agents Skills
>>>>>>> feeae4e793a4322097fe90864e1f58e0edcd49da

**Version 2.0** — see [CHANGELOG.md](./CHANGELOG.md) for what changed
from the original single-agent setup (v1.0.0).

A ready-to-use `.opencode/` config that runs your software ideas through
a full SDLC pipeline — Planning → Requirements → Design → Development →
Testing → Maintenance — with each stage on the model you assign, and a
verification gate between every stage before it's allowed to proceed.

## Why This Framework Exists

This came out of hitting the same wall repeatedly, not from theory.
After building several applications with OpenCode, the real limitation
wasn't the models — it was consistency. Variable and method naming
drifted between projects. Documentation happened however a given
session happened to leave it, or not at all. There was no repeatable
structure to start a new project from, so every new app began from a
slightly different, ad hoc setup.

This framework exists to fix that at the root: naming, architecture,
code style, testing, and git conventions are defined once and enforced
by every agent automatically — never re-explained, never drifting
between projects. Documentation is treated the same way: an as-built
architecture reference and a session continuity ledger that every
project produces the same way, not whatever a given session happened to
write down.

SDLC is the backbone that ties it together — Planning, Requirements,
Design, Development, Testing, and Maintenance as distinct, gated stages
— so a new idea moves through the same disciplined process every time,
instead of jumping straight to code.

I'm new to building this kind of framework-level tooling, so treat this
as a first version, not a finished one. It will keep evolving as real
usage surfaces gaps I didn't anticipate — a fair amount of what's
already in this README came from exactly that: hitting a real limitation
while actually using it, then closing the gap. If you spot something
that doesn't hold up in practice, that's expected at this stage, not a
sign the approach is wrong — feedback and issues are genuinely welcome.

## Reference Implementation — Calculator Web App

To make this framework verifiable rather than theoretical, I'm building
a small reference application with it end-to-end: a **web-based
calculator**. It's deliberately simple — the point isn't the app, it's
being able to see every stage of this framework's SDLC pipeline (idea →
plan → requirements → design → code → tests → docs) applied to
something small enough to read in full, so anyone evaluating this
framework has a concrete, practical scenario to check it against instead
of just this README's word for it.

**Repository:** _coming soon — link will be added here once the initial
pipeline run is complete._

What to look for once it's up:
- `Docs/sdlc/<slug>/00-idea.md` through `05-test-report.md` — the full
  paper trail for a real (if small) feature set, including any
  `NEEDS-INPUT` exchanges that came up along the way
- `Docs/architecture-and-api-reference.md` — the as-built reference,
  kept current by `@doc-agent`, not hand-written after the fact
- The actual folder scaffold from `/sdlc-init` (`Backend/`, `Frontend/`,
  `Configurations/`, etc.) populated with real, working code
- Commit history showing the pipeline's gate-driven revisions rather
  than a single "implement calculator" commit

## Quick Start

Follow these in order the first time. Steps 1–3 are one-time setup;
steps 4 onward repeat for every project.

### 1. Install the framework globally (once, ever)
Agents, commands, standards, and templates live in OpenCode's global
config — not copied into each project — so every project shares one
source of truth.
```powershell
# Windows
Copy-Item -Recurse .opencode\* "$env:APPDATA\opencode\"
```
```bash
# macOS / Linux
cp -r .opencode/* ~/.config/opencode/
```

### 2. Point it at your real models (once, ever)
Open `opencode.jsonc` in the global location and replace the placeholders:
- `openrouter/qwen/qwen3-coder` → your actual Qwen model id
- `openrouter/deepseek/deepseek-v3` → your actual DeepSeek model id
- `your-provider/big-pickle` → your actual "Big Pickle" model id/provider

### 3. Set your real conventions (once, ever)
Open `standards/naming.md`, `architecture.md`, `code-style.md`,
`testing.md`, `git.md` and replace the generic starter content with
your actual house rules. Every implementation agent, in every project,
reads these live — edit once here, never again per project.

### 4. Open your project (every project, once each)
```bash
cd F:\GitHub\my-actual-app     # the real repo, freshly created or existing
opencode
```
Desktop app: open that folder as your workspace instead of `cd`-ing.

### 5. Scaffold the repo (every project, once each)
```
/sdlc-init
```
Detects your project slug automatically from `git remote origin` (or
the folder name if there's no remote yet) — no name to type. Creates
`Backend/`, `Frontend/`, `Scripts/`, `Tests/`, `Configurations/`, and
`Docs/` (including the empty `Docs/sdlc/<slug>/` artifact skeleton).
Safe to run again later — it won't overwrite an existing scaffold.

### 6. Start the pipeline for an idea
Two ways in, pick one:

**Structured (recommended — cheaper and faster on Qwen/DeepSeek stages):**
```
/sdlc-brief
```
Presents a fill-in-the-blanks template (app type, platform, core
features, non-goals, constraints, etc.). Reply with it filled in —
blank or "not sure yet" is fine for anything you don't know yet. This
removes most of the guesswork `@planner`/`@requirements` would otherwise
have to do, which directly cuts their token usage and the number of
`NEEDS-INPUT` round-trips. Then run:
```
/sdlc-start
```

**Free-form (still works, costs a bit more):**
```
/sdlc-start a habit-tracking mobile app with streaks, daily reminders,
and a friend leaderboard. Offline-first, no backend server for v1.
```

Either way, this runs Planning → gate → Requirements → gate → Design →
gate (pauses for your review) → Development → gate (pauses again) →
Testing → gate → Maintenance, writing each stage's output to
`Docs/sdlc/<slug>/`.

### 7. Check in on progress anytime
```
/sdlc-status
```
Shows the current stage and full gate pass/fail history. Use
`/sdlc-next` instead of full-auto if you'd rather approve each
stage transition by hand while you're still building trust in the gates.

### 8. Keep documentation current as you go
```
/docs-sync       # as-built architecture/API reference — also runs
                         # automatically after Development, Testing, Maintenance
/wrap-session            # cross-machine session ledger — run at the end of
                         # ANY coding session, pipeline-driven or ad hoc
```

### 9. Switch modes when you're not building
```
@learn how do CRDTs handle offline conflict resolution?
```
Research/explanation only — no write access, doesn't load your
standards, can't touch project files.

### 10. Evolve your standards over time
```
/promote-standard always use optimistic locking for offline sync conflicts
```
Shows a diff against the relevant `standards/*.md` file and waits for
your confirmation before writing — nothing updates silently.

## What is `<slug>`?

Wherever you see `<slug>` in this README or in the agent/command files,
it's a **placeholder** — not something you type literally. It stands
for a short, filesystem-safe identifier for whichever project you're
currently in (e.g. `habit-tracker`), and it becomes the actual subfolder
name under `Docs/sdlc/<slug>/`.

**You never have to supply it by hand.** `/sdlc-init`, `/sdlc-start`,
`/sdlc-status`, `/sdlc-next`, and `/docs-sync` all detect it the same
way, automatically:
1. Git remote origin's repo name (e.g. from `github.com/you/habit-tracker.git`)
2. If no remote yet: the git repo's top-level folder name
3. If not a git repo at all: the current directory's name

Then it's sanitized — lowercased, spaces/underscores turned into
hyphens — to keep it consistent with the kebab-case filenames used
elsewhere in this framework.

Because it's derived from the repo/folder you're actually sitting in
when you run a command, and every command's file paths are relative to
that same working directory, you'll only ever need to type a slug
manually if you deliberately want to check a **different** project than
the one you're currently in — e.g. `/sdlc-status other-project` while
sitting inside a different repo. That's the exception, not the norm.

## Full Workflow — Start to End

This is the complete picture: every command in order, and exactly what
the orchestrator does behind each one. Steps 1–3 are one-time; step 4
runs once per repo; steps 5 onward are the actual project lifecycle.

### Step 1 — Install globally (once, ever)
```powershell
Copy-Item -Recurse .opencode\* "$env:APPDATA\opencode\"
```
Agents, commands, standards, and templates now apply to every project.

### Step 2 — Configure models (once, ever)
Edit the global `opencode.jsonc`: set real model IDs for Qwen,
DeepSeek, and your "Big Pickle" model, replacing the placeholders.

### Step 3 — Set your conventions (once, ever)
Edit `standards/naming.md`, `architecture.md`, `code-style.md`,
`testing.md`, `git.md` with your actual house rules. Every
implementation agent reads these live, in every project, forever.

### Step 4 — Open and scaffold a project (once per repo)
```bash
cd F:\GitHub\my-actual-app
opencode
```
```
/sdlc-init
```
**What happens:** detects the slug (git remote → git folder name → cwd
name), checks it isn't already initialized, then creates `Backend/`,
`Frontend/`, `Scripts/`, `Tests/`, `Configurations/` (with your real
`.gitignore`/`.dockerignore`/`.env`/`appsettings.*` templates copied in
verbatim), and `Docs/` — including the empty `Docs/sdlc/<slug>/`
skeleton (`00-idea.md` through `05-test-report.md`, `state.json` with
`"stage": "not_started"`).

### Step 5 — Capture the idea
```
/sdlc-brief
```
**What happens:** the orchestrator shows you the fill-in-the-blanks
intake template (app type, platform, core features, non-goals,
constraints, etc.) and waits for your reply. Once you answer, it writes
your filled-in template into `Docs/sdlc/<slug>/00-idea.md`, preserving
the section structure exactly. (Skip this and just run `/sdlc-start
<your idea in plain English>` instead if you'd rather — it works, it
just costs more tokens on the Qwen/DeepSeek stages since they have more
to infer.)

### Step 6 — Run the pipeline
```
/sdlc-start
```
**What the orchestrator does, stage by stage:**

1. **Detects the slug** the same way `/sdlc-init` did.
2. **Checks for an existing `00-idea.md`** — uses the one `/sdlc-brief`
   already created rather than asking again.
3. **Updates `state.json`** from `"not_started"` to `"plan"`.
4. **For each of the 5 gated stages, in order:**
   - Prints a status line so you always know what's active:
     `▶ Stage 2/6 — Requirements — handing off to @requirements (Qwen)...`
   - Calls that stage's agent with the idea file **and every prior
     stage's output** (not just the immediately preceding one).
   - **If the agent responds with `NEEDS-INPUT: <question>`** instead of
     its normal output: the orchestrator relays that question to you
     directly, waits for your answer, and re-invokes the *same* agent
     with your answer added — this is not a failure, just a pause.
   - Once the agent produces real output, calls that stage's
     gatekeeper — always a **different model** than the one that just
     produced the work, so review isn't the model agreeing with itself.
   - **`GATE: PASS`** → records it in `state.json`, gives you a short
     summary of what was produced. For **Design** and **Development**
     specifically, it pauses and asks whether to continue automatically
     or let you review the artifact first; Planning, Requirements, and
     Testing proceed on their own.
   - **`GATE: FAIL`** → sends the exact failure list back to the same
     agent for revision, logs the attempt, and re-checks. After **3**
     consecutive failures on one stage, it stops retrying and asks you
     what to do instead of looping forever.
5. **After the Development gate passes, and again after Testing passes:**
   calls `@doc-agent` to sync `Docs/architecture-and-api-reference.md`
   against the real code — not blocking, just a sync, and a no-op if
   nothing user-facing changed.
6. **If you ask for something new mid-flight** (a feature request while
   any stage is actively running): the orchestrator pauses immediately,
   tells you which stage is active and why timing matters, and asks you
   to choose — defer it to a `@maintainer` pass after this run finishes,
   or inject it now (rolling back to Requirements with the current
   requirements/design/dev-notes preserved as `*.pre-change.md` before
   being revised, then re-cascading through Design → Development →
   Testing so nothing stale gets tested). It never decides this for you.
7. **Testing gate passes** → the formal pipeline run is complete. The
   app exists in `Backend/`/`Frontend/`, fully documented in
   `Docs/sdlc/<slug>/` and `Docs/architecture-and-api-reference.md`.

### Step 7 — Check in whenever you want
```
/sdlc-status
```
Reports the current stage and the full gate pass/fail history — useful
mid-run or long after, to confirm exactly where things stand.

### Step 8 — Ongoing maintenance (after the first release)
```
/sdlc-next        # or just describe a change and @maintainer picks it up
```
**What happens:** `@maintainer` handles bug fixes and small enhancements
directly, updating `02-requirements.md`/`03-design.md`/`04-dev-notes.md`
if the change affects what they describe. Anything that would
materially change the requirements or design triggers a `NEEDS-INPUT`
asking whether to patch minimally or re-run the pipeline from
Requirements/Design instead of drifting from the documented design.

### Step 9 — Keep documentation current forever
```
/docs-sync         # manual as-built architecture/API doc refresh, anytime
/wrap-session      # run at the END of every session, pipeline or ad hoc
```
`/wrap-session` is the one command that isn't tied to the pipeline at
all — run it whenever you stop working, so `Docs/context-ledger.md` has
what changed, what's still broken, and exactly how to pick back up,
even from a different machine.

### The whole thing, one screen

```
/sdlc-init  ──▶  /sdlc-brief  ──▶  /sdlc-start
                                       │
                     ┌─────────────────┴─────────────────┐
                     │   for each stage:                  │
                     │   announce → call agent             │
                     │     ├─ NEEDS-INPUT? → ask you, retry │
                     │   → call gatekeeper (different model)│
                     │     ├─ FAIL → revise, retry (max 3)  │
                     │     └─ PASS → log, maybe pause       │
                     └─────────────────┬─────────────────┘
        Planning → Requirements → Design → Development → Testing
                              (Qwen)   (Qwen)  (DeepSeek)  (Big Pickle)  (Big Pickle)
                                                    │            │
                                              @doc-agent    @doc-agent
                                              (architecture sync, both times)
                                                                 │
                                                          Testing gate PASS
                                                                 │
                                                          Maintenance (ongoing)
                                                       /sdlc-next, /docs-sync,
                                                          /wrap-session
```

### Full command reference

| Command | When to use it | Requires a typed argument? |
|---|---|---|
| `/sdlc-init` | Once, right after creating a repo | No — slug auto-detected |
| `/sdlc-brief` | Before starting, to capture the idea in structured form | No — interactive prompt |
| `/sdlc-start` | Kick off (or resume) the pipeline | No, unless skipping `/sdlc-brief` |
| `/sdlc-status` | Check current stage + gate history anytime | No — defaults to current project |
| `/sdlc-next` | Advance exactly one stage instead of full-auto | No |
| `/docs-sync` | Manually refresh the as-built architecture doc | No |
| `/wrap-session` | End of any session, pipeline or ad hoc | No |
| `/standards` | Review current global naming/architecture/style/testing/git rules | No |
| `/promote-standard <text>` | Turn something you just learned into a permanent rule | Yes — what to promote |
| `@learn <question>` | Research/explain something — no writes, no standards loaded | N/A |

## What's inside

```
.opencode/
├── opencode.jsonc        # agent registry: which model runs which stage
├── AGENTS.md             # minimal global instruction (kept deliberately small)
├── standards/            # naming, architecture, code-style, testing, git —
│   │                       loaded by every implementation agent, edited once,
│   │                       applied to every project automatically
│   ├── naming.md
│   ├── architecture.md
│   ├── code-style.md
│   ├── testing.md
│   └── git.md
├── templates/            # doc-agent's output templates + repo scaffold files —
│   │                       real files, referenced by other agents/commands
│   │                       rather than retyped, so nothing drifts
│   ├── architecture-and-other-details.md
│   ├── context-ledger.md
│   ├── idea-intake-template.md
│   └── scaffold/
│       ├── gitignore.txt
│       ├── dockerignore.txt
│       ├── env.txt
│       ├── appsettings.backend.json
│       └── appsettings.frontend.json
├── agent/
│   ├── orchestrator.md   # primary agent — drives the whole pipeline
│   ├── planner.md        # Stage 1 — Planning
│   ├── requirements.md   # Stage 2 — Requirements
│   ├── designer.md       # Stage 3 — Design
│   ├── developer.md      # Stage 4 — Development
│   ├── tester.md         # Stage 5 — Testing
│   ├── maintainer.md     # Stage 6 — Maintenance (ongoing)
│   ├── gatekeeper.md     # reused verification agent between every stage
│   ├── doc-agent.md      # as-built docs + cross-session ledger (see below)
│   └── learn.md          # separate primary agent for research/learning —
│                            no standards, no write access, structurally
│                            walled off from your project work
└── command/
    ├── sdlc-init.md        # /sdlc-init — scaffold a brand-new repo (run once, first)
    ├── sdlc-brief.md        # /sdlc-brief — structured idea intake (cheaper than free-form)
    ├── sdlc-start.md      # /sdlc-start <idea or file> — kick off a project
    ├── sdlc-next.md       # /sdlc-next — advance one stage manually
    ├── sdlc-status.md     # /sdlc-status — see current stage + gate history
    ├── standards.md       # /standards — show current global conventions
    ├── promote-standard.md # /promote-standard <text> — turn a learning into a rule
    ├── docs-sync.md        # /docs-sync — manual as-built architecture doc sync
    └── wrap-session.md     # /wrap-session — update the ledger at end of any session
```

## Install

**This machinery is designed to be global, so install it once:**

```powershell
# Windows (PowerShell)
Copy-Item -Recurse .opencode\* "$env:APPDATA\opencode\"
```
```bash
# macOS / Linux
cp -r .opencode/* ~/.config/opencode/
```

This puts `opencode.jsonc`, `AGENTS.md`, `standards/*.md`, `templates/*`,
`agent/*.md`, and `command/*.md` in the global config, so every agent,
every command, and your naming/architecture/style conventions are
available in **every** project you open with OpenCode — no per-project
copying, no drift between projects.

**Do NOT copy a `project/` or `Docs/sdlc/` folder globally.** There isn't
one shipped — it gets created per-project the first time you run
`/sdlc-init` in that repo. When you do:

```bash
cd F:\GitHub\my-actual-app     # cd into the real (empty or fresh) project repo first
opencode
/sdlc-init
/sdlc-start a habit-tracking app with streaks and reminders
```

OpenCode resolves the agents' relative paths (`Docs/sdlc/<slug>/...`,
`Docs/...`) against your **current working directory** — i.e. that
project's own repo — even though the agent definitions themselves came
from the global config. So each project's SDLC artifacts and docs live
inside that project, get committed with its own git history, and never
mix with another project's artifacts.

**Config merging:** OpenCode merges global and project config rather
than one replacing the other — project settings only override global on
actual conflicts. So if a specific project needs an exception (a
different model, an overridden standard), drop a local `opencode.jsonc`
or `AGENTS.md` in that project's own repo root; it layers on top of the
global one without you touching the global files. See "Per-project
standard overrides" below.

## Before you run it

Edit `opencode.jsonc` (in the global location, once) and replace the
placeholder model strings:

- `openrouter/qwen/qwen3-coder` → your actual Qwen model id
- `openrouter/deepseek/deepseek-v3` → your actual DeepSeek model id
- `your-provider/big-pickle` → your actual "Big Pickle" model id/provider

Also edit `standards/*.md` — the shipped content is a sensible generic
starting point, not your actual conventions. Replace it with your real
naming/architecture/style/testing/git rules. This is the one-time setup;
you won't need to touch agent or command files again after this.

## Usage

**Initialize a brand-new repo (run once, first):**
```
/sdlc-init
```
No project name needed — it detects the slug from your git remote
origin (falling back to the repo's top-level folder name, then the
current directory, if there's no remote yet). This creates the full
folder scaffold — `Backend/`, `Frontend/`, `Scripts/`, `Tests/`,
`Configurations/`, `Docs/` — plus the empty `Docs/sdlc/<slug>/` artifact
skeleton (`00-idea.md` through `05-test-report.md`, `state.json`), ready
for the pipeline to fill in. It's idempotent — running it again on an
already-initialized repo reports what exists instead of overwriting it.

**Start a new project:**
```
/sdlc-start a habit-tracking mobile app with streaks, daily reminders,
and a friend leaderboard. Offline-first, no backend server for v1.
```
or, for a longer idea already written out:
```
/sdlc-start @my-idea.md
```

The orchestrator detects the same slug `/sdlc-init` used, fills in
`Docs/sdlc/<slug>/00-idea.md` (or reuses it if `/sdlc-init` already
created a placeholder), then runs Planning → gate → Requirements → gate
→ Design → gate (pauses here for your review) → Development → gate
(pauses here too) → Testing → gate → Maintenance.

After `/sdlc-init` and a full pipeline run, your project repo (not the
global config) looks like:
```
my-actual-app/                    # your real project repo, wherever it lives
├── .gitignore
├── .dockerignore
├── README.md
├── docker-compose.yml
├── Backend/                      # actual application code, written by @developer
├── Frontend/
├── Scripts/
├── Tests/
├── Configurations/
│   ├── .env
│   ├── appsettings.backend.json
│   └── appsettings.frontend.json
└── Docs/
    ├── architecture-and-api-reference.md   # written/kept current by doc-agent
    ├── context-ledger.md                   # written/kept current by doc-agent
    └── sdlc/
        └── habit-tracker/
            ├── 00-idea.md
            ├── 01-plan.md
            ├── 02-requirements.md
            ├── 03-design.md
            ├── 04-dev-notes.md
            ├── 05-test-report.md
            └── state.json
```
Commit `Docs/` (including `Docs/sdlc/`) along with your code — that's
the point of putting them in the repo instead of the global config.

**Check progress:**
```
/sdlc-status
```
Detects the current project automatically — same slug logic as
`/sdlc-init`. Pass a slug explicitly (`/sdlc-status other-project`) only
if you want to check a project other than the one you're currently in.

**Advance one stage at a time** (useful while tuning prompts, instead of
letting it run full-auto):
```
/sdlc-next
```

**Just research/learn something** (switch to the `learn` primary agent
with Tab, or):
```
@learn explain how CRDTs handle offline conflict resolution
```
This agent never touches your standards or your project files.

**Turn something you learned into a permanent standard:**
```
/promote-standard always use optimistic locking for offline sync conflicts
```
This shows a diff against the relevant `standards/*.md` file and waits
for your confirmation before writing anything.

## Design decisions baked into this framework

- **Every stage agent asks instead of guessing on material ambiguity.**
  If @planner, @requirements, @designer, @developer, @tester, or
  @maintainer hits something that would genuinely change its output
  depending on the answer, it stops and emits `NEEDS-INPUT: <question>`.
  The orchestrator relays this to you directly, waits for your answer,
  then resumes that same agent — it never falls through the pipeline
  silently just because a gate eventually would have caught it. Minor,
  non-blocking judgment calls still get made and briefly noted in the
  artifact, so you're not interrupted for every trivial decision — only
  ones that matter.
- **You always know which agent is active.** Before every stage handoff
  and every gatekeeper check, the orchestrator prints a status line
  (`▶ Stage 3/6 — Design — handing off to @designer (DeepSeek)...`) so
  you can tell at a glance where the pipeline is, without asking.
- **Mid-pipeline feature requests never get silently absorbed.** If you
  ask for something new while a stage is actively running — say, a new
  feature while @tester is working — the orchestrator pauses immediately
  and asks you to choose: defer it to a @maintainer pass after this run
  finishes, or inject it now by rolling back to Requirements (with the
  affected stages' current artifacts preserved as `*.pre-change.md`
  before being revised) and re-running Design → Development → Testing
  against the expanded scope. It never decides this for you.
- **Structured intake keeps the Qwen/DeepSeek stages cheap.** `/sdlc-brief`
  trades free-form prose for a fixed template (app type, core features,
  non-goals, constraints, etc.). Filled fields are treated as ground
  truth by `@planner`/`@requirements` — nothing gets re-inferred — which
  means fewer reasoning tokens spent per call and far fewer `NEEDS-INPUT`
  round-trips than starting from an open-ended idea. Free-form
  `/sdlc-start <idea>` still works; it just costs more on the lighter
  models the earlier stages run on.

- **Gates always use a different model than the stage they check**
  (e.g. DeepSeek gates Qwen's planning output, Qwen gates DeepSeek's
  design output) so review isn't just a model agreeing with itself.
- **Standards live once, globally**, referenced by file (`{file:...}`)
  from every implementation agent's prompt — update a standard in one
  place and every future project, every future agent call, picks it up
  automatically. Nothing is duplicated per project.
- **Learning is structurally separated from building.** The `learn`
  agent has no write/edit/bash tools and never loads `standards/*.md`,
  so switching into "explain this to me" mode can't accidentally bias
  toward your house style or touch your codebase.
- **Design and Development stages pause for human review by default**
  (configured in `orchestrator.md`); Planning, Requirements, and Testing
  proceed automatically on gate PASS. Adjust this in `orchestrator.md`
  as you build trust in the pipeline.
- **Artifacts are files, not chat context** — `Docs/sdlc/<slug>/00-idea.md`
  through `05-test-report.md` are the actual contract between stages,
  so nothing depends on everything fitting in one conversation's memory.
- **`doc-agent` covers two distinct jobs the pipeline itself doesn't**,
  each with its own trigger:
  - *As-built architecture/schema/API reference* (`Docs/architecture-and-api-reference.md`) —
    what the code actually is right now, kept separate from
    `03-design.md` (what was planned). Triggered automatically after
    Development, Testing, and Maintenance; `/docs-sync` runs it manually.
  - *Cross-session/machine continuity ledger* (`Docs/context-ledger.md`) —
    git-based session handoff notes. This is session-based, not
    stage-based, so it's triggered by `/wrap-session` at the end of
    *any* coding session, whether it went through the pipeline or not —
    not wired to the orchestrator's gates.
  - `doc-agent` has `bash: ask` (not fully denied) specifically because
    the ledger requires read-only git inspection (`git status/log/diff`)
    to work — you approve each call, it still can't run anything
    destructive on its own.

## Per-project standard overrides

If one project needs to deviate from a global standard (e.g. a legacy
codebase stuck on snake_case), don't edit the global files — add a
project-level `AGENTS.md` in that project's own `.opencode/` or repo root:

```markdown
# Overrides to global standards for this project only
- Variables use snake_case (legacy codebase convention), not camelCase
```

OpenCode merges project-level `AGENTS.md` on top of the global one, so
the exception stays contained instead of leaking into every other project.

## Multi-project isolation

Running this same global setup across many projects doesn't mix their
data — global config is the *behavior* (agent instructions, standards,
model assignments), never the *content*. Every agent writes to paths
relative to whatever directory OpenCode was launched from
(`Docs/sdlc/<slug>/...`, `Docs/...`), so `cd`-ing into `project-a` and
running `/sdlc-start` writes only into `project-a`; doing the same later
in `project-b` writes only into `project-b`. Same recipe, separate
bowls.

The only real failure mode is running a command from the **wrong
directory** — not a framework issue, just a "did I `cd` correctly"
mistake. `/sdlc-status` right after opening a project is a quick
sanity check: if it reports the wrong slug or "no such project," you're
in the wrong folder before anything gets written. Two projects
detecting the same slug (e.g. both repos named `api`) is also harmless
— the slug is only a subfolder name inside each project's own folder,
not a shared key across projects.

Editing `standards/*.md` or an agent's prompt globally *does* change
behavior for every project going forward — that's the intended design —
but it never rewrites artifacts a past run already produced elsewhere.
