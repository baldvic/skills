## Context

The skills repo already ships `skill-standard` (a meta-skill for authoring skills) and distributes skills via `skills.sh.json`. Several existing skills touch pieces of "agent readiness" individually: `init` (CLAUDE.md only), `fewer-permission-prompts` (settings.json allowlist from transcripts), `update-config` (settings.json/hooks/env in general), `find-skills` (discovering installable skills). `agentify-project` needs to cover the same ground as a single, opinionated, one-shot pass — without silently duplicating or fighting these other skills, and without taking any action a user would consider surprising (overwriting docs, widening permissions, inventing MCP servers).

## Goals / Non-Goals

**Goals:**
- Define one checklist ("agent-readiness") with four pillars: agent instructions file(s), skill wiring, tool-permission hygiene where a harness supports it, MCP server notes.
- For each pillar, define how a gap is *detected* (read-only) and how a fix is *proposed*, with an explicit confirmation gate before any write that overwrites existing content or broadens permissions/trust.
- Make the skill's own agent-instructions generation self-contained (no runtime dependency on the `init` skill), while keeping the generated `CLAUDE.md` content reasonably close to what `init` already produces, so switching from `init` to `agentify-project` is not a regression for Claude Code users.
- **Work across at least Claude Code and Cursor without hardcoding a single harness's file layout.** Two pillars (agent instructions, skill wiring) have a concrete, differing mechanism per harness — the skill detects which harness(es) are in scope and targets the right file(s)/directory for each. The other two pillars (permission hygiene, MCP notes) stay generic/advisory by construction, so they need no per-harness branching.
- Keep the skill harness-agnostic per skill-standard wherever the underlying mechanism actually is generic: detection logic should work from repo inspection (files on disk) and from whatever skill/tool inventory the current harness exposes, not from one vendor's internals.

**Non-Goals:**
- Not replacing `fewer-permission-prompts` or `update-config` as standalone skills — `agentify-project` performs a *minimal, safe* first pass on Claude Code's permissions (common safe read-only allowlist entries) and points to those skills for deeper tuning, rather than reimplementing their full logic.
- Not inventing a permission-file convention for harnesses that don't have an established, documented, repo-committed one. If Cursor (or another harness) later adopts one, extend the checklist then — don't guess now.
- Not auto-installing or auto-authenticating MCP servers. Output is a written recommendation only.
- Not a general skill *authoring* tool — that remains `skill-standard`'s job. `agentify-project` only *wires in* (copies/references) skills that already exist somewhere accessible to the user; it never invents new skill content.
- Not responsible for keeping itself in sync if `init`, `fewer-permission-prompts`, or `update-config` change — noted as a maintenance risk below, not solved here.
- Not committing to exhaustive harness coverage — Claude Code and Cursor are the two concretely designed for; the checklist is structured so a third harness can be added by extending the instructions-file and skill-wiring pillars, not by redesigning the skill.

## Decisions

1. **Agent-instructions ownership, per harness in scope**: `agentify-project` generates/updates the relevant instructions file(s) itself using its own template (`references/claude-md-template.md`, which covers both `CLAUDE.md` and Cursor's equivalent), rather than invoking the `init` skill. Rationale: the user explicitly chose "supersede `init`" so activation isn't ambiguous between two ~overlapping skills, and so `agentify-project` has full control over diffing/confirmation behavior for existing files. Alternative considered — delegate to `init` and layer the rest on top — rejected because it splits confirmation/diff logic across two skills and makes `agentify-project`'s behavior depend on `init`'s (which the user does not control here).

2. **Harness detection precedes both harness-specific pillars**: the skill determines which harness(es) are in scope for a given run by checking for existing harness-specific directories/files in the target repo (`.claude/` implies Claude Code, `.cursor/` implies Cursor), and asks the user when neither is present or both plausibly apply. This single detection step feeds both the agent-instructions pillar (which file(s) to draft/update) and the skill-wiring pillar (which directory/directories to wire into) — it isn't repeated logic per pillar. Rationale: avoids guessing a harness from weak signals, and avoids asking the same question twice.

3. **Four-pillar checklist as a dedicated reference file**: `references/agent-readiness-checklist.md` holds the detection + remediation rules for each pillar (agent instructions, skills, permissions, MCP notes), separate from `SKILL.md`'s process narrative. Rationale: matches skill-standard's own pattern (thin `SKILL.md`, detail in `references/`) and keeps `SKILL.md` under the ~500-line guidance while making the checklist independently reusable/auditable.

4. **Agent instructions cover both CLAUDE.md and Cursor's equivalent, kept consistent**: when Claude Code is in scope, the skill drafts/updates `CLAUDE.md`; when Cursor is in scope, it drafts/updates Cursor project rules (`.cursor/rules/*.mdc`, or the legacy single `.cursorrules` file if that's what the repo already uses) with equivalent content; when both are in scope, it keeps them in sync rather than picking one as canonical and leaving the other stale. Rationale: this is the one pillar where two harnesses genuinely need different file conventions, so the skill must know both rather than assuming Claude Code's convention is universal.

