# Pre-flight: prerequisite gate and detection map

Two distinct steps, run in this order, before any pipeline stage touches a file. Read this in full before starting a run.

## Step 1 — Uniform prerequisite gate (hard stop)

Three prerequisites, all equal weight, all hard-stop. Check every one before creating a branch or writing anything. If **any** check fails, stop immediately, make no changes of any kind, and report the failing check(s) with the diagnosis and fix below. Do not proceed with a partial or degraded run — there is no "two out of three is fine" mode.

| Check | How to verify | Diagnosis if it fails | Recommended fix to report |
|---|---|---|---|
| Git working tree is clean | `git status --porcelain` produces no output | "The working tree has uncommitted changes, so a dedicated branch can't be cut cleanly from the current state." | "Commit or stash your changes (`git stash -u` for untracked files too), then re-run." |
| `gh` CLI present, authenticated, pushable remote | `gh --version` succeeds; `gh auth status` succeeds; the repo has a remote `gh` can push to | "The `gh` CLI is missing, not authenticated, or this repo has no remote it can push to — a pull request can't be opened without it." | If missing: "Install the GitHub CLI (https://cli.github.com), then run `gh auth login`." If unauthenticated: "Run `gh auth login`." If no pushable remote: "Configure a pushable `origin` remote (`git remote -v` to check, `git remote add origin <url>` to add one)." |
| Node/npm available | `node --version` and `npm --version` both succeed | "Node.js/npm isn't available, and every `npx`-based tool in the supported-tools registry depends on it." | "Install Node.js (https://nodejs.org or your OS package manager), then re-run." |

Report every failing check at once (don't stop at the first one) so the user can fix all of them in one pass before re-running.

## Step 2 — Detection map (read-only)

Only runs once the gate passes. Builds one shared map of "what's already here" that every later pipeline stage reads from — no stage re-detects the same fact twice. Build it read-only, before creating the branch.

Inventory, in one pass:

- **Agent instructions files** — `CLAUDE.md`, `AGENTS.md` at repo root (and per sub-project root for a monorepo in scope); Cursor `.cursor/rules/*.mdc` or legacy `.cursorrules`.
- **Harness directories** — contents of `.claude/`, `.cursor/`, `.agents/` (skills, commands, settings files present in each).
- **Skill classification** — for every skill folder found under `.claude/skills/`, `.cursor/skills/`, `.agents/skills/`, check whether it's listed in root `skills-lock.json`. Present in the lockfile → registry-sourced (trusted, never re-validated). Absent → custom/local (subject to [custom-skill-drift-enforcement.md](custom-skill-drift-enforcement.md)).
- **OpenSpec state** — does `openspec/` exist already.
- **Docs state** — does `docs/ARCHITECTURE.md` exist; does `docs/services/` exist and what's in it.
- **Repomix config** — does `repomix.config.json` (or equivalent) exist, and does its shape match [supported-tools-registry.md](supported-tools-registry.md)'s known-good shape.
- **MCP registration** — project-scoped entries in `.mcp.json` (Claude Code) and `.cursor/mcp.json` (Cursor), specifically whether the codebase-memory MCP server is already registered.
- **Detected tech stack** — see the stack-detection guidance in [agent-readiness-checklist.md](agent-readiness-checklist.md): walk top-level directories for independent manifests rather than assuming one workspace root.

The detection map's findings drive every stage's "is there anything to do here" decision. A pillar the map already shows as satisfied produces no commit for that stage (see [pr-delivery.md](pr-delivery.md)).

## Idempotent re-runs: existing-branch/PR detection

Before creating a new branch, check whether a prior run already opened one that's still open:

1. Look for a branch matching this run's naming convention (`agentify/<date>-setup`, e.g. `agentify/2026-07-setup` — one branch per calendar month is enough granularity; don't mint a new branch per day).
2. Query open PRs from that branch: `gh pr list --head <branch-name> --state open --json number,url`.
3. **If an open PR/branch already exists** — check it out (or fetch it) and push this run's new commits onto it instead of creating a second branch. The PR description gets updated (or a new stage-summary comment added) to reflect the additional commits, rather than opening a duplicate PR.
4. **If no open PR/branch exists** (first run, or the prior one was merged/closed) — create a fresh branch per [pr-delivery.md](pr-delivery.md).

This is why the detection map matters even for idempotency: a second run against an unchanged repo should find every pillar already satisfied, produce zero new commits, and either skip opening a PR entirely or open one whose description says plainly that no changes were needed.
