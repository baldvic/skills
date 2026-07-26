---
name: skills-catalog-housekeeping
description: Keeps this repo's skills.sh.json catalog config and README.md in sync with its actual top-level skill directories after a skill is added, renamed, or removed. Fixes additive gaps (a new skill missing from the catalog or README) directly; flags stale references and description-quality issues for review rather than auto-fixing them. Use when you've just added, renamed, or removed a skill in this repo, or want to check that the skills catalog and README are current.
---

# Skills Catalog Housekeeping

This repo publishes multiple Agent Skills through skills.sh, configured via `skills.sh.json`, and documents them in `README.md`'s "Skills" table and "Repository layout" tree. Nothing keeps those in sync with the actual skill directories automatically — this skill is that check, run on demand.

This skill exists in two identical copies — one under `.claude/skills/skills-catalog-housekeeping/` and one under `.cursor/skills/skills-catalog-housekeeping/` — so it's usable from either harness. **Every run starts by diffing these two copies against each other** (see step 0 below); if they've drifted, report that before doing anything else.

## Important: there is no "publish" or "register" step

skills.sh has no CLI command to publish, register, or submit a skill. The `skills` CLI's only commands are consumer-side (`add`, `use`, `remove`, `list`, `find`, `update`) and local-authoring (`init`, `experimental_sync`, `experimental_install`) — none of them push anything to skills.sh. Discovery is purely skills.sh crawling public GitHub repos that contain `skills.sh.json` + `SKILL.md` files; its leaderboard ranking comes from anonymous `skills` CLI install telemetry. **The only thing that makes a skill "registered" is committing and pushing an accurate `skills.sh.json` and `README.md`.** Do not invent a registration step beyond that — this is exactly the kind of thing that gets reinvented if it isn't written down.

## Process

0. **Self-consistency check.** Compare this skill's `.claude/skills/` copy against its `.cursor/skills/` copy (this file and [references/skills-sh-json-schema.md](references/skills-sh-json-schema.md)). If they differ, report the divergence before continuing — don't silently treat one copy as authoritative.
1. **Build the map.** Read-only: list every top-level directory containing a `SKILL.md`; read `skills.sh.json`'s current `groupings`; read `README.md`'s current "Skills" table and "Repository layout" tree.
2. **Check `skills.sh.json` against the actual skill directories.**
   - **Additive gap** (a skill directory not named in any `groupings[].skills` entry, and not deliberately left for `notGrouped` placement) — fix directly: add it to whichever existing grouping fits, or ask if the right grouping isn't obvious.
   - **Stale reference** (a `groupings[].skills` entry naming a directory that no longer exists) — report it, don't remove it automatically; it might be an in-progress rename rather than a real removal.
3. **Check `skills.sh.json` against its schema.** See [references/skills-sh-json-schema.md](references/skills-sh-json-schema.md) for the confirmed shape and limits. Mechanical violations (e.g. the legacy `schema` field instead of `$schema`, an invalid `notGrouped` value) get fixed or flagged per that reference.
4. **Check `README.md` against the actual skill directories.**
   - **Missing row/tree entry** — add it directly. Ground the "Skills" table's one-line description in that skill's own `SKILL.md` frontmatter `description`, condensed to one line in the same style as the existing rows.
   - **Stale row/tree entry** (names a skill directory that no longer exists) — report it, don't remove it automatically.
   - **Description-quality judgment** (an existing row's description reads as stale or inaccurate, but the directory itself still exists — not a mechanical gap) — report as a recommendation; never rewrite someone's descriptive prose automatically.
5. **Report.** Summarize what was fixed directly (additive gaps) versus what's flagged for a decision (stale references, description-quality issues, self-consistency divergence, schema violations).

## Non-negotiable rules

- Never silently delete a `skills.sh.json` grouping entry or a `README.md` row/tree line just because the directory it names doesn't currently exist — report it instead.
- Never rewrite an existing, accurate skill description just to reword it — only fix demonstrable gaps (missing entirely) or report demonstrable quality issues.
- Never invent a skills.sh publish/registration step — see "Important" above.
- Never wire this skill to a git hook, CI check, or other automated trigger — it runs only when explicitly asked, in either harness.
- Never modify anything outside `skills.sh.json`, `README.md`, and this skill's own two copies (for the self-consistency check) — this is not a general-purpose repo-editing skill.

## Edge cases

- **A skill directory has a `SKILL.md` but isn't yet in `skills.sh.json` at all** — treat as an additive gap for both `skills.sh.json` and `README.md`; fix both together in one pass rather than requiring two separate invocations.
- **Both copies of this skill are already known to match** — the self-consistency check still runs every time (it's cheap); skip straight to step 1 once confirmed.
- **`README.md` has no "Skills" table yet at all** — this shouldn't happen for this repo currently, but if it ever does, report it rather than fabricating an entire new README structure — that's beyond this skill's narrow scope.
