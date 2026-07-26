---
name: skills-catalog-housekeeping
description: Keeps this repo's skills.sh.json catalog config and README.md in sync with its actual top-level skill directories after a skill is added, renamed, or removed, and scans the whole repo for leaked machine-specific data (local absolute paths, usernames, private/internal repo or system names used as examples). Fixes additive gaps (a new skill missing from the catalog or README) and confirmed local-data leaks directly; flags stale references, description-quality issues, and anything already pushed to the remote default branch for review rather than auto-fixing. Use when you've just added, renamed, or removed a skill in this repo, want to check that the skills catalog and README are current, or want to verify nothing machine-specific leaked into a public repo.
---

# Skills Catalog Housekeeping

This repo publishes multiple Agent Skills through skills.sh, configured via `skills.sh.json`, and documents them in `README.md`'s "Skills" table and "Repository layout" tree. Nothing keeps those in sync with the actual skill directories automatically — this skill is that check, run on demand.

This skill exists in two identical copies — one under `.claude/skills/skills-catalog-housekeeping/` and one under `.cursor/skills/skills-catalog-housekeeping/` — so it's usable from either harness. **Every run starts by diffing these two copies against each other** (see step 0 below); if they've drifted, report that before doing anything else.

This repo is published publicly (see `README.md`), so it also gets scanned end-to-end for accidentally-leaked machine-specific data — see step 5 below. That scan is the one place this skill looks outside `skills.sh.json`/`README.md`; everything else it does stays scoped to those two files plus its own copies.

## Important: there is no "publish" or "register" step

skills.sh has no CLI command to publish, register, or submit a skill. The `skills` CLI's only commands are consumer-side (`add`, `use`, `remove`, `list`, `find`, `update`) and local-authoring (`init`, `experimental_sync`, `experimental_install`) — none of them push anything to skills.sh. Discovery is purely skills.sh crawling public GitHub repos that contain `skills.sh.json` + `SKILL.md` files; its leaderboard ranking comes from anonymous `skills` CLI install telemetry. **The only thing that makes a skill "registered" is committing and pushing an accurate `skills.sh.json` and `README.md`.** Do not invent a registration step beyond that — this is exactly the kind of thing that gets reinvented if it isn't written down.

## Drift enforcement

