# skills

[![skills.sh](https://skills.sh/b/baldvic/skills)](https://skills.sh/baldvic/skills)

Portable Agent Skills library. Install from [skills.sh](https://skills.sh/baldvic/skills/skill-standard):

```bash
npx skills add baldvic/skills@skill-standard -y
```

## skill-standard

Meta-skill for authoring skills (open Agent Skills format) with eval and benchmark artifacts.

- **Runtime:** fully self-contained under `skill-standard/` (no external repos required to author skills).
- **Maintenance:** [references/upstream-sources.md](skill-standard/references/upstream-sources.md) lists [agentskills/agentskills](https://github.com/agentskills/agentskills) and [anthropics/skills](https://github.com/anthropics/skills) (`skill-creator`); [references/upstream-sync.md](skill-standard/references/upstream-sync.md) is the agent pipeline; [upstream.lock.json](skill-standard/upstream.lock.json) pins last merged commits.

Copy or install into your agent client's skills directory.
