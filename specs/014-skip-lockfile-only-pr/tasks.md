# Tasks: Skip Lockfile-Only Automated PRs

**Input**: Design documents from `specs/014-skip-lockfile-only-pr/`
**Branch**: `014-skip-lockfile-only-pr`
**Status**: Done — feature delivered on this branch.
**Prerequisites**: spec.md ✅

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to ([US1], [US2])

## Path Conventions

Hugo site root. All paths relative to `website/`.

---

## Phase 1: Go Tool Fix (US2 — Blocking)

**Purpose**: Fix `hasChanges()` to include file-level changes before adding the workflow gate. The workflow gate in Phase 2 depends on an accurate `has_changes` output.

**⚠️ CRITICAL**: Phase 2 workflow changes depend on this being correct first.

- [x] T001 [US2] Extend `syncResult.hasChanges()` in `cmd/sync-content/sync.go`: iterate `changedRepoFiles` and return `true` if any repo has at least one changed file, so doc-page-level changes are included in the `has_changes` GitHub Actions output alongside the existing repo-level (`added`/`updated`/`removed`) checks.
- [x] T002 [P] [US2] Add `TestHasChanges` table-driven test in `cmd/sync-content/sync_test.go`: 7 subtests covering empty result, unchanged-only, added, updated, removed, `changedRepoFiles` populated with no repo-level changes (the critical new case), and `changedRepoFiles` present but with an empty slice.

**Checkpoint**: `go test -race ./cmd/sync-content/...` passes. `TestHasChanges/doc_file_changed_without_repo-level_change` passes.

---

## Phase 2: Workflow Gate (US1)

**Purpose**: Split the single workflow step into two and guard PR creation on actual doc content changes.

- [x] T003 [US1] Split "Check for upstream changes" step in `.github/workflows/sync-content-check.yml` into two steps:
  1. **"Update content lock"** — runs `--update-lock` only (no `--summary`), writes updated `.content-lock.json` to disk.
  2. **"Check for doc content changes"** (id: `check`) — runs `--write --summary sync-summary.md` with the updated lock, capturing the `has_changes` output.
- [x] T004 [P] [US1] Add `if: steps.check.outputs.has_changes == 'true'` condition to the "Create or update PR" step in `.github/workflows/sync-content-check.yml`.

**Checkpoint**: Workflow correctly skips PR creation when `has_changes=false`. PR body (`sync-summary.md`) reflects actual file changes from the `--write` step.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1**: No dependencies — start immediately.
- **Phase 2**: Depends on Phase 1 — the workflow gate is only meaningful once `has_changes` accurately reflects file-level changes.

### Parallel Opportunities

- T001 and T002 are in the same package but different locations within `sync.go` / `sync_test.go` — can be authored in parallel.
- T003 and T004 are both in `sync-content-check.yml` — sequential edits to the same file.

---

## Notes

- No new Go dependencies introduced.
- `content/docs/projects/*/` is gitignored; the `--write` run in Phase 2 writes to the working tree but the files are never staged. Only `.content-lock.json` reaches the PR branch via `add-paths: .content-lock.json`.
- The PR body now comes from the `--write` step summary (step 2) rather than the `--update-lock` step (step 1), which means it accurately describes which files would change rather than which repo SHAs changed.
