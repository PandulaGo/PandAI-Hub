# Architecture & Project Structure

## Layering
- Separate concerns into: presentation / application (use-cases) / domain
  (business logic) / infrastructure (DB, external APIs)
- Domain layer must not import from infrastructure or presentation

## Folder structure (default — override per project if needed)
```
src/
  domain/
  application/
  infrastructure/
  presentation/
tests/
  unit/
  integration/
  e2e/
```

## Dependency direction
- Dependencies point inward: presentation -> application -> domain
- Infrastructure implements interfaces defined in domain/application,
  never the reverse

## Error handling
- Domain/application layers throw typed errors, never raw strings
- Presentation layer is the only place that translates errors into
  HTTP responses / UI messages

## Configuration
- All environment-specific values come from config/env, never hardcoded
- Secrets are never committed; use .env.example to document required keys

---
> Adjust folder names and layering rules to match your actual stack
> (this looks different for a mobile app vs. a backend service). Keep
> this file as the single source of truth for structure decisions.
