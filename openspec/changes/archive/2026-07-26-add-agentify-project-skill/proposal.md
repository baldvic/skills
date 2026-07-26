## Why

Agent coding tools (Claude Code, Cursor, and similar) work far better in repos that are explicitly set up for them: current agent instructions, the right skills wired in, sane tool permissions where the harness supports them, and any useful MCP servers noted. Today that setup is manual, inconsistent, and usually only done for one harness at a time — the built-in `init` skill only covers `CLAUDE.md` generation for Claude Code, leaving skill discovery, permissions, MCP wiring, and other harnesses like Cursor undone. We need one skill that does the whole "make this repo agent-ready" pass in a single, safe, repeatable run — without hardcoding a single vendor's file layout where a mechanism genuinely differs by harness.

## What Changes

- Add a new skill `agentify-project` at `agentify-project/SKILL.md`, authored per the skill-standard Agent Skills format.
- The skill audits an arbitrary target codebase against an "agent-readiness" checklist covering: presence/freshness of agent instructions (`CLAUDE.md` for Claude Code, `AGENTS.md`/Cursor project rules for Cursor and other tools), discoverable and relevant skills wired into whichever harness-specific skill directory applies (`.claude/skills/`, `.cursor/skills/`), tool-permission hygiene where a harness has an established repo-committed mechanism (currently Claude Code's `.claude/settings.json`), and candidate MCP servers worth noting.
- The skill generates/updates the target harness(es)' agent instructions file(s) itself (does not delegate to the built-in `init` skill) — **this supersedes `init`'s role** for anyone using `agentify-project`, and covers Cursor's equivalent alongside Claude Code's `CLAUDE.md`.
- The skill proposes fixes for detected gaps and only applies changes with explicit confirmation for destructive or broad actions (overwriting existing instructions, widening permissions); MCP server suggestions are advisory only, never auto-installed. It never fabricates a permission-file format for a harness that doesn't have an established one.
- Ships with a reference checklist (`references/agent-readiness-checklist.md`) defining what "agent-ready" means per harness and how each gap is detected, so the audit logic isn't buried only in prose in `SKILL.md`.
- Includes `evals/evals.json` per skill-standard so the skill's activation and output can be graded, including at least one Cursor-oriented scenario.

## Capabilities

### New Capabilities
- `agentify-project-skill`: Defines the behavior of the `agentify-project` skill — the agent-readiness checklist it audits against, what it generates/modifies (agent instructions file(s), skill wiring, harness-specific permission files, MCP notes) across at least Claude Code and Cursor, and the safety rules around confirmation before overwriting, broadening permissions, or fabricating a harness mechanism that isn't established.

### Modified Capabilities
_None — this is a new, additive skill in the skills repo; no existing spec's requirements change._

## Impact

- Affected paths: new `agentify-project/` directory (SKILL.md, references/, evals/) in this repo, alongside the existing `skill-standard/` skill.
- No code/runtime systems affected — this repo only distributes skill definitions (see `skills.sh.json`), so impact is limited to the shipped skill content and its packaging entry.
- Depends on `skill-standard` for format rules, validation checklist, and eval-artifact conventions; must stay conformant if skill-standard's format changes.
