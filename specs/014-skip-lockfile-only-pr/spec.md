# Feature Specification: Skip Lockfile-Only Automated PRs

**Feature Branch**: `014-skip-lockfile-only-pr`
**Created**: 2026-04-09
**Status**: Done (retroactive spec)
**Input**: Review comment by @marcusburghardt on complytime/website PR #8

## Overview

The weekly `sync-content-check.yml` workflow scans upstream repositories for new commits and updates `.content-lock.json` with the latest branch SHAs. It then unconditionally opens an automated pull request — even when every upstream doc file is byte-for-byte identical to what the site would already render at the new SHA. This produces review noise (PR #8 example: 5 repos added to the lock, 32 files unchanged, PR opened) and wastes reviewer attention on approving a lockfile bump that results in zero site content changes.

This feature adds a two-step check so the automated PR is only created when at least one doc file would actually change as a result of accepting the new lock SHAs.

## User Scenarios & Testing

### User Story 1 — No PR When Docs Are Unchanged (Priority: P1)

A maintainer monitoring the repository's PR list should not see automated content-sync PRs that carry zero doc content changes. Only the lockfile SHA would differ, and the rendered site would look identical before and after merge.

**Why this priority**: Review fatigue from spurious PRs erodes trust in the automation. If reviewers learn to rubber-stamp lockfile-only PRs, they lose their function as a genuine content approval gate.

**Independent Test**: Run the workflow against a state where all upstream repo SHAs have changed but no doc files differ (byte-level). Confirm the "Create or update PR" step is skipped.

**Acceptance Scenarios**:

1. **Given** the weekly check runs and all upstream repos have new commits but zero doc files changed, **When** the `has_changes` output from the content-sync step is `false`, **Then** the "Create or update PR" step is skipped and no PR is opened or updated.
2. **Given** the weekly check runs and at least one doc file has changed content, **When** the `has_changes` output is `true`, **Then** the PR is created or updated as before.
3. **Given** a new repo is added to the lockfile for the first time and its docs are already in sync with the on-disk state, **When** the content-sync step runs with `--write`, **Then** `has_changes=false` is emitted and no PR is opened.

---

### User Story 2 — Accurate `has_changes` Detection Includes Doc-File-Level Changes (Priority: P1)

The `has_changes` GitHub Actions output must reflect actual file writes, not only repo-level project-page (README) changes. A repo whose README SHA is unchanged but whose doc sub-pages have new content must still set `has_changes=true`.

**Why this priority**: Without this fix the gate added in US1 has a gap — doc page changes would slip through undetected and the PR would still be skipped even when content did change.

**Independent Test**: Run `go test ./cmd/sync-content/... -run TestHasChanges` and confirm the case `changedRepoFiles populated, added/updated/removed empty` returns `true`.

**Acceptance Scenarios**:

1. **Given** `syncResult` has a non-empty `changedRepoFiles` map but empty `added`, `updated`, and `removed` slices, **When** `hasChanges()` is called, **Then** it returns `true`.
2. **Given** `syncResult` has an empty `changedRepoFiles` map (or all slices within it are empty) and empty `added`/`updated`/`removed`, **When** `hasChanges()` is called, **Then** it returns `false`.

---

### Edge Cases

| Case | Expected Behaviour |
|------|--------------------|
| Lock SHA unchanged for all repos | `git diff .content-lock.json` is empty; `writeFileSafe` writes nothing; `has_changes=false`; no PR |
| New repo added to lock, docs already in sync | Repo processed with `--write`; `writeFileSafe` skips all writes; `has_changes=false`; no PR |
| Upstream SHA bumped, README unchanged, doc pages changed | `changedRepoFiles` populated by `syncRepoDocPages`; `has_changes=true`; PR opened |
| Workflow step 1 (`--update-lock`) fails | Step 2 does not run; PR step does not run; workflow exits non-zero |
| `GITHUB_OUTPUT` not set (local run) | `writeGitHubOutputs` is a no-op; behaviour unchanged for local `--write` runs |

## Requirements

### Functional Requirements

- **FR-001**: The `sync-content-check.yml` workflow MUST update `.content-lock.json` with current upstream SHAs in a dedicated step before running the content check.
- **FR-002**: The workflow MUST run a `--write` content sync using the updated lockfile and capture its `has_changes` GitHub Actions output.
- **FR-003**: The "Create or update PR" step MUST be guarded by `if: steps.check.outputs.has_changes == 'true'` so it is skipped when no doc files changed.
- **FR-004**: `syncResult.hasChanges()` MUST return `true` when `changedRepoFiles` contains at least one non-empty file slice, even if `added`, `updated`, and `removed` are all empty.
- **FR-005**: The PR body summary (`sync-summary.md`) MUST be generated during the `--write` step (step 2), not the `--update-lock` step (step 1), so it reflects actual file changes.

### Key Entities

- **`syncResult.changedRepoFiles`**: map of repo name → list of file paths actually written to disk during `--write`. Populated by `syncRepoDocPages` and `syncConfigSource` when `writeFileSafe` returns `written=true`.
- **`syncResult.hasChanges()`**: boolean predicate controlling the `has_changes` GitHub Actions output. Previously only checked `added`/`updated`/`removed`; now also checks `changedRepoFiles`.
- **`GITHUB_OUTPUT`**: environment file written by `writeGitHubOutputs`; contains `has_changes`, `changed_count`, `files_processed`, `error_count`.

## Success Criteria

### Measurable Outcomes

- **SC-001**: The "Create or update PR" step is skipped when all 32 doc files are unchanged (reproducing the PR #8 scenario).
- **SC-002**: `go test ./cmd/sync-content/... -run TestHasChanges` passes all 7 subtests including `doc_file_changed_without_repo-level_change`.
- **SC-003**: `go test -race ./cmd/sync-content/...` passes with zero data race warnings after the `hasChanges()` change.
- **SC-004**: The PR body accurately describes which files changed, generated from the `--write` step summary.

## Assumptions

- The content files written by `--write` land in `content/docs/projects/*/` which is gitignored; they are not committed to the PR branch. The PR only ever commits `.content-lock.json`.
- The deploy workflow (`deploy-gh-pages.yml`) continues to run `--write` at build time from the merged lockfile, so no content is lost by skipping the PR.
- Running the sync tool twice per workflow execution (once `--update-lock`, once `--write`) is acceptable given the weekly cron cadence and ~3-minute timeout.
