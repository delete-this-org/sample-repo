# Versioning — how to write commits so releases work

Versions are never set by hand. release-please reads the commits on `main`,
decides the next version, writes the changelog, and opens a release PR. That means
**your commit message is the version bump.** Get it wrong and the release is wrong
— or doesn't happen at all.

## The one thing to remember

Because merges are **squash** with the commit message set to *Pull request title
and description*, the commit that lands on `main` is:

```
<PR title>  (#42)

<PR description>
```

So **the PR title is the commit message that matters.** Individual commits on your
branch are collapsed and discarded — no need to make each one conventional. Get
the PR title right.

## Quick reference

| PR title starts with | Version effect | `1.4.2` becomes |
|---|---|---|
| `fix:` | patch | `1.4.3` |
| `feat:` | minor | `1.5.0` |
| any type + `!`, or a `BREAKING CHANGE:` footer | major | `2.0.0` |
| `chore:` `docs:` `refactor:` `test:` `ci:` `build:` `style:` `perf:` | none | `1.4.2` |

Only `feat`, `fix`, and breaking changes move the version. The other types are
recorded but don't create a release on their own — a branch of pure `chore:` work
produces no release PR.

## Format

```
<type>(<optional scope>): <description>
```

- **type** — from the table above. Lowercase.
- **scope** — optional, the area touched: `feat(payments):`. Shows in the changelog.
- **description** — imperative mood, lowercase start, no trailing period.
  "add retry logic", not "Added retry logic." or "adds retry logic".

## Examples

**Good**

```
fix: handle null customer id in order lookup
feat: add retry logic to payment client
feat(orders): support partial refunds
chore(deps): bump Serilog to 4.2.0
docs: document the retry backoff settings
refactor: extract OrderValidator from OrderService
perf: cache currency lookups per request
```

**Bad, and why**

| Title | Problem |
|---|---|
| `Fixed the null bug` | No type — release-please ignores it, no version bump |
| `feat : add refunds` | Space before the colon; not parsed |
| `Feat: add refunds` | Type must be lowercase |
| `feat: Added support for partial refunds.` | Past tense and trailing period |
| `update stuff` | No type, and says nothing for the changelog |
| `feat: fix null id` | Type contradicts the change — inflates the version |

The description ends up verbatim in the public changelog. Write it for someone
reading release notes, not for yourself.

## Breaking changes

Two ways, both valid. Either bumps **major**.

**1. `!` before the colon** — for when the title says it all:

```
feat!: drop support for API v1
```

**2. A `BREAKING CHANGE:` footer** — preferred when it needs explanation. This goes
in the **PR description**, because the description becomes the commit body:

```
feat: require an explicit retry policy

PaymentClient no longer constructs a default policy.

BREAKING CHANGE: PaymentClient constructor now requires an IRetryPolicy.
Callers must pass one explicitly.
```

The footer must be `BREAKING CHANGE:` — uppercase, space not hyphen, at the start
of a line. `Breaking change:` or `BREAKING-CHANGE` in the middle of a paragraph
will not be detected.

> For a **shared library**, a missed major bump is the worst mistake in this
> document: every consuming service picks up a breaking change as if it were a
> patch. When in doubt on a contracts repo, mark it breaking.

## Worked example, end to end

Jira sub-task `PROJ-124`. Branch:

```
feature/PROJ-124-retry-payment-client
```

PR title:

```
feat: add retry logic to payment client
```

PR description:

```
Retries transient 5xx responses up to 3 times with exponential backoff.

BREAKING CHANGE: PaymentClient constructor now requires an IRetryPolicy.
```

Squash-merged, this lands on `main` as one commit:

```
feat: add retry logic to payment client (#42)

Retries transient 5xx responses up to 3 times with exponential backoff.

BREAKING CHANGE: PaymentClient constructor now requires an IRetryPolicy.
```

release-please sees the breaking footer and opens a release PR bumping
`1.4.2 → 2.0.0`, with a changelog entry like:

```markdown
## 2.0.0 (2026-08-14)

### ⚠ BREAKING CHANGES

* PaymentClient constructor now requires an IRetryPolicy.

### Features

* add retry logic to payment client ([#42](...))
```

## Where the Jira key goes

In the **branch name**, not the commit title. `feature/PROJ-124-…` is what links
the work to Jira; the commit title stays a clean conventional message. Don't write
`feat: PROJ-124 add retry logic` — the ticket number would end up in the public
changelog.

## What happens after you merge

1. Your commit lands on `main` → DEV deploys automatically, QA waits for approval.
2. release-please opens a **release PR** — or updates the existing one if it's
   already open. Nothing is released yet.
3. That PR accumulates every releasable commit until someone merges it.
4. Merging the release PR cuts the tag, the GitHub Release, and the changelog —
   and that is what starts the PreProd and Prod pipeline.

So merging your PR does **not** ship to production. Cutting the release does.

## Fixing mistakes

- **Wrong type, not yet merged** — edit the PR title. The squash commit is built at
  merge time, so the fix takes effect.
- **Wrong type, already merged** — don't rewrite `main`. Land a follow-up commit
  with the right type. If a breaking change shipped as a `feat`, the next release
  can be a deliberate major.
- **Released a bad version** — roll forward. Releases are immutable and published
  package versions can't be replaced, so ship the next patch rather than trying to
  fix `1.4.3` in place.