5. **Skill wiring is recommend-first, copy-on-confirm, into whichever harness directory applies**: the skill inspects the target repo's stack (manifest files, config, test runner, etc.) and cross-references against whatever skills are discoverable in the current environment (harness skill listing, or a user-specified local skills directory such as this repo). It presents a shortlist with rationale and only copies a skill's directory into the target's harness-appropriate local skill location — `.claude/skills/<name>/` for Claude Code, `.cursor/skills/<name>/` for Cursor, per this repo's own README precedent — after the user confirms, wiring into both if both harnesses are in scope. Rationale: skill sources vary a lot by harness/user (marketplace, local repo, org registry) — the one thing every harness can support is "copy files the user already trusts," so that's the only mechanism baked in as automatic; the destination directory is the only harness-specific variable.

6. **Permission-hygiene pillar stays scoped to harnesses with an established, documented, repo-committed mechanism**: today that's only Claude Code's `.claude/settings.json` (`permissions.allow`). `agentify-project` may merge in a small, well-known safe allowlist (common read-only git/build/test commands) automatically there, mirroring the "safe" subset `fewer-permission-prompts` already establishes as precedent. Any change that would widen an existing rule, add a non-read-only command, or touch `permissions.deny`/hook config requires explicit confirmation and a shown diff. When Cursor (or another harness) is the one in scope and has no equivalent established repo-committed convention, this pillar is report-only for that harness — the skill states plainly that it found no applicable mechanism rather than fabricating one. Rationale: guessing a config schema we're not confident is current/correct is worse than admitting the gap; this keeps the one-shot pass useful without ever risking either silently granting access or writing a file the harness won't recognize.

7. **MCP notes are advisory text only**, written into the audit report / agent-instructions "Tooling" section as suggestions with names and why they'd help (e.g. "a Postgres MCP server would let the agent query the schema directly") — never a generated config with endpoints or credentials, and identical regardless of harness since no config is ever produced. Rationale: MCP server availability and credentials are host/account-specific; guessing or fabricating them is worse than omitting them.

8. **Idempotency via diff-before-write**: every write (agent instructions, settings.json) is computed as a diff against the current file (if any) and shown before applying, so re-running the skill on an already-agentified repo converges instead of duplicating sections.

## Risks / Trade-offs

- [Overlap/drift with `init`, `fewer-permission-prompts`, `update-config` as those skills evolve independently] → Document the relationship explicitly in `agentify-project`'s `SKILL.md` ("supersedes `init`'s CLAUDE.md generation; for deeper permission tuning see `fewer-permission-prompts`") so future edits to either side surface the coupling instead of silently diverging.
- [Skill-wiring recommendations are only as good as what's discoverable in the current harness/session] → Be explicit in the checklist that "no skills found to recommend" is a valid, reported outcome, not a bug to work around by inventing skills.
- [Users may run this on a repo they don't want touched broadly] → Every destructive/broadening action requires confirmation; default behavior on ambiguity is "report only, don't write."
- [Definition of "agent-ready" is opinionated and may not fit every org's conventions] → Keep the checklist in a separate reference file specifically so it's easy to fork/adjust without touching `SKILL.md`'s control flow.
- [Cursor's project-rules and permission conventions may change format over time, or this design may misjudge the current one] → Keep the Cursor-specific detail minimal and isolated to the two pillars where it's unavoidable (instructions file, skill directory), and treat anything uncertain (like a Cursor permission-file schema) as out of scope rather than guessed — see Decision 6.
- [Harness detection based on `.claude/`/`.cursor/` directory presence can be wrong or ambiguous (e.g. a repo with neither yet, or a template repo copied from elsewhere with a stale directory)] → When detection is ambiguous or absent, ask the user which harness(es) to target rather than guessing silently.

## Migration Plan

Purely additive: new skill directory, no existing skill files touched. No rollback beyond removing the new directory and its `skills.sh.json` entry if added.

## Open Questions

- Should `agentify-project` eventually get a `--report-only` mode surfaced as an explicit skill argument, or is "ask before every write" sufficient? Left as a future refinement; current design defaults to always-confirm for writes, which already covers the report-only use case in practice.
