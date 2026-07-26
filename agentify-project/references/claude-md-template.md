# Agent instructions template and update procedure

Covers drafting content for any target format — `CLAUDE.md` / `AGENTS.md` (Claude Code) or Cursor project rules — from the same underlying draft, plus the merge procedure for updating an existing file. Read this before drafting or updating agent instructions for any in-scope harness. All writes here are committed on the run's dedicated branch as part of the harness-scaffolding stage — see [pr-delivery.md](pr-delivery.md) — never written directly to the branch the invoking session started on, and never gated on a mid-run confirmation.

## Sections to draft

Fill each section from what's actually discoverable in the repo — manifests, config files, existing docs, CI config. Never invent a command, path, or convention that isn't grounded in something present in the repo. The section content is the same regardless of target harness; only the surrounding file format differs (see "Per-harness format" below).

```markdown
# <Project name>

<1-3 sentence overview: what this project is, inferred from README/package manifest/repo name.>

## Commands

- Build: `<command>`
- Test: `<command>`
- Lint/format: `<command>`

<Only include commands you found evidence for — a package.json script, a Makefile target,
a CI workflow step. Omit a row rather than guessing.>

## Code conventions

<Notable conventions actually observed: language/framework, formatting tool config present
(e.g. .prettierrc, .eslintrc), test framework in use, directory naming patterns.>

## Directory map

<Top-level directories with a one-line purpose each, inferred from names and contents —
skip if the repo is flat or trivially small.>

## Tooling

<Skills wired in during this pass (name + one-line purpose), and any advisory MCP server
notes from Pillar 4 of the checklist. Omit this section entirely if neither applies.>
```

Keep the generated content proportional to the repo's actual size and complexity — a small repo gets a short file, not padded sections.

## Per-harness format

- **Claude Code, no existing pointer/log split** — write the drafted content as `CLAUDE.md` at the repo root (or sub-project root, for a monorepo scoped that way). Don't create `AGENTS.md` preemptively for a small/new repo that has no signal it wants the split — one reasonably-sized `CLAUDE.md` is the right output until the repo grows.
- **Claude Code, existing pointer/log split** — if the repo already has both `CLAUDE.md` (short, stable pointer) and `AGENTS.md` (living, append-only log of learned conventions), preserve that split: merge pointer-level updates (new top-line commands, a changed directory map) into `CLAUDE.md`, and append newly-learned conventions/decisions to `AGENTS.md`. Never collapse the two back into one file or merge `AGENTS.md`'s fuller content back into `CLAUDE.md`.
- **Cursor** — write the drafted content as a Cursor project rule. If the repo already uses the legacy single `.cursorrules` file, update that file in place with the drafted content. Otherwise, write it under `.cursor/rules/` as a `.mdc` file with frontmatter (`description`, and `alwaysApply: true` so it's always in context, since this content is general project context rather than a narrowly-scoped rule).
- **Both harnesses in scope** — draft the content once, then write it into each harness's file/location. If the two already exist and have diverged, reconcile them as part of the merge (see below) rather than silently picking one as authoritative.

## Merge procedure for an existing file

1. Draft the candidate content using the template above, grounded in current repo state (including the stack walk in [agent-readiness-checklist.md](agent-readiness-checklist.md)).
2. Read the existing instructions file(s) for each in-scope harness.
3. Compute a section-level merge (which sections are new, changed, or newly stale) rather than a raw line diff when the existing file's structure differs from the template, so the result reads coherently rather than as patchwork.
4. Write the merged result directly, as part of the harness-scaffolding commit — no mid-run confirmation step; the pull request itself is where a human reviews the change (see [pr-delivery.md](pr-delivery.md)).
5. Note briefly in the PR description *why* each substantive change was made (e.g. "Test command updated from `jest` to `vitest` — found `vitest.config.ts`, no `jest.config.*`"), so the reviewer can judge intent without re-deriving it from the diff alone.

Never regenerate and overwrite the whole file just because *some* section is stale — preserve sections that are clearly hand-authored and not contradicted by current repo state (e.g. a "Why this project exists" section with context no manifest could supply). This applies equally to `CLAUDE.md`, `AGENTS.md`, and Cursor project rules. The only case where a file is wholesale-replaced is when the user's own request explicitly asks for that specific file to be regenerated from scratch.
