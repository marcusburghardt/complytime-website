## Why

The weekly `sync-content-check.yml` workflow unconditionally opens an automated PR whenever upstream repository SHAs change — even when every doc file is byte-for-byte identical to what is already on disk. PR #8 demonstrated this: 5 repos newly added to the lockfile, 32 files unchanged, yet a PR was opened for review. This generates noise, dilutes the content approval gate, and trains reviewers to rubber-stamp lockfile-only PRs.

## What Changes

- Split the single "Check for upstream changes" step into two steps: one that updates the lockfile (`--update-lock`) and one that runs a `--write` content sync to detect actual file changes.
- Guard the "Create or update PR" step with `if: steps.check.outputs.has_changes == 'true'` so the PR is only opened when doc content would actually change.
- Extend `syncResult.hasChanges()` in the Go sync tool to also return `true` when `changedRepoFiles` is non-empty — previously only repo-level project-page changes (`added`/`updated`/`removed`) triggered `has_changes=true`, missing doc-page-level changes tracked in `changedRepoFiles`.

## Capabilities

### New Capabilities

- `pr-noise-reduction`: Gate automated content-sync PR creation on actual doc file changes, not just lockfile SHA drift.

### Modified Capabilities

- `content-change-detection`: `syncResult.hasChanges()` now includes file-level changes (`changedRepoFiles`) in addition to repo-level changes.

## Impact

- **`.github/workflows/sync-content-check.yml`**: Step split, `id: check` added, `if:` guard on PR step.
- **`cmd/sync-content/sync.go`**: `hasChanges()` iterates `changedRepoFiles`.
- **`cmd/sync-content/sync_test.go`**: 7-subtest `TestHasChanges` table added.
- No new dependencies. No breaking changes. Deploy workflow (`deploy-gh-pages.yml`) is unaffected.
