# skills

[![skills.sh](https://skills.sh/b/baldvic/skills)](https://www.skills.sh/baldvic/skills)

Portable Agent Skills library. Install from [skills.sh](https://www.skills.sh/baldvic/skills/skill-standard):

```bash
npx skills add baldvic/skills@skill-standard -y
```

## skill-standard

Meta-skill for authoring skills (open Agent Skills format) with eval and benchmark artifacts.

- **Runtime:** fully self-contained under `skill-standard/` (no external repos required to author skills).
- **Maintenance:** [references/upstream-sources.md](skill-standard/references/upstream-sources.md) lists [agentskills/agentskills](https://github.com/agentskills/agentskills) and [anthropics/skills](https://github.com/anthropics/skills) (`skill-creator`); [references/upstream-sync.md](skill-standard/references/upstream-sync.md) is the agent pipeline; [upstream.lock.json](skill-standard/upstream.lock.json) pins last merged commits.

Copy or install into your agent client's skills directory.

## Publishing on skills.sh

There is no separate registration. A public GitHub repo with Agent Skills folders (`<name>/SKILL.md`) is installable via `npx skills add baldvic/skills` and gets a repo page at [skills.sh/baldvic/skills](https://www.skills.sh/baldvic/skills).

- **Leaderboard / install counts:** anonymous telemetry from the [skills CLI](https://www.skills.sh/docs/cli) when users install (opt out with `DISABLE_TELEMETRY=1`).
- **Repo page layout:** optional [skills.sh.json](skills.sh.json) at the repo root ([customize docs](https://www.skills.sh/docs/customize)).
- **Badge:** `[![skills.sh](https://skills.sh/b/owner/repo)](https://www.skills.sh/owner/repo)` — see [docs overview](https://www.skills.sh/docs).
