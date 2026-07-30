# sample-repo (microservice shape)

A service repo. Holds only what's **repo-specific**; policy/quality pieces are
called from `YOUR_ORG/.github`.

## What's here

| Path | Phase | Purpose |
|---|---|---|
| `.github/workflows/pr.yml` | 1 | Local `build` + `unit-tests` (via the org composite action) + calls the centralized `branch-name` and `ai-review` reusable workflows |
| `.github/workflows/deploy.yml` | 1 → 2 | Builds the image **once** as `sha-*`, deploys DEV automatically and QA gated. No PreProd/Prod here — deliberately |
| `.github/workflows/release-please.yml` | 2 | Maintains the release PR; merging it cuts the tag, Release, and CHANGELOG |
| `.github/workflows/release-deploy.yml` | 2 | On published release: re-tags the existing digest, then gated PreProd → Prod |
| `.github/CODEOWNERS` | 1 | Repo-specific reviewer routing; overrides the org default |
| `.github/review-guidelines.md` | 1 | Rubric the `ai-review` gate applies here (not `CLAUDE.md`, which Claude Code claims for its own context) |
| `docs/Versioning.md` | 2 | **Read this before opening a PR.** How commit/PR titles drive the version bump and changelog |

## Required checks (`protect-main` ruleset)

`build` · `unit-tests` · `ai-review/review` · `branch-name/validate`

Displayed with spaces (`ai-review / review`), stored without. Select from the
"Add checks" picker rather than typing.

## Two rules the pipeline depends on

- **Build once, promote by digest.** The image is built only in `deploy.yml`.
  Deploys reference `@sha256:…`, never a tag, and the release re-tags the existing
  digest instead of rebuilding — so PreProd and Prod run the exact bytes QA ran.
- **PreProd/Prod are release-triggered.** They are not reachable from a merge.
  Merging release-please's PR is the ship decision.

## release-please outputs

`release-please.yml` exposes these; the release-triggered workflows read them. If
you change the release-please config, keep them exposed.

| Output | Used for |
|---|---|
| `release_created` | whether to run publish/promotion steps at all |
| `tag_name` (`v1.2.3`) | the git ref that was tagged |
| `version` (`1.2.3`) | Docker tag, NuGet version, Android `versionName` |

## Before you push

1. Push `YOUR_ORG/.github` first — `pr.yml` references its workflows/action.
2. Replace `YOUR_ORG` in `pr.yml` (3 refs).
3. Org secret `ANTHROPIC_API_KEY` (forwarded via `secrets: inherit`).
4. Org variable `RELEASE_PLEASE_CLIENT_ID` (the App's Client ID, not the numeric
   App ID) + secret `RELEASE_PLEASE_APP_PRIVATE_KEY`. **Required**: with the default
   `GITHUB_TOKEN`, releases don't trigger `release-deploy.yml` and the release PR's
   checks stall behind an "Approve workflows to run" banner.
5. Repo secret `AZURE_CREDENTIALS`; create `dev`, `qa`, `preprod`, `prod`
   environments with required reviewers on `qa`, `preprod`, `prod`.
6. Search files for `🔧`.
