# Naming Conventions

## Variables
- camelCase for variables and function parameters
- Booleans prefixed with is / has / should (isActive, hasPermission, shouldRetry)
- No abbreviations except widely known ones (id, url, config, req, res)

## Functions / Methods
- verbNoun pattern (getUser, calculateTotal, validateInput)
- Private/internal helpers prefixed with _ (or the language's native private modifier)
- Async functions don't need an "Async" suffix — rely on the language's own
  signature/typing to signal that

## Files
- kebab-case for filenames (user-service.ts, order-repository.py)
- One exported concept per file; filename matches its primary export

## Classes / Types / Interfaces
- PascalCase
- Interface prefix (I...) only if the project's language convention already
  uses it (e.g. C#) — never invent one for languages that don't

## Constants
- SCREAMING_SNAKE_CASE for true constants (MAX_RETRIES, DEFAULT_TIMEOUT_MS)
- camelCase for config objects that group several constants

## Database / API
- snake_case for DB columns and JSON API fields, unless the platform
  convention says otherwise (e.g. GraphQL uses camelCase)
- Table names: plural, snake_case (users, order_items)

---
> Customize this file with your actual preferred conventions. Every
> implementation agent loads this automatically — edit once here,
> not per project.
