## Context

The `sync-content-check.yml` workflow runs weekly on Monday 06:00 UTC. It scans upstream repositories for new commits, updates `.content-lock.json` with the latest branch SHAs, and creates an automated PR for human review. Content files land in `content/docs/projects/*/` which is gitignored — they are synced at deploy time from the merged lockfile, not committed to the PR branch. The PR serves exclusively as a human approval gate for the lockfile.

The problem: the workflow opened PRs even when no doc files would change at the new SHAs, as demonstrated by PR #8 (5 repos, 32 files, all unchanged).

## Goals / Non-Goals

**Goals**:
- Skip PR creation when no doc files would change.
- Ensure `has_changes` reflects file-level writes, not just repo-level SHA drift.
- Keep the PR-only-for-lockfile model intact (content still synced at deploy time).

**Non-Goals**:
- Committing content files into the PR branch.
- Changing the deploy workflow or lockfile approval semantics.
- Achieving file-level comparison without actually fetching file content.

## Decisions

**Decision: run `--write` in the check workflow to detect file changes**

The sync tool's `writeFileSafe` performs a byte-for-byte comparison before writing. Running with `--write` is the only way to get accurate file-level change detection without a new flag. Since `content/docs/projects/*/` is gitignored, the written files do not affect the PR branch — only `.content-lock.json` is staged via `add-paths`.

Alternatives considered:
- `--dry-run` with a new `--check-files` flag: would require fetching and comparing file content without writing; same API cost, more code. Deferred.
- Checking `git diff content/`: fragile — content may already be stale from a prior run.

**Decision: two sequential steps rather than one combined step**

Separating `--update-lock` from `--write` makes each step's purpose explicit and allows the workflow to fail fast if the lock update fails before attempting the more expensive content fetch. It also makes the `id: check` output reference unambiguous.

**Decision: extend `hasChanges()` rather than add a new output**

`changedRepoFiles` was already tracked but not included in `hasChanges()`. Adding it there fixes the gap consistently across all callers (`printSummary`, `writeGitHubOutputs`, `toMarkdown`) without introducing a parallel code path.

## Risks / Trade-offs

- **Two API passes per weekly run** → The sync tool runs twice (once for lock, once for content). API cost doubles for the check workflow. Acceptable given the weekly cadence and 3-minute timeout.
- **Written files linger in working tree** → The gitignored `content/` files written by `--write` remain on the runner's disk. They are not committed and are discarded when the runner terminates.
- **`--write` side effects if runner is reused** → Not applicable for GitHub-hosted runners (ephemeral). Would need care on self-hosted runners.
