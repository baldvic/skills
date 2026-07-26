## Why

A live audit of this session found real leaks: an internal/private repo's name and local absolute paths had been committed and pushed to this public repo across several `openspec/` design docs. `skills-catalog-housekeeping` had no check for this class of problem — it only reconciled `skills.sh.json`/`README.md` against actual skill directories. Separately, the skill's existing additive-fix-vs-report split (used independently for `skills.sh.json` gaps and `README.md` gaps) was never named as a single reusable concept, even though `agentify-project` already has one (its mechanical-vs-judgment split for custom-skill validation) that this skill's behavior already mirrors in spirit.

## What Changes

- Add a **machine-specific data scan** to the skill's process: a read-only, whole-repo sweep (not limited to `skills.sh.json`/`README.md`) for absolute local paths, usernames/emails, and private/internal repo or system names used as illustrative examples. A confirmed leak with an obvious generic replacement is fixed directly (the one case where this skill's writes extend beyond `skills.sh.json`/`README.md`, and only after confirming with the user first). A leak already reachable from the tracked remote's default branch is flagged distinctly, since redacting the working tree doesn't remove it from git history — rewriting history to scrub an already-pushed commit is never done without the user explicitly asking for it.
- Name and document the **drift enforcement** concept explicitly: every check this skill performs sorts findings into exactly two buckets, mirroring `agentify-project`'s own mechanical-vs-judgment split — **mechanical** (deterministic, no guessing intent: additive gaps, legacy schema field, a leak with an obvious fix) is fixed directly; **judgment-based** (could revert someone's in-progress work, or is a subjective call: stale references, description-quality calls, self-consistency divergence, a leak requiring a history rewrite) is always reported, never auto-applied. Existing Process steps are labeled with which bucket they fall into.

## Capabilities

### Modified Capabilities
- `skills-catalog-housekeeping`: adds the machine-specific data scan as a new process step and formalizes the mechanical-vs-judgment classification (already implicit in the existing additive-gap/stale-reference behavior) as a named, explicit concept applied consistently across every check.

### New Capabilities
_None — this extends the existing `skills-catalog-housekeeping` capability, not a new one._

## Impact

- Affected paths: `.claude/skills/skills-catalog-housekeeping/SKILL.md` and its `.cursor/skills/` twin (kept byte-identical, per the skill's own self-consistency rule); `openspec/specs/skills-catalog-housekeeping/spec.md` once synced.
- Concrete leaks found and fixed this session as motivating, real-world validation of the new scan step: a private reference repo's name and a local absolute path referencing it (across `openspec/specs/agentify-project-skill/spec.md` and the archived `enhance-agentify-project-staged-multi-harness-setup` change's `design.md`/`proposal.md`/`tasks.md`/synced spec), and this repo's own local checkout path (in the archived `add-skills-catalog-housekeeping-skill` change's `design.md`), were redacted to generic replacements, and — per explicit user request — scrubbed from git history entirely via `git filter-repo`, since the affected commits were already pushed to the public remote.
- No `skills.sh.json` or `README.md` change: this skill remains repo-internal, non-distributed tooling (per the original change's Decision 2).
