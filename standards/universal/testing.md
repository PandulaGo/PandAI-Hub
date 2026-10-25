# Testing Conventions

## Naming
- Test files mirror source files: `user-service.ts` -> `user-service.test.ts`
- Test names describe behavior, not implementation:
  `it("rejects login with an expired token")`, not `it("test2")`

## Structure
- Arrange / Act / Assert, with blank lines separating the three
- One logical assertion focus per test (multiple `expect`s are fine if
  they verify the same behavior)

## Coverage expectations
- New business logic requires unit tests before the Testing gate can pass
- Integration tests required for anything touching a DB, external API,
  or filesystem
- Don't chase 100% coverage on trivial glue code

## Test data
- Use factories/builders for fixtures, not hand-copied JSON blobs
- No real credentials or production data in test fixtures

---
> Tune coverage thresholds and test-runner specifics (Jest, pytest, etc.)
> to your stack.
