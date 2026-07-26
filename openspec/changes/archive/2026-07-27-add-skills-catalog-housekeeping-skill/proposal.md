## Why

This repo publishes multiple Agent Skills (`skill-standard`, `agentify-project`) through skills.sh, configured via `skills.sh.json`. Nothing currently keeps that catalog config or `README.md` in sync with the actual set of skill directories: `agentify-project` is already committed, pushed, and listed in `skills.sh.json`'s groupings, yet `README.md`'s "Skills" table still only documents `skill-standard`. There's no mechanism that catches this kind of drift after a skill is added, renamed, or removed, and no single place documenting how skills.sh actually discovers a repo's skills (there is no publish/register CLI step — it's pure GitHub-repo crawling plus install telemetry) — without that context, someone might invent a fake "registration" step later.

## What Changes

- Add a new, repo-scoped skill that runs as post-authoring housekeeping after a skill is added, renamed, or removed in this repo: it checks `skills.sh.json` against the actual top-level skill directories (every directory with a `SKILL.md` appears in exactly one `groupings[].skills` entry or is deliberately left ungrouped; no grouping references a directory that no longer exists; `$schema` preferred over the legacy `schema` field; `notGrouped` is a valid enum value; grouping/skill-count limits from the schema are respected), and checks `README.md`'s "Skills" table, "Repository layout" tree, and any per-skill description prose against the same actual skill list.
- The new skill's own `SKILL.md` SHALL document, as grounding content (not a task to perform), that skills.sh has no publish/register/submit CLI command — discovery is pure GitHub-repo crawling of `skills.sh.json` + `SKILL.md` files, plus CLI install telemetry for leaderboard ranking — so this doesn't get reinvented as a fake step later.
- The new skill SHALL be made available for **both** Claude Code and Cursor, following this repo's own established dual-harness skill-availability pattern (canonical store mirrored per harness, tracked via a lockfile) rather than being usable from only one harness.
- Fix the concrete drift that motivated this proposal as part of this change's own tasks: add `agentify-project` to `README.md`'s "Skills" table (and any other stale spots the audit finds), so the repo starts from a caught-up state rather than leaving the gap for the new skill's first future run to discover.

## Capabilities

### New Capabilities
- `skills-catalog-housekeeping`: repo-scoped skill that audits and fixes drift between this repo's actual skill directories, `skills.sh.json`, and `README.md`, and documents skills.sh's real (registration-free) discovery mechanism.

### Modified Capabilities
<!-- None: this does not change requirements of skill-standard or agentify-project-skill. -->

## Impact

- Affected paths: a new skill directory (name TBD, e.g. `skills-catalog-housekeeping/`) plus its `SKILL.md` and any references; `README.md` (fixing the current `agentify-project` gap); this repo's dual-harness skill-availability scaffolding (e.g. `.agents/skills/`, `.claude/skills/`, `.cursor/skills/`, a lockfile) to the extent needed to make the new skill available on both harnesses.
- No `skills.sh.json` change is expected unless the design decides this new skill should itself be a published, catalog-listed skill rather than purely local repo tooling — see design.md for that decision.
- Depends on: the skills.sh schema and CLI findings already gathered this session (`skills.sh.json` schema fields/limits; confirmed absence of any producer-side publish/register command) — read at design time, not re-derived.
