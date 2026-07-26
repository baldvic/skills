# Upstream sources

skill-standard is **self-contained at runtime**. The repos below are **maintenance inputs** only — used when syncing editorial changes into this bundle. They are not loaded during normal skill authoring.

## Primary sources

| ID | Repository | Role in skill-standard |
|----|------------|-------------------------|
| `agentskills` | [agentskills/agentskills](https://github.com/agentskills/agentskills) | Open **Agent Skills** format: specification, skill-creation guides, validation semantics |
| `anthropics-skill-creator` | [anthropics/skills](https://github.com/anthropics/skills) (`skills/skill-creator/`) | **Eval / benchmark** JSON shapes, grader/analyzer playbooks, aggregation script |

Install reference copy via Skills CLI (optional):

```bash
npx skills add anthropics/skills@skill-creator -y
```

Browse format docs: [agentskills.io](https://agentskills.io)

## File lineage

| Local file | Derived from |
|------------|----------------|
| `references/agent-skills-format.md` | `agentskills` → `docs/specification.mdx`, `docs/skill-creation/best-practices.mdx` |
| `references/validation.md` | `agentskills` spec + `skills-ref` test expectations |
| `references/eval-artifacts.md` | `agentskills` → `docs/skill-creation/evaluating-skills.mdx` + `anthropics` → `skills/skill-creator/references/schemas.md` |
| `references/description-tuning.md` | `agentskills` → `docs/skill-creation/optimizing-descriptions.mdx` |
| `agents/grader.md` | `anthropics` → `skills/skill-creator/agents/grader.md` (harness-neutral edit) |
| `agents/analyzer.md` | `anthropics` → `skills/skill-creator/agents/analyzer.md` + benchmark notes patterns |
| `scripts/aggregate_benchmark.py` | `anthropics` → `skills/skill-creator/scripts/aggregate_benchmark.py` |
| `references/upstream-sync.md` | skill-standard original (sync pipeline) |
| `upstream.lock.json` | Last merged upstream commits + mapping metadata |

## Editorial layer (do not overwrite from upstream)

When syncing, **preserve** unless upstream explicitly improves portable behavior:

- Harness-agnostic rules in `SKILL.md`
- Removal of product-specific CLIs, subagents, HTML viewers, and absolute paths
- Wording that skills must stay portable (relative paths, no secrets)
- `local_only` entries listed in `upstream.lock.json`

## Package registry mirror

Published skill packages (for diffing installed copies):

| Package | URL |
|---------|-----|
| `anthropics/skills@skill-creator` | [skills.sh](https://skills.sh/anthropics/skills/skill-creator) |
| `mattpocock/skills@writing-great-skills` | Optional quality reference (not merged by default) — [skills.sh](https://skills.sh/mattpocock/skills/writing-great-skills) |

`writing-great-skills` is a **optional** cross-check for description/pruning quality; it is not part of the default sync mapping.

## Lock file

`upstream.lock.json` records `resolved_commit` per source after each successful sync. Compare upstream `git log` since that commit to scope work.

Pipeline: [upstream-sync.md](upstream-sync.md).
