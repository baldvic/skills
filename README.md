# skills

[![skills.sh](https://skills.sh/b/baldvic/skills)](https://www.skills.sh/baldvic/skills)

Reusable **Agent Skills** for AI coding agents — procedural knowledge (workflows, checklists, format rules) that agents load when your task matches the skill’s `description`. Skills follow the open [Agent Skills](https://github.com/agentskills/agentskills) layout: each skill is a folder with a `SKILL.md` file plus optional `references/`, `scripts/`, and `assets/`.

This repository is published on [skills.sh](https://www.skills.sh/baldvic/skills) and installable with the [skills CLI](https://www.skills.sh/docs/cli) (Cursor, Claude Code, Codex, and [other supported agents](https://www.skills.sh/docs)).

## Install

Install one skill:

```bash
npx skills add baldvic/skills@skill-standard -y
```

Install every skill in this repo:

```bash
npx skills add baldvic/skills -y
```

**Manual install:** copy a skill folder (for example `skill-standard/`) into your agent’s skills directory (e.g. `.cursor/skills/`, `.claude/skills/`, or the path your client documents). Keep the folder name identical to the skill’s `name` in frontmatter.

Review skill content before use; installed skills run with the same permissions as your agent session.

## Skills

| Skill | Purpose |
|-------|---------|
| [**skill-standard**](skill-standard/SKILL.md) | Author, validate, and improve portable skills — including evals and benchmarks to show a skill actually helps. |

### skill-standard — when to use it

Use **skill-standard** when you (or your agent) are:

- Creating a new skill from a conversation or repo convention
- Editing `SKILL.md` frontmatter or tightening activation triggers
- Validating structure before sharing a skill
- Running **with_skill / without_skill** evals and grading runs
- Tuning descriptions so the right skill activates

The skill is **self-contained**: format spec, validation checklist, eval JSON schemas, grader/analyzer procedures, and an optional benchmark aggregation script all live under `skill-standard/`. You do not need to clone Anthropic’s or other upstream repos to follow the workflow day to day.

**High-level workflow:** capture intent → draft `skill-name/SKILL.md` → validate → add `evals/evals.json` → run eval matrix → grade → iterate description and content → (optional) aggregate benchmarks.

**Start reading inside the skill:**

| Topic | File |
|-------|------|
| Full format spec | [skill-standard/references/agent-skills-format.md](skill-standard/references/agent-skills-format.md) |
| Pre-publish checklist | [skill-standard/references/validation.md](skill-standard/references/validation.md) |
| Evals & benchmark artifacts | [skill-standard/references/eval-artifacts.md](skill-standard/references/eval-artifacts.md) |
| Grading a run | [skill-standard/agents/grader.md](skill-standard/agents/grader.md) |

**Maintenance:** [upstream-sources.md](skill-standard/references/upstream-sources.md) documents lineage from [agentskills/agentskills](https://github.com/agentskills/agentskills) and [anthropics/skills](https://github.com/anthropics/skills) (`skill-creator`); [upstream-sync.md](skill-standard/references/upstream-sync.md) describes refreshing this meta-skill; [upstream.lock.json](skill-standard/upstream.lock.json) pins last merged upstream commits.

## Repository layout

```
skills/
├── skills.sh.json          # Optional grouping for the skills.sh repo page
├── skill-standard/
│   ├── SKILL.md            # Entry point (read this first)
│   ├── references/         # Format, validation, eval schemas
│   ├── agents/             # Grader & analyzer roles
│   └── scripts/            # Optional tooling (e.g. benchmark aggregation)
└── LICENSE
```

Future skills will appear as sibling directories (`another-skill/SKILL.md`). [skills.sh.json](skills.sh.json) controls how skills are grouped on [the directory page](https://www.skills.sh/baldvic/skills).

## License

MIT — see [LICENSE](LICENSE). Individual skills may repeat the license in `SKILL.md` frontmatter.
