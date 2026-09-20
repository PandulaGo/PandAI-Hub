# Changelog

All notable changes to this OpenCode SDLC framework are documented here.
Format loosely follows [Keep a Changelog](https://keepachangelog.com/),
versioned with [Semantic Versioning](https://semver.org/).

## [2.0.0] — 2026-09-20

### Added
- Full six-stage SDLC pipeline: Planning, Requirements, Design,
  Development, Testing, Maintenance — each on its own assigned model
  (Qwen, Qwen, DeepSeek, Big Pickle, Big Pickle, Big Pickle).
- `@gatekeeper-deepseek` / `@gatekeeper-qwen` verification gates between
  every stage, always run on a different model than the stage they check,
  so review is never the model agreeing with itself.
- `@orchestrator` primary agent driving the whole pipeline end to end,
  including stage visibility announcements and gate pass/fail handling.
- Global `standards/` (naming, architecture, code-style, testing, git) —
  loaded by every implementation agent, edited once, applied to every
  project automatically.
- `@learn` — a structurally separate research/learning agent with no
  write access and no standards loaded, so research never leaks into or
  biases project work.
- `@doc-agent` reworked with two explicit, separately-triggered
  responsibilities: as-built architecture/API reference sync, and a
  cross-session/machine continuity ledger. Its `bash` permission was
  fixed from `deny` to `ask` — the ledger's own instructions require
  `git status`/`log`/`diff`, which `deny` silently blocked in v1.
- `/sdlc-init` — scaffolds a full repo (`Backend/`, `Frontend/`,
  `Scripts/`, `Tests/`, `Configurations/`, `Docs/`) plus the
  `Docs/sdlc/<slug>/` artifact skeleton, with the project slug
  auto-detected from git (remote origin → top-level folder → cwd
  fallback) — never typed manually.
- `/sdlc-brief` — structured intake template, reducing inference load
  (and token cost) on the Qwen/DeepSeek stages versus free-form prose.
- `/sdlc-start`, `/sdlc-next`, `/sdlc-status`, `/docs-sync`,
  `/wrap-session`, `/promote-standard`, `/standards` — full command set,
  all with consistent auto-detected project slugs.
- `NEEDS-INPUT` escalation: every stage agent asks the user directly on
  material ambiguity instead of silently guessing or only logging it in
  an "Open Questions" note that might go unread.
- Mid-pipeline change handling: a new feature request while a stage is
  actively running pauses the orchestrator and offers defer-vs-inject,
  rather than being silently absorbed by whichever agent is active.

### Changed
- Config moved from project-local to OpenCode's **global** config
  (`%APPDATA%\opencode\` / `~/.config/opencode/`) — one install applies
  to every project; only `Docs/sdlc/<slug>/` and `Docs/` stay
  per-project, resolved relative to the working directory.

## [1.0.0] — prior to this framework

The starting point: a minimal `opencode.jsonc` with a global `model` /
`small_model`, a couple of `instructions` files loaded for every agent,
a single `doc-agent` (documentation sync, `bash: deny`), and an override
of the built-in `plan` agent's model. No pipeline, no standards, no
per-project scaffolding, no gates, no auto-detected project identity.
