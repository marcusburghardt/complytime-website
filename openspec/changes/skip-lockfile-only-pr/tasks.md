## 1. Fix `hasChanges()` — content-change-detection

- [x] 1.1 Extend `syncResult.hasChanges()` in `cmd/sync-content/sync.go` to iterate `changedRepoFiles` and return `true` when any repo has at least one written file
- [x] 1.2 Add `TestHasChanges` table-driven test in `cmd/sync-content/sync_test.go` with 7 subtests: empty result, unchanged-only, added, updated, removed, `changedRepoFiles` non-empty with no repo-level changes (the critical new case), and `changedRepoFiles` present with empty slice
- [x] 1.3 Verify `go test -race ./cmd/sync-content/...` passes

## 2. Split workflow step — pr-noise-reduction

- [x] 2.1 Rename the existing "Check for upstream changes" step to "Update content lock" and remove `--summary sync-summary.md` from its command (lock update only)
- [x] 2.2 Add a new "Check for doc content changes" step with `id: check` running `--lock .content-lock.json --write --summary sync-summary.md`
- [x] 2.3 Add `if: steps.check.outputs.has_changes == 'true'` condition to the "Create or update PR" step
