## ADDED Requirements

### Requirement: conditional-pr-creation

The automated content-sync workflow SHALL only open or update a pull request when at least one upstream doc file has actually changed content. A lockfile SHA update that results in zero file writes MUST NOT trigger a PR.

#### Scenario: docs unchanged — PR skipped

WHEN the weekly sync runs and all upstream doc files are byte-for-byte identical to their current on-disk state
THEN the "Create or update PR" step SHALL be skipped and no PR SHALL be opened or updated

#### Scenario: docs changed — PR created

WHEN the weekly sync runs and at least one upstream doc file has changed content
THEN the "Create or update PR" step SHALL execute and a PR SHALL be created or updated with the new lockfile SHA

#### Scenario: new repo, docs already in sync — PR skipped

WHEN a repository is added to the lockfile for the first time but its doc files match what would be fetched at the new SHA
THEN `has_changes` SHALL be `false` and no PR SHALL be opened

### Requirement: two-step-workflow

The `sync-content-check.yml` workflow SHALL run the sync tool in two sequential steps:

1. **Update lock step**: runs `--update-lock` to write the new lockfile to disk.
2. **Check content step** (id: `check`): runs `--write` with the updated lockfile and captures the `has_changes` output.

The PR creation step SHALL reference the `check` step output via `steps.check.outputs.has_changes`.

#### Scenario: step ordering

WHEN the workflow executes
THEN the lock update step SHALL complete before the content check step runs
AND the content check step SHALL use the lockfile written by the lock update step