Every check below sorts what it finds into exactly two buckets — the same mechanical-vs-judgment split `agentify-project` uses for its own custom-skill validation (see that skill's `references/custom-skill-drift-enforcement.md`):

- **Mechanical** — deterministic, doesn't require guessing anyone's intent: a skill directory missing entirely from `skills.sh.json`/`README.md` (additive gap), the legacy `schema` field instead of `$schema`, a confirmed machine-specific-data leak with an obvious generic replacement. Fixed directly, then reported as done.
- **Judgment-based** — could be reverting someone's in-progress work, or requires a subjective call: a `groupings[].skills` entry or `README.md` row/tree line naming a directory that no longer exists (might be a rename mid-flight, not a real removal), a description that merely *reads* stale, a self-consistency divergence between this skill's two copies, a data leak whose remediation would mean rewriting git history. Always reported for a decision — never auto-applied.

When a finding doesn't obviously fit one bucket, default to judgment-based: report it, don't touch it. This split is the one invariant across every step in Process below; each step's bullets are labeled with which bucket they fall into.

## Process

0. **Self-consistency check.** Compare this skill's `.claude/skills/` copy against its `.cursor/skills/` copy (this file and [references/skills-sh-json-schema.md](references/skills-sh-json-schema.md)). If they differ, report the divergence before continuing — don't silently treat one copy as authoritative.
1. **Build the map.** Read-only: list every top-level directory containing a `SKILL.md`; read `skills.sh.json`'s current `groupings`; read `README.md`'s current "Skills" table and "Repository layout" tree.
2. **Check `skills.sh.json` against the actual skill directories.**
   - **Additive gap** *(mechanical)* — a skill directory not named in any `groupings[].skills` entry, and not deliberately left for `notGrouped` placement — fix directly: add it to whichever existing grouping fits, or ask if the right grouping isn't obvious.
   - **Stale reference** *(judgment-based)* — a `groupings[].skills` entry naming a directory that no longer exists — report it, don't remove it automatically; it might be an in-progress rename rather than a real removal.
3. **Check `skills.sh.json` against its schema.** See [references/skills-sh-json-schema.md](references/skills-sh-json-schema.md) for the confirmed shape and limits. Mechanical violations *(mechanical)* (e.g. the legacy `schema` field instead of `$schema`, an invalid `notGrouped` value) get fixed directly; anything ambiguous about the schema itself is flagged per that reference.
4. **Check `README.md` against the actual skill directories.**
   - **Missing row/tree entry** *(mechanical)* — add it directly. Ground the "Skills" table's one-line description in that skill's own `SKILL.md` frontmatter `description`, condensed to one line in the same style as the existing rows.
   - **Stale row/tree entry** *(judgment-based)* — names a skill directory that no longer exists — report it, don't remove it automatically.
   - **Description-quality judgment** *(judgment-based)* — an existing row's description reads as stale or inaccurate, but the directory itself still exists, so it's not a mechanical gap — report as a recommendation; never rewrite someone's descriptive prose automatically.
5. **Machine-specific data scan.** Read-only sweep of the *entire* repo (not just `skills.sh.json`/`README.md`) for signs this is someone's local machine or private environment leaking into a public repo:
   - Absolute local paths — drive-letter paths (`C:\Users\...`, `D:\projects\...`), `/home/<user>/...`, `/Users/<user>/...`, `AppData\...`, `.claude\` under a personal profile path.
   - The current OS username, real name, or personal/company email address (compare against the session's own `userEmail` context and OS user if known).
   - Names of private/internal repos, systems, or hostnames used as illustrative examples (e.g., a design doc citing how "our internal repo X" does something) — these leak information about systems outside this repo, not just paths.
   - For every hit: report the file and line. Then check whether the commit that introduced it is already an ancestor of the tracked remote's default branch (`git merge-base --is-ancestor <commit> origin/<branch>`) — if yes, flag this prominently and separately from not-yet-pushed hits *(judgment-based — never auto-applied)*: **redacting the working tree does not remove it from public git history**; rewriting history to scrub an already-pushed commit is a destructive operation on a shared/public repo and must never be done without the user explicitly asking for it first.
   - A confirmed leak with an obvious generic replacement *(mechanical)* — e.g. swapping a private repo name for a generic descriptive term, or an absolute path for a relative one or generic example — is safe to fix directly in the working tree, the same as other mechanical fixes this skill makes. This is the one case where a mechanical fix touches files outside `skills.sh.json`/`README.md`; confirm with the user first since it's outside this skill's normal write scope (see Non-negotiable rules).
6. **Report.** Summarize what was fixed directly (additive gaps, confirmed local-data redactions) versus what's flagged for a decision (stale references, description-quality issues, self-consistency divergence, schema violations, any leak already on the pushed default branch).

## Non-negotiable rules

- Never silently delete a `skills.sh.json` grouping entry or a `README.md` row/tree line just because the directory it names doesn't currently exist — report it instead.
- Never rewrite an existing, accurate skill description just to reword it — only fix demonstrable gaps (missing entirely) or report demonstrable quality issues.
- Never invent a skills.sh publish/registration step — see "Important" above.
- Never wire this skill to a git hook, CI check, or other automated trigger — it runs only when explicitly asked, in either harness.
- Never modify anything outside `skills.sh.json`, `README.md`, and this skill's own two copies (for the self-consistency check), with one narrow exception: redacting a confirmed machine-specific-data leak found anywhere in the repo during step 5, and only after confirming with the user first since that touches files outside the normal scope — this is not a general-purpose repo-editing skill.
- Never rewrite git history (rebase, filter-branch, force-push) to scrub an already-pushed leak without the user explicitly asking for it — flag it and let them decide.

## Edge cases

- **A skill directory has a `SKILL.md` but isn't yet in `skills.sh.json` at all** — treat as an additive gap for both `skills.sh.json` and `README.md`; fix both together in one pass rather than requiring two separate invocations.
- **Both copies of this skill are already known to match** — the self-consistency check still runs every time (it's cheap); skip straight to step 1 once confirmed.
- **`README.md` has no "Skills" table yet at all** — this shouldn't happen for this repo currently, but if it ever does, report it rather than fabricating an entire new README structure — that's beyond this skill's narrow scope.
- **A machine-specific leak is found in a commit that's already on the remote's default branch** — the data is already public; report this distinctly from not-yet-pushed hits, and don't attempt any git-history rewrite unless the user explicitly asks for it.
- **A "leak" turns out to be a generic example, not real local data** (e.g., a placeholder path like `/path/to/repo`, or a well-known public project name) — don't flag it; the scan is for genuine machine/environment fingerprints, not any string that looks like a path.
