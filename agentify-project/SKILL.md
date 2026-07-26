---
name: agentify-project
description: Makes a codebase ready for AI coding agents in one non-interactive staged pipeline — always scaffolding both Claude Code and Cursor, installing real tooling from an extensible supported-tools registry (OpenSpec CLI, the skills CLI, and repomix/codebase-memory MCP/Serena MCP each set up by discovering and installing their corresponding skill via find-skills), wiring in relevant skills, validating custom skills against skill-standard, and generating architecture docs — delivered as a single reviewable pull request after a hard pre-flight prerequisite gate (clean git tree, authenticated `gh`, Node/npm). Use when the user asks to "agentify" a repo, make a project agent-ready, onboard a codebase for Claude Code and/or Cursor, or set up agent scaffolding from scratch.
license: MIT
---

# Agentify Project

A non-interactive, staged pipeline that makes a codebase ready for AI coding agents, delivered as a single pull request for the repo owner to review and merge. This skill **supersedes the built-in `init` skill**: it drafts and updates agent instructions itself. Do not invoke `init` alongside it.

Related skills, for narrower follow-up work this skill doesn't fully replace:
- **`fewer-permission-prompts`** — deeper `.claude/settings.json` allowlist tuning from transcript history, beyond the safe baseline this skill adds automatically.
- **`update-config`** — general `settings.json`/hooks/env changes outside the safe-baseline scope.
- **`find-skills`** — broader skill discovery, invoked internally by the skill-wiring stage below.

## Safety model

Nothing pauses mid-run to ask for confirmation. Safety instead comes from two independent, structural mechanisms:

1. **A uniform pre-flight prerequisite gate**, run before any branch is created or file touched — see [references/preflight-gate-and-detection.md](references/preflight-gate-and-detection.md). Git working tree clean, `gh` present/authenticated/with a pushable remote, and Node/npm available are treated as **equally hard** prerequisites. If any is missing, the skill stops immediately, makes no changes, and reports a diagnosis with a concrete fix for each failing check.
2. **Delivery as a single pull request.** Every change lands on a dedicated branch, one commit per pipeline stage, pushed and opened as a PR via `gh pr create` — never written directly to the branch the invoking session started on. Nothing is real until a human reviews and merges it. See [references/pr-delivery.md](references/pr-delivery.md).

