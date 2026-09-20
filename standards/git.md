# Git Conventions

## Commits
- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`
- Imperative mood: "add retry logic", not "added" or "adds"
- Body explains why, not a restatement of the diff

## Branches
- `feature/<short-description>`, `fix/<short-description>`
- No direct commits to main/master from agents — always via branch + PR,
  even in solo projects, so the diff is reviewable

## PRs (when applicable)
- One logical change per PR
- PR description references the SDLC stage/gate it corresponds to

---
> Adjust to match whatever branching model (trunk-based, gitflow, etc.)
> you actually use.
