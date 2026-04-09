## MODIFIED Requirements

### Requirement: has-changes-detection

`syncResult.hasChanges()` SHALL return `true` when ANY of the following are true:

- `result.added` is non-empty (new repos discovered)
- `result.updated` is non-empty (repo-level project page changed)
- `result.removed` is non-empty (repos removed)
- `result.changedRepoFiles` contains at least one repo with a non-empty file slice (doc pages written to disk)

Previously, `hasChanges()` only checked `added`, `updated`, and `removed`. Doc-page-level changes tracked in `changedRepoFiles` were not reflected in the `has_changes` GitHub Actions output.

#### Scenario: doc file changed without repo-level change

WHEN a repo's README SHA is unchanged but one or more of its doc sub-pages have new content
THEN `changedRepoFiles[repoName]` SHALL be non-empty
AND `hasChanges()` SHALL return `true`
AND the `has_changes` GitHub Actions output SHALL be `"true"`

#### Scenario: changedRepoFiles present but empty

WHEN `changedRepoFiles` contains a repo key but its file slice is empty
AND `added`, `updated`, `removed` are all empty
THEN `hasChanges()` SHALL return `false`

#### Scenario: all unchanged

WHEN no repos are added, updated, or removed and no doc files were written
THEN `hasChanges()` SHALL return `false`
AND the `has_changes` GitHub Actions output SHALL be `"false"`
