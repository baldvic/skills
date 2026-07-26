## 1. Rewrite SKILL.md around the pre-flight gate + PR-delivered pipeline

- [x] 1.1 Replace the process section with: pre-flight prerequisite gate (hard stop) → detection map → tool setup → repo/stack analysis → harness scaffolding (both) → skill wiring → permission hygiene → MCP/repomix setup → architecture docs → push branch + open PR — per `design.md` Decisions 1, 2, 4
- [x] 1.2 Document the uniform prerequisite gate explicitly at the top of the process: git-clean, `gh` present/authenticated/pushable, Node/npm present — all equal, stop-with-diagnosis-and-fix-recommendation if any missing, before any branch or file change (Decision 4)
- [x] 1.3 Document branch + one-commit-per-stage + push + `gh pr create` as the delivery mechanism, replacing all "write directly" language with "commit on the dedicated branch" (Decision 2)
- [x] 1.4 Keep merge-first content semantics (preserve hand-authored content, add what's missing) as PR-hygiene guidance, distinct from the prerequisite gate's hard-stop safety role (Decision 3)
- [x] 1.5 Change the harness-scope framing to "both, always, fixed at exactly Claude Code + Cursor" unless the user's request explicitly scopes narrower (Decision 5)
- [x] 1.6 Update frontmatter `description` to mention PR-based delivery, the pre-flight prerequisite gate, both harnesses by default, and the extensible supported-tools registry (not a fixed four)
- [x] 1.7 Keep `SKILL.md` under ~500 lines; push expanded pillar detail into references

## 2. Add a pre-flight gate + detection-map reference

- [x] 2.1 Create `references/preflight-gate-and-detection.md` documenting the uniform prerequisite gate (git-clean, `gh` present/authenticated/pushable remote, Node/npm) with exact diagnosis + recommended-fix wording per failure mode — per `design.md` Decision 4
- [x] 2.2 Document the detection-map build (existing instructions files, `.claude`/`.cursor`/`.agents` contents plus `skills-lock.json` classification into registry-sourced vs. custom/local, `openspec/`, `docs/`, repomix configs, MCP registration, detected stack) that every later stage reads from — per `design.md` Decision 13
- [x] 2.3 Document the branch-naming and existing-open-PR detection logic for idempotent re-runs (push more commits to an existing open PR/branch rather than opening a duplicate) — per `design.md` Decision 2 and the "Idempotent re-runs" requirement

## 3. Add a PR-delivery mechanics reference

- [x] 3.1 Create `references/pr-delivery.md` documenting: branch creation, one commit per applicable stage, push, `gh pr create` invocation, and the required PR description structure (summary per stage + explicit "manual follow-ups" section for anything that couldn't be automated, e.g. global MCP registration)
- [x] 3.2 Document that a stage with nothing to do produces no commit, and is noted as "already satisfied" in the PR description rather than a no-op commit

## 4. Update the agent-readiness checklist reference for merge-first content + PR framing

- [x] 4.1 Rewrite the agent-instructions pillar to cover `CLAUDE.md` + `AGENTS.md` (not just `CLAUDE.md`) + Cursor rules, including the "short pointer vs. living log" split, and note these are committed on the dedicated branch — per `design.md` Decision 6
- [x] 4.2 Rewrite the skill-wiring pillar to use `find-skills` + `npx skills add <source>@<skill> -y`, and to explicitly state that registry-installed skills are trusted and excluded from the custom-skill validation pillar (task group 6) — per `design.md` Decision 10
- [x] 4.3 Rewrite the permission-hygiene pillar so broadening changes are reported-in-the-PR-description, not applied, unless the user's request explicitly asked for that broadening
- [x] 4.4 Rewrite the MCP-notes pillar to split "advisory only" (unknown/generic MCP servers) from "real, project-scoped-only install via find-skills" (codebase-memory MCP **and Serena MCP**, each discovered as a skill via `find-skills` + installed via the `skills` CLI, with agentify-project no longer hardcoding either server's install commands), with global/cross-repo registration always deferred to a documented manual PR follow-up — per `design.md` Decisions 7 and 8

## 5. Add an extensible supported-tools registry reference

- [x] 5.1 Create `references/supported-tools-registry.md` (or a structured `references/supported-tools.json` alongside a short prose intro) defining the entry shape: `detect`, `install`, `scope` (`repo` vs `machine-global`), idempotency check — per `design.md` Decision 7
- [x] 5.2 Populate the initial five entries: OpenSpec CLI (detect `openspec --version`; install `npm install -g openspec`; machine-global CLI producing a repo-scoped `openspec init` deliverable, run only if `openspec/` absent), `skills` CLI (detect `npx skills` resolves; install devDependency add for Node projects; repo-scoped), repomix (detect `repomix.config.json` or the `repomix-explorer` skill already installed; install via `find-skills` discovery + `npx skills add yamadashy/repomix@repomix-explorer -y`, then follow that skill's own instructions; repo-scoped), codebase-memory MCP (detect project-scoped `.mcp.json`/`.cursor/mcp.json` entry or the `codebase-memory-mcp-intelligence` skill already installed; install via `find-skills` discovery + `npx skills add aradotso/mcp-skills@codebase-memory-mcp-intelligence -y`, then follow that skill's own instructions; repo-scoped, project-only — never global), Serena MCP (detect project-scoped `.mcp.json`/`.cursor/mcp.json` entry or the `setup-serena-mcp` skill already installed; install via `find-skills` discovery + `npx skills add neolabhq/context-engineering-kit@setup-serena-mcp -y`, then follow that skill's own instructions; repo-scoped, project-only — never global)
- [x] 5.3 Document how to add a sixth tool later (same entry shape, no pipeline changes needed) so the registry's extensibility is explicit, not just implied
- [x] 5.4 Reframe the known-good `repomix.config.json` shape (XML output, summary/directory-structure on, compression on, gitignore-aware, security check on, excluding skill-store paths) as fallback/descriptive context only — agentify-project no longer authors this config directly; it's produced (or not) by the installed `repomix-explorer` skill's own instructions
- [x] 5.5 Document the "global/cross-repo MCP registration is never automatic — always a documented manual PR follow-up" rule explicitly for both the codebase-memory MCP and Serena MCP entries, so this doesn't quietly regress toward auto-applying global state

## 6. Add a custom-skill validation and drift-enforcement reference

- [x] 6.1 Create `references/custom-skill-drift-enforcement.md` documenting the registry-sourced vs. custom/local classification (via `skills-lock.json` presence) — per `design.md` Decision 9
- [x] 6.2 Document the mechanical-vs-judgment split for `skill-standard` validation violations: mechanical (frontmatter fields, directory-name mismatch, disallowed keys, absolute paths, description length) get auto-fixed and committed as their own PR commit; judgment-based (description quality, step actionability) get reported in the PR description only, never rewritten
- [x] 6.3 Cross-reference `skill-standard/references/validation.md` directly rather than duplicating its checklist

## 7. Add an architecture-documentation reference

- [x] 7.1 Create `references/architecture-docs-template.md` mirroring the reference repo's `docs/ARCHITECTURE.md` structure (system context, component/container diagram, service/component catalog, messaging topology if present, cross-component sequences, shared data model, known issues) using Mermaid + tables
- [x] 7.2 Document the `docs/services/README.md` + `docs/services/<name>.md` per-component tier for multi-service repos, and the rule to skip it entirely for single-service repos
- [x] 7.3 Document the merge behavior for existing docs (preserve hand-authored prose, update only demonstrably stale sections)

## 8. Generalize stack detection

- [x] 8.1 Update the checklist's stack-detection guidance to walk top-level directories for independent manifests (`package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, etc.) instead of assuming a single workspace root or JS monorepo tooling — per `design.md` Decision 12
- [x] 8.2 Document reading each component's CI workflow files to determine actual per-component test coverage rather than assuming uniform coverage

## 9. Update evals

- [x] 9.1 Rewrite existing evals in `evals/evals.json` to reflect PR-based delivery (branch + staged commits + PR, not direct writes), the pre-flight prerequisite gate, and the find-skills-driven MCP/repomix setup (codebase-memory MCP, Serena MCP, repomix all installed as skills, not bespoke commands)
- [x] 9.2 Add an eval covering a hard-stop pre-flight failure (e.g. dirty working tree, or `gh` unauthenticated) verifying the skill makes zero changes and reports a diagnosis + fix
- [x] 9.3 Add an eval covering the full staged pipeline on a clean repo with nothing set up (both harnesses scaffolded, all registry tools set up, docs generated, delivered as one PR with per-stage commits)
- [x] 9.4 Add an eval covering a polyglot multi-service repo to exercise generalized stack detection and the `docs/services/` tier
- [x] 9.5 Add an eval covering a custom/local skill with a mechanical violation (gets auto-fixed and committed) alongside one with only a judgment-based issue (gets reported, not rewritten), and a registry-installed skill (excluded from validation entirely)
- [x] 9.6 Add an eval verifying global/cross-repo MCP registration is never auto-applied (for both codebase-memory MCP and Serena MCP), only documented as a manual PR follow-up
- [x] 9.7 Add an eval verifying idempotency against an already-open prior-run PR (pushes more commits, doesn't open a duplicate)
- [x] 9.8 Add fixtures under `evals/files/` as needed for the new evals
- [x] 9.9 Add or update an eval verifying the codebase-memory MCP, Serena MCP, and repomix registry entries are each satisfied by discovering the corresponding skill via `find-skills` and installing it via the `skills` CLI, not by agentify-project's own hardcoded install commands

## 10. Validate against skill-standard

- [x] 10.1 Apply every item in `skill-standard/references/validation.md` to the revised `SKILL.md` and all new/changed reference files
- [x] 10.2 Confirm no host-specific absolute paths or secrets were introduced anywhere in the skill tree, especially in any reference-repo-derived examples (genericize any repo-specific names/paths used as illustrations)

## 11. Package

- [x] 11.1 Confirm `skills.sh.json`'s existing `agentify-project` entry still applies (no change expected, but verify after the description/scope update)
