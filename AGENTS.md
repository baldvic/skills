# Agent instructions for this repo

Instructions for any AI coding agent (Claude Code, Cursor, or otherwise) working in this repository.

## What this repo is

This repo publishes reusable **Agent Skills** — procedural knowledge (`SKILL.md` plus optional `references/`, `scripts/`, `evals/`) that other coding agents install and use. It's published publicly on skills.sh (see `README.md`), which discovers skills purely by crawling public GitHub repos that contain `skills.sh.json` + `SKILL.md` files — there is no separate "publish" or "register" step. Do not invent one.

## Two categories of skill directory — don't confuse them

- **Top-level, distributed** (`skill-standard/`, `agentify-project/`): the actual product. Each has its own `SKILL.md`, is listed in `skills.sh.json`'s `groupings`, and is documented in `README.md`'s "Skills" table and "Repository layout" tree. Anyone can `npx skills add baldvic/skills@<name>` these.
- **Repo-internal, not distributed** (`.claude/skills/`, `.cursor/skills/`): meta-skills that help maintain *this* repo itself (the `openspec-*` skills powering `/opsx:*`, `skills-catalog-housekeeping`). These are never added to `skills.sh.json` or `README.md`'s Skills table — they're tooling, not product. Note the `openspec-*` meta-skills currently exist only under `.claude/skills/` (Claude Code), not mirrored to `.cursor/skills/` — if you're operating as a Cursor agent, the `/opsx:*` workflow isn't available there yet.

When adding, renaming, or removing a top-level distributed skill, run the **skills-catalog-housekeeping** skill to keep `skills.sh.json` and `README.md` in sync. It exists in two byte-identical copies (`.claude/skills/skills-catalog-housekeeping/` and `.cursor/skills/skills-catalog-housekeeping/`) — if you edit one, mirror the edit into the other.

Known, currently-unresolved exception to "byte-identical": `.claude/skills/skills-catalog-housekeeping/evals/` exists but has no `.cursor/` counterpart. `SKILL.md` and `references/` do match. This is a real, standing divergence, not a one-off — expect the self-consistency check to keep flagging it until someone deliberately decides to add the missing `evals/` or leave it Claude-only on purpose.

## Public repo: never leak machine-specific data

This repo is crawled publicly by skills.sh and cloned by strangers. Before committing or pushing anything:

- No absolute local paths (`C:\Users\...`, `D:\projects\...`, `/home/<user>/...`) in any file.
- No real usernames, personal/company emails, or hostnames.
- No names of private/internal repos or systems used as illustrative examples — genericize them (e.g. "a reference repo" instead of naming one).
- If something like this is already pushed, redacting the working tree isn't enough — it's still in git history. Don't rewrite history (rebase/filter-branch/filter-repo/force-push) to scrub it without the user explicitly asking; flag it instead, and never force-push to `main`/`master` even if asked — warn instead and let the user run that push themselves.

`skills-catalog-housekeeping`'s machine-specific-data-scan step does exactly this check on demand — use it before pushing content that references real-world examples.

## Change workflow: OpenSpec

Non-trivial changes go through `openspec/`: propose → design/tasks → implement → sync specs → archive. Use the `opsx:*` slash commands / `openspec-*` skills for this rather than editing `openspec/specs/` by hand. `openspec/changes/archive/` holds completed changes for historical reference — treat them as a record of past decisions, not a live source of truth (specs under `openspec/specs/` are current; archived proposals/designs describe the state of the world *when written*, which may have since evolved).

## Mechanical vs. judgment-based fixes

Skills in this repo (`agentify-project`'s custom-skill validation, `skills-catalog-housekeeping`'s drift checks) follow one shared convention, worth reusing in any similar drift-detection work added here: **mechanical** fixes (deterministic, no guessing anyone's intent — a missing catalog entry, a legacy schema field, a leak with an obvious generic replacement) are applied directly and reported as done. **Judgment-based** findings (a stale-looking reference that might be an in-progress rename, a description that merely reads as stale, anything whose remediation would mean rewriting shared git history) are always reported for a human decision, never auto-applied. When something doesn't clearly fit either bucket, default to judgment-based.

## No build/test tooling

This repo has no `package.json`, build step, or CI test suite — it only distributes skill definitions (markdown plus optional scripts/evals). Individual skills may carry their own `evals/evals.json` (see `skill-standard/references/eval-artifacts.md`) for self-validation; there's no repo-wide test command to run.

## Keeping this file current

Unlike the short-pointer-vs-living-log split `agentify-project` sets up for *other* repos' `CLAUDE.md`/`AGENTS.md`, this repo has only this one file — there's no separate `CLAUDE.md` to keep terse while this one grows. If something here goes stale (a new top-level skill category, a changed workflow), update this file directly as part of that change rather than leaving it to drift; it isn't synced by any skill or check.
