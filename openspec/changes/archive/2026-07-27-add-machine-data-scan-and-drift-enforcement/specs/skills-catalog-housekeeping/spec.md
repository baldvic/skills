## ADDED Requirements

### Requirement: Machine-specific data scan
The skill SHALL perform a read-only sweep of the entire repository (not limited to `skills.sh.json` or `README.md`) for signs of leaked machine-specific or private data: absolute local paths (drive-letter paths, `/home/<user>/...`, `/Users/<user>/...`), the current OS username/real name/personal or company email address, and names of private/internal repos, systems, or hostnames used as illustrative examples. For each finding, the skill SHALL check whether the commit that introduced it is already an ancestor of the tracked remote's default branch. A confirmed leak with an obvious generic replacement SHALL be fixed directly in the working tree after confirming with the user first, since this is the one case where a fix extends beyond `skills.sh.json`/`README.md`. The skill SHALL NOT rewrite git history to scrub an already-pushed leak without the user explicitly requesting it.

#### Scenario: Leak not yet pushed
- **WHEN** the scan finds a machine-specific data leak whose introducing commit is not an ancestor of the remote's default branch
- **THEN** the skill reports the file and line, and offers to redact it directly in the working tree after user confirmation

#### Scenario: Leak already on the pushed default branch
- **WHEN** the scan finds a machine-specific data leak whose introducing commit is already an ancestor of the remote's default branch
- **THEN** the skill flags this distinctly from not-yet-pushed hits, states that redacting the working tree alone does not remove it from public git history, and does not attempt any git-history rewrite unless the user explicitly asks for it

#### Scenario: Generic example, not a real leak
- **WHEN** the scan encounters a placeholder path or well-known public project name used as a generic illustrative example
- **THEN** the skill does not flag it as a leak

### Requirement: Drift enforcement classification
Every finding this skill produces, across every check it performs, SHALL be classified into exactly one of two buckets: **mechanical** (deterministic, requires no guess about anyone's intent — fixed directly and reported as done) or **judgment-based** (could revert someone's in-progress work, or requires a subjective call — always reported for a decision, never auto-applied). When a finding does not clearly fit one bucket, the skill SHALL default to judgment-based.

#### Scenario: Ambiguous finding defaults to judgment-based
- **WHEN** a finding does not clearly fit either the mechanical or judgment-based bucket
- **THEN** the skill reports it for a decision rather than fixing it automatically
