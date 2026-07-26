# PR delivery mechanics

How the staged pipeline's output actually reaches the repo: a dedicated branch, one commit per applicable stage, a push, and a single `gh pr create`. Read this before starting the branch-and-commit sequence, once the pre-flight gate has passed (see [preflight-gate-and-detection.md](preflight-gate-and-detection.md)).

## Branch creation

- Name the branch `agentify/<year>-<month>-setup` (e.g. `agentify/2026-07-setup`) — see [preflight-gate-and-detection.md](preflight-gate-and-detection.md)'s idempotency section for reusing an existing open one instead of creating a fresh branch every run.
- Cut it from the current default branch tip, only after the prerequisite gate has confirmed a clean working tree.

## One commit per applicable stage

Apply the pipeline stages in order, each as its own commit, only when that stage actually has something to do (per the detection map):

1. Tool setup (per-tool commits are fine if that's cleaner, or one combined commit — group by what changed, not by rule)
2. Harness scaffolding (both harnesses)
3. Skill wiring
4. Permission hygiene
5. MCP + repomix setup
6. Custom-skill mechanical drift fixes (its own commit, separate from everything else — see [custom-skill-drift-enforcement.md](custom-skill-drift-enforcement.md))
7. Architecture docs

Write a short, conventional commit message per stage (e.g. `agentify: scaffold Claude Code + Cursor harnesses`, `agentify: add repomix + codebase-memory MCP config`). A stage with nothing to do — every fact in the detection map already satisfied — produces **no commit at all**; don't create an empty or trivial commit just to mark a stage as "visited." Note it as already-satisfied in the PR description instead (see below).

## Push and open the PR

- Push the branch: `git push -u origin <branch-name>` (or a plain `git push` if the branch already tracks its remote from a prior run).
- Open the PR non-interactively: `gh pr create --title "Agentify: set up agent scaffolding" --body "<description>"` (or `gh pr edit` to update the description if pushing more commits onto an already-open PR from a prior run).

## Required PR description structure

```markdown
## Summary

One paragraph: what this PR sets up and why (agent-readiness pass for Claude Code + Cursor).

## Stages

- **Tool setup** — <what was installed/configured, or "already satisfied">
- **Harness scaffolding** — <what was added for Claude Code / Cursor, or "already satisfied">
- **Skill wiring** — <skills installed via `find-skills` + `skills` CLI, or "no relevant skills discoverable">
- **Permission hygiene** — <safe read-only entries added, or "already satisfied", or "not applicable for <harness>">
- **MCP + repomix setup** — <what was configured>
- **Custom-skill drift fixes** — <mechanical fixes applied, or "no custom/local skills found" / "all compliant">
- **Architecture docs** — <docs generated/updated, or "already current">

## Manual follow-ups

<Anything that couldn't be automated — e.g. "Global/cross-repo registration of the codebase-memory MCP server wasn't performed automatically; if you want cross-repo indexing, run: <commands read from the codebase-memory MCP skill at runtime>." Omit this section entirely if there's nothing to follow up on.>

## Judgment-based recommendations

<Custom-skill validation issues that weren't auto-fixed because they require judgment (vague description, non-actionable steps) — named per skill, not rewritten. Omit if none.>
```

Every stage gets a line, even a satisfied/no-op one — the PR description is the complete report of what the run found and did, not just a diff of what changed.

## No-op runs

If the detection map shows every pillar already satisfied (nothing to commit anywhere), still consider whether to open a PR at all: an empty PR with a description saying "no changes needed" is acceptable but not required — reporting "everything already agent-ready, no PR opened" directly to the user is equally valid and avoids PR noise. Either way, no branch, no commit, and no PR ever gets created with actual file changes when there's nothing to change.
