# sample-service  (per-repo files)

A service repo. Holds only what's **repo-specific**; policy/quality pieces are
called from `YOUR_ORG/.github`.

## What's here

| Path | Type | Purpose |
|---|---|---|
| `.github/workflows/pr.yml` | Caller | Local `build` + `unit-tests` (via the org composite action) + calls the centralized `branch-name` and `ai-review` reusable workflows |
| `.github/workflows/deploy.yml` | Local | Auto DEV/QA on merge to `main`; gated PreProd/Prod |
| `.github/CODEOWNERS` | Local | Repo-specific reviewer routing; overrides the org default |
| `.github/review-guidelines.md` | Local | Rubric the `ai-review` gate applies here (not `CLAUDE.md`, which Claude Code claims for its own context) |

## Required checks (mark these in the `protect-main` ruleset)

`build` · `unit-tests` · `ai-review/review` · `branch-name/validate`

Select them from the "Add checks" picker (they appear only after running once).

## Before you push

1. Push `YOUR_ORG/.github` first — `pr.yml` references its workflows/action.
2. Replace `YOUR_ORG` in `pr.yml` (3 refs).
3. Set `ANTHROPIC_API_KEY` as a repo/org secret (forwarded via `secrets: inherit`).
4. Set `AZURE_CREDENTIALS`; create `dev`/`qa`/`preprod`/`prod` environments with
   required reviewers on `preprod`/`prod`.
5. Search files for `🔧 CUSTOMIZE FOR REAL FIRM`.
