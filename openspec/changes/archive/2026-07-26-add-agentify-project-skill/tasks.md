## 1. Scaffold the skill directory

- [x] 1.1 Create `agentify-project/` with `SKILL.md`, `references/`, `evals/` following skill-standard's directory layout (see `skill-standard/references/agent-skills-format.md`)
- [x] 1.2 Write frontmatter: `name: agentify-project`, a description covering what (agent-readiness audit + CLAUDE.md/skills/permissions/MCP setup) and when (activate on "agentify this repo", "make this agent-ready", "set this project up for Claude Code/agents", etc.), `license: MIT`

## 2. Write the agent-readiness checklist reference

- [x] 2.1 Create `references/agent-readiness-checklist.md` covering the four pillars (CLAUDE.md, skill wiring, `.claude/settings.json` permissions, MCP notes), each with: detection method, remediation, and confirmation requirement — per `design.md` Decisions 2-5
- [x] 2.2 Define the safe read-only settings.json allowlist (git status/log/diff, common build/test read commands) that may be added without confirmation, and explicitly list what always requires confirmation (broadening, non-read-only additions, `permissions.deny`, hooks)
- [x] 2.3 Document the "no delegation to `init`" relationship and the "advisory-only MCP" rule inline so the checklist is self-contained

## 3. Write the CLAUDE.md template reference

- [x] 3.1 Create `references/claude-md-template.md` with the sections `agentify-project` should draft into a new `CLAUDE.md` (project overview, build/test/lint commands, code conventions, directory map, tooling/MCP notes section)
- [x] 3.2 Document the diff-and-confirm procedure for updating an existing `CLAUDE.md` (never blind-overwrite)

## 4. Write SKILL.md process flow

- [x] 4.1 Write the step-by-step process: (a) run read-only audit across all four pillars, (b) present findings report, (c) for each gap, propose a fix and apply per the confirmation rules in the checklist, (d) summarize what changed vs what still needs manual follow-up
- [x] 4.2 Cross-link `references/agent-readiness-checklist.md` and `references/claude-md-template.md`; note the relationship to `init`, `fewer-permission-prompts`, and `update-config` per `design.md` Risk 1
- [x] 4.3 Keep `SKILL.md` body under ~500 lines; push detail into references

## 5. Add evals

- [x] 5.1 Create `evals/evals.json` with 2-3 realistic prompts (fresh repo with nothing, repo with partial setup, repo already agent-ready) per skill-standard's eval-artifacts schema
- [x] 5.2 Add fixture repos/snippets under `evals/files/` if needed to exercise each scenario in the spec

## 6. Validate against skill-standard

- [x] 6.1 Apply every item in `skill-standard/references/validation.md` to the new `SKILL.md` and fix issues
- [x] 6.2 Confirm directory name equals `name` field, frontmatter uses only allowed keys, no host-specific absolute paths or secrets appear anywhere in the skill tree

## 7. Package

- [x] 7.1 Add an entry for `agentify-project` in `skills.sh.json` alongside the existing `skill-standard` entry, matching its existing schema/fields
- [x] 7.2 Confirm `evals/` is excluded from whatever packaging step ships the skill folder, per skill-standard's Packaging section

## 8. Generalize for multi-harness support (Claude Code + Cursor)

- [x] 8.1 Update `SKILL.md` frontmatter `description` and body to name both Claude Code and Cursor explicitly, and rephrase the process/pillars summary in harness-agnostic terms except where a mechanism genuinely differs (agent instructions file, skill-wiring directory) — per `design.md` Decisions 2 and 4-6
- [x] 8.2 Add a harness-detection step to `SKILL.md`'s process (before acting on the agent-instructions or skill-wiring pillars): check for `.claude/` and `.cursor/` directories, ask the user when neither or ambiguous — per `design.md` Decision 2 and the new "Harness detection" requirement
- [x] 8.3 Rewrite `references/agent-readiness-checklist.md` Pillar 1 to cover both `CLAUDE.md` (Claude Code) and Cursor project rules (`.cursor/rules/*.mdc` or existing `.cursorrules`), including keeping both in sync when both harnesses are in scope
- [x] 8.4 Rewrite `references/agent-readiness-checklist.md` Pillar 2 to wire into `.claude/skills/` and/or `.cursor/skills/` depending on which harness(es) are in scope
- [x] 8.5 Rewrite `references/agent-readiness-checklist.md` Pillar 3 to explicitly scope automatic/confirmed remediation to Claude Code's `.claude/settings.json`, and make explicit that harnesses without an established repo-committed permission convention (Cursor, as of this writing) get a report-only "not applicable" outcome instead of a fabricated config
- [x] 8.6 Broaden `references/claude-md-template.md` to cover drafting content for both `CLAUDE.md` and Cursor project rules from the same underlying draft, and note the keep-in-sync procedure when both exist
- [x] 8.7 Add or revise an eval in `evals/evals.json` covering a Cursor-only or mixed Claude Code + Cursor repo, with fixtures under `evals/files/` if helpful
- [x] 8.8 Re-run the skill-standard validation checklist (references/validation.md) against the revised `SKILL.md` and fix any issues introduced by these edits