Content-level changes are still computed as **merges** (preserve hand-authored content, add what's missing) rather than wholesale replacements — this is good PR hygiene that produces a smaller, more reviewable diff, not the primary safety guarantee. The PR gate is.

## Process

Run the stages below in order. Stop immediately if step 1 fails. Nothing after step 1 pauses for confirmation — proceed straight through to the PR.

1. **Pre-flight prerequisite gate (hard stop).** Verify git-clean, `gh`-usable, and Node/npm-present, all as equally-weighted hard requirements. If any check fails: stop, make zero changes, report which check(s) failed and the recommended fix for each. Full detail: [references/preflight-gate-and-detection.md](references/preflight-gate-and-detection.md).
2. **Detection map.** Once the gate passes, build one read-only map of what's already here — instructions files, `.claude`/`.cursor`/`.agents` contents (including registry-sourced vs. custom/local skill classification via `skills-lock.json`), `openspec/`, `docs/`, repomix config, MCP registration, detected tech stack. Every later stage reads from this map instead of re-detecting facts. Also check here whether a prior run's PR/branch is still open — reuse it instead of opening a duplicate. Same reference as above.
3. **Create the dedicated branch** (or check out the existing open one from a prior run, per step 2).
4. **Tool setup**, driven by the extensible supported-tools registry: [references/supported-tools-registry.md](references/supported-tools-registry.md). Initial entries — OpenSpec CLI, the `skills` CLI, repomix, codebase-memory MCP, Serena MCP — each installed/configured only if the detection map shows it's missing. For repomix, codebase-memory MCP, and Serena MCP specifically, "installed" means discovering the corresponding skill via `find-skills` and installing it via the `skills` CLI, then following that skill's own instructions — not a bespoke command agentify-project runs itself. Repo-scoped output is included in this stage's commit; machine-global output (e.g. a global CLI install) is a local prerequisite step, never part of the PR diff.
5. **Repo/stack analysis.** Walk top-level directories for independent manifests (polyglot, non-monorepo-tooled repos included) and read each component's CI config for actual test coverage. Detail: [references/agent-readiness-checklist.md](references/agent-readiness-checklist.md).
6. **Harness scaffolding — both, always.** Scaffold Claude Code and Cursor unconditionally — a fixed set of exactly two harnesses — regardless of which harness directories currently exist, unless the user's own request explicitly scopes the run to one harness. Generate/merge `CLAUDE.md` + `AGENTS.md` (Claude Code) and Cursor project rules, per [references/agent-readiness-checklist.md](references/agent-readiness-checklist.md) and [references/claude-md-template.md](references/claude-md-template.md). Committed as its own stage.
7. **Skill wiring.** Use `find-skills` against the detected stack, then `npx skills add <source>@<skill> -y` for each relevant, not-yet-installed match — no manual folder copying. Detail: [references/agent-readiness-checklist.md](references/agent-readiness-checklist.md).
8. **Permission hygiene.** For Claude Code, add missing safe read-only allowlist entries automatically. Any change that would broaden access is **reported in the PR description**, not applied, unless the user's request explicitly asked for that broadening. Cursor has no established mechanism — report not-applicable. Detail: [references/agent-readiness-checklist.md](references/agent-readiness-checklist.md).
9. **MCP + repomix setup.** Real, non-interactive install — via `find-skills` + the `skills` CLI, then that skill's own instructions — for repomix, the codebase-memory MCP server, and the Serena MCP server (project-scoped registration **only, always** for the two MCP servers — global/cross-repo registration is never automatic, documented as a manual PR follow-up instead). Every other MCP server stays advisory text only. Detail: [references/supported-tools-registry.md](references/supported-tools-registry.md) and [references/agent-readiness-checklist.md](references/agent-readiness-checklist.md).
10. **Custom-skill validation.** Classify every skill found in step 2 as registry-sourced (trusted, skip) or custom/local (validate against `skill-standard`). Mechanical violations get fixed and committed as their own commit; judgment-based ones get reported in the PR description only. Detail: [references/custom-skill-drift-enforcement.md](references/custom-skill-drift-enforcement.md).
11. **Architecture documentation.** Generate or merge `docs/ARCHITECTURE.md`, plus `docs/services/<name>.md` per component for multi-service repos (skipped entirely for single-service repos). Grounded strictly in what the stack analysis found. Detail: [references/architecture-docs-template.md](references/architecture-docs-template.md).
12. **Push and open the PR.** Push the branch; open (or update, if reusing an existing open PR) a single pull request whose description summarizes every stage's outcome, including a "Manual follow-ups" section for anything that couldn't be automated. Detail: [references/pr-delivery.md](references/pr-delivery.md).

A stage with nothing to do (everything already satisfied per the detection map) produces no commit — it's noted as "already satisfied" in the PR description and the pipeline continues to the next stage without pausing.

## Pillars covered by the pipeline

| Pillar | What "ready" looks like | Both harnesses? | Where it's committed |
|---|---|---|---|
| Agent instructions | `CLAUDE.md`/`AGENTS.md` (Claude Code), Cursor project rules — present, matches current repo state | Yes | Harness-scaffolding stage |
| Skill wiring | Relevant skills from `find-skills` installed via the `skills` CLI | Yes | Skill-wiring stage |
| Permission hygiene | Safe read-only commands allowlisted where the harness has a mechanism (currently Claude Code only) | Partial | Permission-hygiene stage |
| MCP + repomix | codebase-memory MCP, Serena MCP, repomix each really installed via find-skills (MCP registration project-scoped); other MCP servers advisory only | Yes | MCP/repomix-setup stage |
| Custom-skill drift | Non-registry skills validated against `skill-standard`; mechanical fixes applied | Yes | Custom-skill-validation stage |
| Architecture docs | `docs/ARCHITECTURE.md` (+ `docs/services/*` if multi-component) | N/A (harness-agnostic) | Architecture-docs stage |

## Non-negotiable rules

- Never proceed past the pre-flight gate with a degraded/partial run — all three prerequisites are equally hard-stop.
- Never write pipeline output directly to the branch the invoking session started on — everything lands via the dedicated branch and its PR.
- Never wholesale-replace an existing instructions or docs file — merge, preserving hand-authored content — unless the user's own request explicitly asks for that specific file to be regenerated from scratch.
- Never widen an existing `.claude/settings.json` permission, add a non-read-only command, or touch `permissions.deny`/hooks without the user's request explicitly asking for that broadening — report it in the PR description instead.
- Never register the codebase-memory MCP server, the Serena MCP server, or any tool globally/cross-repo automatically — project-scoped only; document global registration as a manual follow-up.
- Never hardcode or duplicate install/registration commands for repomix, codebase-memory MCP, or Serena MCP — discover and install the corresponding skill via `find-skills` + the `skills` CLI, and follow that skill's own instructions instead.
- Never fabricate a skill that `find-skills` doesn't actually surface, and never re-validate a registry-sourced skill against `skill-standard`.
- Never auto-rewrite a custom skill's judgment-based content (description quality, step actionability) — only mechanical/structural violations are auto-fixed.
- Never invent a service, endpoint, or data model in architecture docs that isn't grounded in something the stack analysis actually found.
- Re-running on an already-agentified, unchanged repo should converge: every pillar reports satisfied, no new commits, no duplicate PR.

## Edge cases

- **Monorepo / polyglot repo with multiple components** — the stack-detection walk (see [references/agent-readiness-checklist.md](references/agent-readiness-checklist.md)) finds each independently-versioned component; architecture docs get the `docs/services/` tier, skill wiring considers each component's stack.
- **Not a git repository** — the pre-flight gate's git-clean check has nothing to verify against; treat this as a gate failure (no repo to branch from) rather than proceeding.
- **Both harnesses in scope, only one has an instructions file** — draft the missing one with equivalent content rather than leaving it out.
- **User request explicitly scopes to one harness** — scaffold only that harness; skip the other entirely for this run.
- **A prior run's PR is still open** — push new commits onto that branch/PR instead of opening a second one (see [references/preflight-gate-and-detection.md](references/preflight-gate-and-detection.md)).
- **Everything already satisfied** — no branch, no commits, no PR with file changes; report that plainly (an information-only PR describing "no changes needed" is acceptable but not required).
