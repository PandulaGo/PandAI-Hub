# Code Style

## General
- Small functions: one responsibility, ideally under ~40 lines
- Prefer early returns over deep nesting
- No magic numbers/strings — name them as constants

## Comments
- Explain *why*, not *what* the code does
- No commented-out code left in commits
- TODOs must include an owner or ticket reference: `// TODO(john): ...`

## Formatting
- The project's configured formatter/linter is the final authority
  (Prettier, Black, gofmt, etc.) — do not hand-format against it
- Line length, quote style, semicolons: whatever the linter config says

## Error handling
- Never swallow exceptions silently
- Always log with enough context to reproduce (inputs, not secrets)

## Imports
- Group and order: standard library -> third-party -> internal
- No wildcard imports

---
> Point this at your actual linter config once you have one, and keep
> only the rules here that a linter can't enforce automatically.
