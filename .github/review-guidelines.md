# Code review guidelines — Orders API

Rubric the `ai-review` gate applies to PRs in this repo. Review the diff against
these rules; call out blocking issues explicitly and separately from suggestions.

## About this service

🔧 CUSTOMIZE FOR REAL FIRM

- Trunk `main`, squash-only, linear history. Conventional Commit PR titles.

## Blocking — request changes if any apply

  🔧 CUSTOMIZE FOR REAL FIRM
  Consider lines below only as example, customize it for the real context

- **Money & correctness:** monetary values use `decimal`, never `double`/`float`.
  Rounding is explicit. No silent currency assumptions.
- **Input from the edge:** request DTOs are validated; no binding straight to EF
  entities; IDs and quantities range-checked before use.
- **SQL & injection:** all data access via EF Core / parameterized queries — no
  string-concatenated SQL. No raw user input in logs used for lookups.
- **Secrets & PII:** no connection strings, keys, or tokens in code or config;
  no logging of card data, emails, or full addresses.
- **Async correctness:** no `.Result` / `.Wait()` / `async void` (except event
  handlers); `CancellationToken` flowed through to I/O and DB calls.
- **Error handling:** no empty `catch`; no catching `Exception` just to swallow
  it; failures return a proper status, not a 200 with an error body.
- **Data safety:** EF migrations are additive/backward-compatible (no dropping
  or renaming a column in the same deploy that ships code using it).
- **Tests:** new business logic has unit tests; a bug fix includes a test that
fails without the fix.

## Non-blocking — leave as suggestions

- Naming, readability, dead code, magic numbers, missing XML docs on internals.
- Opportunities to simplify LINQ or extract a method. Note them; don't block.

## Style

- Be specific: cite file and line, and propose the concrete fix.
- If the PR is clean, say so plainly rather than inventing nitpicks.
