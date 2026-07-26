## 1. Author the skill's core content

- [x] 1.1 Draft `SKILL.md` frontmatter (`name: skills-catalog-housekeeping`, a description covering both what it does and when to invoke it — e.g. "just added/renamed/removed a skill," "check the skills catalog") per `skill-standard` conventions
- [x] 1.2 Write the process: build a read-only map of top-level skill directories (each with a `SKILL.md`), `skills.sh.json`'s current groupings, and `README.md`'s current "Skills" table/"Repository layout" tree, before making any change — per `design.md`'s Context
- [x] 1.3 Document the additive-fix vs. stale-reference-report vs. prose-judgment-report classification for both `skills.sh.json` and `README.md` drift — per `design.md` Decision 4
- [x] 1.4 Document the on-demand-only invocation model explicitly (activates on conversational request in either harness; no git hook, no CI check) — per `design.md` Decision 3
- [x] 1.5 Document, as grounding content, that skills.sh has no publish/register/submit CLI command and discovers skills purely via crawling `skills.sh.json` + `SKILL.md` in public repos plus install telemetry — per the spec's "skills.sh discovery mechanism documented as grounding content" requirement
- [x] 1.6 Document the self-consistency check between the `.claude/skills/` and `.cursor/skills/` copies as a step run on every invocation — per `design.md` Decision 1

## 2. Add reference material

- [x] 2.1 Create a reference file documenting the confirmed `skills.sh.json` schema shape (`groupings` 1–50 entries; each with `title` 1–120 chars and `skills` 1–500 names; optional `description` ≤500 chars; `notGrouped` enum `"top"`/`"bottom"`; `$schema` preferred over legacy `schema`), citing the source URL (`https://skills.sh/schemas/skills.sh.schema.json`) and today's verification date — per `design.md` Decision 5

## 3. Place the skill in both harnesses

- [x] 3.1 Create `.claude/skills/skills-catalog-housekeeping/SKILL.md` (plus the reference file from task group 2)
- [x] 3.2 Create an identical copy at `.cursor/skills/skills-catalog-housekeeping/SKILL.md` (plus the same reference file)
- [x] 3.3 Confirm the two copies are identical (the same check the skill itself will run on future invocations)

## 4. Fix the concrete drift found this session

- [x] 4.1 Add `agentify-project` to `README.md`'s "Skills" table with a one-line description grounded in `agentify-project/SKILL.md`'s own frontmatter `description`, alongside the existing `skill-standard` row
- [x] 4.2 Update `README.md`'s "Repository layout" tree to include `agentify-project/` alongside `skill-standard/`
- [x] 4.3 Confirm `skills.sh.json` already correctly lists both skills (expected to already be correct per this session's earlier finding — verify, no change expected)

## 5. Validate

- [x] 5.1 Apply `skill-standard/references/validation.md`'s checklist to the new skill (frontmatter fields, directory-name-matches-`name`, no disallowed top-level keys, no host-specific absolute paths, description length) for both the `.claude/skills/` and `.cursor/skills/` copies
- [x] 5.2 Decide whether a minimal `evals/evals.json` is worth adding for this skill (optional per `skill-standard`'s own conventions) — added 3 scenarios (additive gap, stale reference, self-consistency divergence) under `.claude/skills/skills-catalog-housekeeping/evals/` only, since dev-time eval fixtures aren't runtime content either harness needs, so duplicating them into the `.cursor/skills/` copy would be redundant maintenance with no benefit
