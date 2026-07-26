# skills-catalog-housekeeping

## Purpose

Keeps this repo's `skills.sh.json` catalog config and `README.md` in sync with its actual top-level skill directories after a skill is added, renamed, or removed. Fixes additive gaps (a new skill missing from the catalog or README) directly; flags stale references and description-quality issues for review rather than auto-fixing them. Available as an identical, on-demand skill in both Claude Code and Cursor.

## Requirements

### Requirement: Repo-scoped skill available on both Claude Code and Cursor
The skill SHALL exist as two identical copies — one under `.claude/skills/skills-catalog-housekeeping/` and one under `.cursor/skills/skills-catalog-housekeeping/` — so it is usable from either harness, without requiring a canonical `.agents/skills/` store or `skills-lock.json`. Every invocation SHALL include a self-consistency check comparing the two copies' content and reporting any divergence.

#### Scenario: Both harnesses can invoke it
- **WHEN** a maintainer working in Claude Code or in Cursor asks to check or update the skills catalog
- **THEN** the corresponding harness-local copy of the skill activates and performs the housekeeping pass

#### Scenario: Copies have drifted apart
- **WHEN** the skill runs and finds its `.claude/skills/` and `.cursor/skills/` copies are not identical
- **THEN** it reports the divergence to the maintainer as part of its output, rather than silently treating one copy as authoritative

### Requirement: skills.sh.json catalog sync
The skill SHALL detect gaps between top-level skill directories (each containing a `SKILL.md`) and `skills.sh.json`'s `groupings`: a directory not listed in any grouping's `skills` array, and not deliberately left for `notGrouped` placement, is an additive gap; a grouping entry naming a directory that no longer exists is a stale reference. Additive gaps SHALL be fixed directly by adding the skill to an appropriate grouping. Stale references SHALL be reported to the maintainer, not silently removed.

#### Scenario: New skill directory missing from skills.sh.json
- **WHEN** a top-level directory with a `SKILL.md` exists but no `groupings[].skills` entry names it
- **THEN** the skill adds it to an appropriate grouping directly, or asks the maintainer which grouping fits if that judgment isn't obvious

#### Scenario: Stale grouping reference
- **WHEN** a `groupings[].skills` entry names a directory that no longer exists in the repo
- **THEN** the skill reports this as a stale reference requiring confirmation, and does not remove it automatically

### Requirement: skills.sh.json schema conformance
The skill SHALL validate `skills.sh.json` against the schema shape grounded in the schema fetched from skills.sh (cited with source URL and verification date in the skill's own reference content): `groupings` has 1–50 entries, each with a `title` (1–120 characters) and `skills` (1–500 names), optional `description` (≤500 characters); `notGrouped`, if present, is `"top"` or `"bottom"`; `$schema` is used in preference to the legacy `schema` field. Mechanical violations SHALL be fixed directly.

#### Scenario: Legacy schema field in use
- **WHEN** `skills.sh.json` uses the legacy `schema` field instead of `$schema`
- **THEN** the skill updates it to `$schema` directly

#### Scenario: Invalid notGrouped value
- **WHEN** `notGrouped` is set to a value other than `"top"` or `"bottom"`
- **THEN** the skill reports this as a schema violation rather than silently guessing a fix

### Requirement: README currency
The skill SHALL check `README.md`'s "Skills" table, "Repository layout" tree, and any per-skill descriptive prose against the actual current set of top-level skill directories. Missing entries SHALL be added directly, grounded in each skill's own `SKILL.md` frontmatter `description`. Entries referencing a skill directory that no longer exists SHALL be reported, not silently removed. Judgment-based staleness (a description that no longer accurately reflects a skill's actual scope, in a way that isn't mechanically checkable) SHALL be reported as a recommendation, not automatically rewritten.

#### Scenario: New skill missing from README
- **WHEN** a top-level skill directory exists but `README.md`'s "Skills" table has no row for it
- **THEN** the skill adds a row grounded in that skill's own `SKILL.md` frontmatter description, and updates the "Repository layout" tree to include it

#### Scenario: README references a removed skill
- **WHEN** `README.md`'s "Skills" table or "Repository layout" tree references a skill directory that no longer exists
- **THEN** the skill reports this rather than silently removing the reference

#### Scenario: Description quality judgment
- **WHEN** a skill's README description row looks stale or inaccurate in a way that isn't mechanically checkable
- **THEN** the skill reports this as a recommendation and does not rewrite the prose itself

### Requirement: skills.sh discovery mechanism documented as grounding content
The skill's own `SKILL.md` or references SHALL document that skills.sh has no publish/register/submit CLI command, and that discovery happens purely via crawling public GitHub repos containing `skills.sh.json` and `SKILL.md` files, plus CLI install telemetry for leaderboard ranking, so this is not reinvented as a fabricated step by this skill or a future maintainer.

#### Scenario: Maintainer asks how to "register" a new skill
- **WHEN** a maintainer asks this skill, or reads its documentation, how to register a new skill on skills.sh
- **THEN** the skill's own content explains that no such step exists — only committing and pushing an accurate `skills.sh.json` and `README.md` matters

### Requirement: On-demand invocation only
The skill SHALL activate only when a maintainer explicitly requests a catalog or README check or update in conversation, in either harness. It SHALL NOT be wired to a git hook, CI check, or other automated trigger.

#### Scenario: Maintainer requests a check
- **WHEN** a maintainer says something like "I just added a new skill, check the catalog" in either harness
- **THEN** the skill activates and performs its housekeeping pass

#### Scenario: No automatic trigger
- **WHEN** a new skill directory is added to the repo without a maintainer invoking this skill
- **THEN** no automatic hook or CI step runs it on their behalf

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
