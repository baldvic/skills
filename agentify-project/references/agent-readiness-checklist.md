# Agent-readiness checklist

Four pillars, plus the stack-detection method they all depend on. For each pillar: how to detect a gap against the detection map (see [preflight-gate-and-detection.md](preflight-gate-and-detection.md)), how to remediate it, and which commit in the PR it lands in. Read this in full before the harness-scaffolding/skill-wiring/permission-hygiene/MCP-setup stages of the pipeline. All four pillars apply to **both** Claude Code and Cursor, always, unless the user's request explicitly scopes to one harness (see `SKILL.md`'s process).

## Stack detection (shared by every pillar below)

Don't assume a single workspace root or JS-monorepo tooling (Nx, Turborepo, pnpm workspaces). Instead:

1. Walk top-level directories for independent manifests: `package.json`, `requirements.txt`/`pyproject.toml`, `go.mod`, `Cargo.toml`, `*.csproj`, etc. Each manifest found marks an independently-versioned component, whether or not a monorepo tool ties them together.
2. For each component found, read its CI workflow files (`.github/workflows/*.yml`, or equivalent) to determine what's *actually* tested — don't assume uniform test coverage across components just because one has a test runner configured. A component with a manifest but no corresponding CI test step is a real gap to note, not an oversight to paper over.
3. The resulting component list feeds every pillar below (which harness dirs to scaffold under, which skills are relevant per component's stack, which components get their own `docs/services/<name>.md` — see [architecture-docs-template.md](architecture-docs-template.md)).

A single-manifest repo is just the degenerate case of this walk (one component) — the same method handles both without a special-cased branch.

## Pillar 1 — Agent instructions

**Detect** (against the detection map):
- **Claude Code** — `CLAUDE.md` and `AGENTS.md` at repo root (and per in-scope sub-project root for a monorepo).
- **Cursor** — `.cursor/rules/*.mdc` project rules, or a legacy single `.cursorrules` file if that's what the repo already uses.
- Staleness signals for any existing file, regardless of harness: commands/paths referenced that no longer exist, missing coverage of a major top-level directory or primary language/framework, or a component the stack walk found that the file never mentions.

**Two files, two jobs (Claude Code)**: `CLAUDE.md` and `AGENTS.md` are not near-duplicates and are not forced into lockstep content.
- `CLAUDE.md` — a short, stable pointer: what the project is, where to look for more, the one or two commands someone always needs.
- `AGENTS.md` — a living, append-only log of learned conventions, gotchas, and decisions that accumulates over time. Don't merge `AGENTS.md`'s fuller content back into `CLAUDE.md`, or vice versa, once that split already exists in the repo.
- For a repo with **no existing split** (new or small), draft one reasonably-sized content set — see [claude-md-template.md](claude-md-template.md) — and don't force the two-file split prematurely; offer it as the repo grows rather than upfront.

**Remediate:**
- **Absent for an in-scope harness** — draft new content from [claude-md-template.md](claude-md-template.md), inferring project overview, build/test/lint commands, and directory map from the repo itself (manifests, config files, CI config, existing docs, and the stack walk above). Never invent a command not discoverable in the repo.
- **Present but stale or incomplete** — compute a merge against the existing file: preserve hand-authored content, add or update only what's missing or demonstrably stale. Never wholesale-replace an existing instructions file unless the user's own request explicitly asks for that specific file to be regenerated from scratch.
- **One harness has instructions, the other in-scope harness doesn't** — draft the missing one with equivalent content so the two stay consistent.

**Delivery:** every write here — new file or merge — is committed on the run's dedicated branch as part of the harness-scaffolding stage (see [pr-delivery.md](pr-delivery.md)); nothing is written directly to the branch the invoking session started on, and nothing pauses for confirmation mid-run — merge-first content behavior is what keeps this a good, reviewable diff, not a safety gate (the PR itself is the safety gate).

## Pillar 2 — Skill wiring

**Detect:**
- Determine the repo's stack via the walk above.
- Cross-reference against skills already installed per the detection map's `skills-lock.json` classification (see [preflight-gate-and-detection.md](preflight-gate-and-detection.md)).
- A gap is: a relevant skill the stack analysis and `find-skills` surface together that isn't already installed.

**Remediate:**
1. Run `find-skills` (the built-in skill-discovery skill) with the detected stack as the query — languages, frameworks, test runners, notable tooling per component.
2. For each relevant match not already present under `.agents/skills/`, install it non-interactively: `npx skills add <source>@<skill> -y`. Let the CLI manage `.agents/skills/` (the canonical store), the per-harness mirrors (`.claude/skills/`, `.cursor/skills/`), and `skills-lock.json` itself — do not manually copy skill folders.
3. Never fabricate a skill that `find-skills` or the registry doesn't actually surface.

**Delivery:** committed as its own skill-wiring stage. Because delivery is a PR, running `-y` non-interactively is just the pipeline's normal invocation — there's no separate "confirm each recommendation" step the way there was in the confirm-per-change model. Skills installed this way are trusted by construction and are excluded from the custom-skill validation pillar entirely (see [custom-skill-drift-enforcement.md](custom-skill-drift-enforcement.md)).

**No relevant skills discoverable** — report "no skill recommendations found" in the PR description as a valid, satisfied outcome; don't install an unrelated skill to have something to show.

## Pillar 3 — Tool-permission hygiene

**Detect:**
- **Claude Code in scope** — whether `.claude/settings.json` (or `.claude/settings.local.json`, whichever convention the repo already uses) exists and, if so, whether its `permissions.allow` list is missing common safe, read-only entries most sessions need (e.g. `git status`, `git log`, `git diff`, read-only build/test invocations already used in the repo's own CI config per the stack walk). Separately note — but never auto-apply — anything that would broaden access: a non-read-only command, a wildcard allow, `permissions.deny` entries, hook configuration.
- **Any in-scope harness without an established, documented, repo-committed permission mechanism** (Cursor, as of this skill's authoring) — nothing to detect a gap against; don't guess at a config file or schema for it.

**Remediate:**
- **Claude Code, missing safe read-only entries** — add them directly to `permissions.allow`. If the settings file doesn't exist at all, creating it with just the safe read-only baseline is the same case.
- **Claude Code, anything that would broaden access** — never apply automatically, regardless of how reasonable it looks (e.g. a command the repo's own CI already runs unattended). Report it as a recommendation in the PR description's stage summary instead, and apply it only if the user's own request explicitly asked for that specific broadening.
- **Harness with no established mechanism** — report this pillar as "not applicable for `<harness>`" in the PR description; write no file for it.

**Delivery:** committed as its own permission-hygiene stage when there's a safe addition to make; produces no commit when the baseline is already present or the only findings are broadening recommendations (those go in the PR description text, not a commit).

This remains a light first pass for Claude Code, not a full permissions audit — point the user to `fewer-permission-prompts` (transcript-driven allowlist tuning) or `update-config` (general settings/hooks/env changes) for anything deeper.

## Pillar 4 — MCP setup

Two different treatments depending on which server is involved — see [supported-tools-registry.md](supported-tools-registry.md) for the codebase-memory MCP, Serena MCP, and repomix entries' full detect/install/scope detail.

**Real, non-interactive install via find-skills (codebase-memory MCP + Serena MCP + repomix specifically):**
- **Detect** — per the supported-tools registry: a matching entry in project-scoped `.mcp.json` (Claude Code) / `.cursor/mcp.json` (Cursor) for the codebase-memory MCP server and for the Serena MCP server; a `repomix.config.json` (or the `repomix-explorer` skill already installed) for repomix.
- **Remediate** — for each of the three, discover its corresponding skill via `find-skills` and install it via the `skills` CLI (`npx skills add <source>@<skill> -y`) if not already present, then follow that skill's own instructions to complete the tool's setup. Agentify-project does not hardcode or duplicate any of these three tools' install/registration commands itself — the installed skill's content is the live source of truth, since each surface can drift independently. MCP registration (codebase-memory, Serena) is **project-scoped only, always** — never global/cross-repo, under any circumstance, even when cross-repo indexing is clearly what would help this repo's actual usage pattern. Where that's genuinely wanted, document the manual global-registration steps in the PR description's "Manual follow-ups" section instead of performing them.

**Advisory only (every other MCP server):**
- **Detect** — from the stack walk, identify categories where a different MCP server would materially help (e.g. a database driver present → that database's MCP server; a ticketing system already used by the team → its MCP server).
- **Remediate** — write a short advisory note per candidate (server name/category + one-line rationale) into the PR description or the agent instructions' "Tooling" section (see [claude-md-template.md](claude-md-template.md)). Never generate actual configuration, an endpoint, or a credential for these — advisory text only, for every harness.

**Delivery:** committed as its own MCP/repomix-setup stage for the real installs; the advisory notes are text in the PR description or instructions file, not a separate commit of their own.

## Idempotency

Every pillar's remediation is computed as a merge/diff against the current detection map, never appended blindly. A second run against a repo where nothing changed since the last run finds every pillar already satisfied — across all four pillars, including tool setup and architecture docs — and proposes no further changes, installs, or PR (see [preflight-gate-and-detection.md](preflight-gate-and-detection.md)'s existing-branch/PR detection for how an already-open prior run's PR is handled instead of opening a duplicate).
