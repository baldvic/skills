## Context

`skills-catalog-housekeeping` previously scoped every read and write to exactly two files (`skills.sh.json`, `README.md`), by explicit non-negotiable rule. A real leak found this session — a private/internal repo's name and local absolute paths, committed and pushed across several `openspec/` design docs — falls entirely outside that scope, so the skill had no way to catch it. `agentify-project` already has a named mechanical-vs-judgment classification (`references/custom-skill-drift-enforcement.md`) for a structurally similar problem (auto-fix safe things, report risky ones); this skill's existing behavior already followed that pattern implicitly (additive gaps fixed directly, stale references reported) without ever naming it.

## Goals / Non-Goals

**Goals:**
- Give the skill a real, on-demand check for machine-specific data leaking into this public repo, scoped repo-wide rather than to the two files it otherwise edits.
- Name the mechanical-vs-judgment split explicitly as "drift enforcement," so it reads as one deliberate concept applied consistently, not five separate ad hoc rules.
- Keep the skill's write scope narrow by default — the leak scan is an explicit, called-out exception, not a silent widening of what this skill is allowed to touch.

**Non-Goals:**
- Not automating git-history rewrites — scrubbing an already-pushed leak is destructive and stays a judgment-based, user-confirmed action, never something this skill (or any skill) does on its own initiative.
- Not turning this into a general secret scanner (API keys, credentials) — scope stays specifically machine/environment fingerprints (paths, usernames, private repo names), matching what this repo's own risk actually is (a solo maintainer's local machine, not a team's credential store).

## Decisions

1. **The machine-specific data scan is the one deliberate exception to the skill's file-scope rule.** Every other check stays scoped to `skills.sh.json`/`README.md`; this one explicitly sweeps the whole repo, and its non-negotiable-rules entry says so directly rather than leaving the file-scope rule looking violated. Rationale: the leak that motivated this change couldn't be caught any other way — it lived in `openspec/` docs, nowhere near either of the two files this skill normally edits.
2. **A confirmed leak with an obvious generic replacement is mechanical (fixed directly); a leak requiring a git-history rewrite is judgment-based (reported only).** This mirrors the same reasoning `agentify-project` uses for mechanical vs. judgment violations: a text substitution that doesn't change anyone's intent is safe to apply; rewriting shared/public git history is a destructive, hard-to-reverse action on a shared repo, which must always be a human's explicit call, never a skill's automatic one — regardless of how "obviously right" the fix seems.
3. **"Drift enforcement" is documented as a named section, not left implicit.** The skill already behaved this way (additive gaps auto-fixed, stale references reported) before this change; this just gives that behavior one name and states it once, then labels each existing Process step with which bucket it falls into, so a future addition to this skill has an explicit pattern to slot into rather than reinventing the split per-check.

## Risks / Trade-offs

- [Repo-wide scanning is a broader read surface than this skill previously needed] → Read-only; the only write exception (redacting a confirmed leak) still requires explicit user confirmation before touching anything outside `skills.sh.json`/`README.md`.
- [A leak already on the pushed default branch can't be fully remediated by this skill alone] → By design: flagged distinctly and left to the user to decide whether a history rewrite is worth the disruption (rewritten hashes, required force-push) — not something to auto-attempt.

## Migration Plan

Purely additive to `skills-catalog-housekeeping`'s existing process — no other skill's behavior changes. Rollback is reverting the two `SKILL.md` copies to their prior content if this scan proves too broad or noisy in practice.
