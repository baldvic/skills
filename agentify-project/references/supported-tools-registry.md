# Extensible supported-tools registry

The tool-setup stage is driven by this registry, not bespoke per-tool logic. Read this before running the tool-setup stage of the pipeline.

## Entry shape

Every tool in the registry declares four things:

| Field | Meaning |
|---|---|
| `detect` | How to check the tool is already set up. Read-only, feeds the detection map. |
| `install` | How to set it up if `detect` finds it missing. |
| `scope` | `repo` — output is a committed file or dependency, goes in the PR diff — or `machine-global` — a local prerequisite for running the pipeline at all, never a PR deliverable. |
| `idempotency` | How a re-run confirms the tool is already done, so nothing is reinstalled or reconfigured needlessly. |

The tool-setup stage iterates this table generically: for each entry, run `detect`; if missing, run `install`; if `scope` is `repo`, the result is included in that stage's commit; if `scope` is `machine-global`, it's performed as a local step before the pipeline continues, and never appears in the PR diff.

## Initial five entries

### OpenSpec CLI

- **detect**: `openspec --version` succeeds.
- **install**: `npm install -g openspec`.
- **scope**: the CLI install itself is `machine-global` (a local prerequisite, not a PR deliverable). Its repo-facing output — running `openspec init` — is `scope: repo`, and only runs if the detection map shows no `openspec/` directory yet.
- **idempotency**: skip the global install if `openspec --version` already succeeds; skip `openspec init` if `openspec/` already exists.

### `skills` CLI

- **detect**: `npx skills --version` (or any subcommand) resolves without an install prompt failing.
- **install**: for a Node project, add `skills` as a devDependency (`npm install -D skills`), matching how real repos pin it in `package.json`. The CLI itself is invoked via `npx` rather than installed globally.
- **scope**: `repo` — the devDependency entry in `package.json`/lockfile.
- **idempotency**: skip the devDependency add if `skills` is already listed in `package.json`.

### repomix

- **detect**: a `repomix.config.json` (or equivalent config) exists at the repo root, **or** the `repomix-explorer` skill is already installed under `.agents/skills/` per `skills-lock.json`.
- **install**: discover the corresponding skill via `find-skills` and, if not already present, install it non-interactively: `npx skills add yamadashy/repomix@repomix-explorer -y`. Then follow that skill's own instructions to set repomix up in this repo. Agentify-project does **not** hardcode a devDependency-add or author a config file itself — see the fallback note below for what to do if that skill's instructions don't produce one.
- **scope**: `repo` — the skill's presence in `.agents/skills/` + `skills-lock.json` (mirrored into `.claude/skills/`/`.cursor/skills/` by the skill-wiring pillar), plus whatever config file its instructions produce.
- **idempotency**: skip the skill install if `repomix-explorer` is already listed in `skills-lock.json`; skip any further setup step its instructions describe if their own idempotency check says it's already done.

**Fallback/descriptive context — known-good `repomix.config.json` shape:** agentify-project no longer authors this file directly; whatever the installed `repomix-explorer` skill's own instructions produce (or don't) is authoritative. The shape below is kept only as descriptive context, useful if that skill's instructions leave repomix uninstalled at the CLI level and something needs to sanity-check what a reasonable config looks like — it is not a template agentify-project itself writes:

```json
{
  "output": {
    "filePath": "repomix-output.xml",
    "style": "xml",
    "showLineNumbers": true,
    "topFilesLength": 20,
    "showDirectoryStructure": true
  },
  "include": [],
  "ignore": {
    "useGitignore": true,
    "useDefaultPatterns": true,
    "customPatterns": [".agents/skills/**", ".claude/skills/**", ".cursor/skills/**"]
  },
  "security": {
    "enableSecurityCheck": true
  },
  "compression": {
    "enabled": true
  }
}
```

Key properties, for reference only: XML output, directory structure and summary on, compression on, gitignore-aware, security check on, and installed-skill trees excluded from the packed output (they're vendored dependencies, not project source).

### codebase-memory MCP

- **detect**: a matching server entry in project-scoped `.mcp.json` (Claude Code) or `.cursor/mcp.json` (Cursor), **or** the `codebase-memory-mcp-intelligence` skill is already installed under `.agents/skills/` per `skills-lock.json`.
- **install**: discover the corresponding skill via `find-skills` and, if not already present, install it non-interactively: `npx skills add aradotso/mcp-skills@codebase-memory-mcp-intelligence -y`. Then follow that skill's own instructions to register the MCP server — **project-scoped only, always**. Agentify-project does not hardcode or duplicate that skill's install/registration commands; its content is the live source of truth, since that surface can drift independently of agentify-project.
- **scope**: `repo` — the skill's presence in `.agents/skills/` + `skills-lock.json`, plus the project-scoped MCP registration its instructions produce. See the global/cross-repo rule below.
- **idempotency**: skip the skill install if already listed in `skills-lock.json`; skip registration if `detect` already finds a matching MCP entry.

### Serena MCP

- **detect**: a matching Serena entry in project-scoped `.mcp.json` (Claude Code) or `.cursor/mcp.json` (Cursor), **or** the `setup-serena-mcp` skill is already installed under `.agents/skills/` per `skills-lock.json`.
- **install**: discover the corresponding skill via `find-skills` and, if not already present, install it non-interactively: `npx skills add neolabhq/context-engineering-kit@setup-serena-mcp -y`. Then follow that skill's own instructions to register the MCP server — **project-scoped only, always**, exactly like the codebase-memory MCP entry above.
- **scope**: `repo` — the skill's presence in `.agents/skills/` + `skills-lock.json`, plus the project-scoped MCP registration its instructions produce. See the global/cross-repo rule below.
- **idempotency**: skip the skill install if already listed in `skills-lock.json`; skip registration if `detect` already finds a matching MCP entry.

## Adding a sixth tool

Adding a new supported tool (a linter, Playwright, whatever) means adding one more entry in this exact shape — `detect`, `install`, `scope`, `idempotency` — to this table. The tool-setup stage's control flow doesn't change: it already iterates the registry generically. No pipeline logic needs editing for a new entry to take effect. If the new tool is itself an MCP server or has a matching setup skill, follow the same "discover via `find-skills`, install via the `skills` CLI, then follow that skill's own instructions" pattern used by the repomix, codebase-memory MCP, and Serena MCP entries above, rather than hardcoding its install commands.

## Global/cross-repo MCP registration is never automatic

This rule doesn't have exceptions: no MCP entry in this registry — codebase-memory, Serena, or any added later — performs global or cross-repo MCP server registration automatically, no matter how useful that would be for the user's actual workflow. A pull request can only represent repo-committed state; a global/user-home registration lives outside the repo and can't be reviewed or merged as part of one. When cross-repo indexing is genuinely what the user wants, the PR description's "Manual follow-ups" section (see [pr-delivery.md](pr-delivery.md)) documents the manual global-registration steps — read from the same installed skill used for the project-scoped install — as something the owner can choose to run themselves, outside the PR.
