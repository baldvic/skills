## Purpose

Defines the behavior of the `agentify-project` skill: a non-interactive, staged pipeline that makes a codebase ready for AI coding agents across Claude Code and Cursor by default — agent instructions, skill wiring via `find-skills` and the `skills` CLI, harness-appropriate permission hygiene, real setup for a small set of specifically-supported tools (repomix, the codebase-memory MCP server, the Serena MCP server), advisory-only notes for everything else, and architecture documentation — delivered as commits on a dedicated branch and opened as a single pull request for the repo owner to review, rather than gated by mid-run confirmation prompts.

## Requirements

### Requirement: Agent-readiness audit
The `agentify-project` skill SHALL run a read-only detection pass over the target codebase — after the pre-flight prerequisite gate has passed — building a single shared map of what agent-related scaffolding already exists: agent instructions files, `.claude/`/`.cursor/`/`.agents/` contents (including which installed skills are registry-tracked in `skills-lock.json` versus custom/local), `openspec/` state, `docs/` state, repomix configs, MCP registration, and detected tech stack. Every later stage SHALL read from this one map rather than re-detecting the same facts. The final pull request SHALL include a summary of what was found, what was changed, and what was left as-is; the skill SHALL NOT pause mid-run to ask the user to review findings before proceeding — see the staged pipeline requirement for how results are delivered instead.

#### Scenario: Fresh repo with no agent scaffolding
- **WHEN** the skill is run against a repository that has no agent instructions file, no wired skills, no `openspec/`, and no `docs/`
- **THEN** the detection map records every pillar as a gap, and the skill proceeds through the staged pipeline to fill them in without pausing to ask

#### Scenario: Already agent-ready repo
- **WHEN** the skill is run against a repository that already has current agent instructions, relevant skills wired, tools installed, and current architecture docs
- **THEN** the detection map records each pillar as satisfied, and the run opens no PR (or a PR whose description states no changes were needed) — see idempotent re-runs

### Requirement: Harness detection determines merge targets, not whether to act
The skill SHALL scaffold **both** Claude Code and Cursor by default on every run — a fixed set of exactly two harnesses, not an open-ended list — regardless of which harness-specific directories (`.claude/`, `.cursor/`) currently exist, unless the user's own request explicitly scopes the run to a single harness. Detection of existing `.claude/`, `.cursor/`, and `.agents/` content SHALL still occur (as part of the detection map) to determine what to merge into for each harness, but SHALL NOT be used to decide whether a harness is set up at all.

#### Scenario: Neither harness present
- **WHEN** the target repo has neither a `.claude/` nor a `.cursor/` directory, and the user's request doesn't scope to one harness
- **THEN** the skill creates scaffolding for both Claude Code and Cursor without asking which to set up

#### Scenario: Only one harness present
- **WHEN** the target repo has a `.claude/` directory but no `.cursor/` directory, and the user's request doesn't scope to one harness
- **THEN** the skill adds Cursor scaffolding alongside the existing Claude Code scaffolding, merging into the existing `.claude/` content rather than replacing it

#### Scenario: User request scopes to one harness
- **WHEN** the user's request explicitly says to set up only Claude Code (or only Cursor)
- **THEN** the skill scaffolds only the requested harness and does not create the other

### Requirement: Agent instructions generation and update supersedes `init`
The skill SHALL generate or merge the agent instructions surface for every in-scope harness itself, without delegating to the built-in `init` skill: `CLAUDE.md` and `AGENTS.md` for Claude Code (which SHALL NOT be forced into byte-identical content — `CLAUDE.md` may remain a short, stable pointer while `AGENTS.md` carries fuller, evolving detail), and Cursor project rules (`.cursor/rules/*.mdc`, or the existing `.cursorrules` if that convention is already in use) for Cursor. Every write to an existing file SHALL be a merge that preserves hand-authored content and adds/updates only what's missing or demonstrably stale; the skill SHALL NOT wholesale-replace an existing instructions file unless the user's own request explicitly asks for that specific file to be regenerated from scratch. These writes happen on the dedicated branch created for the run's pull request, as their own commit — never directly on the branch the invoking session started on.

#### Scenario: No existing instructions file for an in-scope harness
- **WHEN** the target repo has no `CLAUDE.md` and Claude Code is in scope
- **THEN** the skill drafts one from its template and commits it directly on the run's dedicated branch, since there is no existing content to merge with

#### Scenario: Existing instructions file present
- **WHEN** the target repo already has a `CLAUDE.md`, `AGENTS.md`, or Cursor rule file for an in-scope harness
- **THEN** the skill merges its proposed updates into the existing content — preserving sections not affected by the merge — and commits the result on the dedicated branch, without pausing for confirmation

#### Scenario: User explicitly requests a from-scratch regeneration
- **WHEN** the user's request explicitly asks to regenerate a specific instructions file from scratch
- **THEN** the skill replaces that specific file's content wholesale, while still leaving every other file's merge behavior unchanged

#### Scenario: CLAUDE.md and AGENTS.md serve different purposes
- **WHEN** the target repo's `CLAUDE.md` is a short, stable pointer and `AGENTS.md` is a living log of learned conventions
- **THEN** the skill preserves that split rather than merging `AGENTS.md`'s fuller content back into `CLAUDE.md` or vice versa

### Requirement: Skill wiring uses `find-skills` and the `skills` CLI
The skill SHALL analyze the target repo's tech stack, use the `find-skills` skill (or an equivalent skills.sh search mechanism) to identify relevant, discoverable skills, and for each relevant match not already installed, run the `skills` CLI (`npx skills add <source>@<skill> -y`) directly and non-interactively to install it — relying on the CLI's own management of the canonical `.agents/skills/` store, `skills-lock.json`, and per-harness mirrors (`.claude/skills/`, `.cursor/skills/`) rather than manually copying skill folders. It SHALL NOT fabricate or invent a skill that `find-skills` or the registry doesn't actually surface. Skills installed this way SHALL be treated as trusted by construction and SHALL NOT be subject to the custom/local skill validation requirement below.

#### Scenario: Relevant skills found
- **WHEN** the target repo's stack matches one or more skills surfaced by `find-skills` that aren't yet present in `.agents/skills/`
- **THEN** the skill installs each one via `npx skills add <source>@<skill> -y` as part of the skill-wiring commit, without pausing to ask, and the CLI's own lockfile/mirroring behavior is left untouched by any custom copy logic

#### Scenario: No relevant skills discoverable
- **WHEN** `find-skills` surfaces no skill matching the detected stack
- **THEN** the skill reports "no skill recommendations found" in the PR description as a valid outcome rather than installing an unrelated skill

#### Scenario: Skill already installed
- **WHEN** a recommended skill is already present under `.agents/skills/` per the detection map
- **THEN** the skill does not reinstall or duplicate it

### Requirement: Tool-permission hygiene is scoped to harnesses with an established mechanism
For a harness with an established, documented, repo-committed permission mechanism — currently Claude Code's `.claude/settings.json` `permissions.allow` — the skill SHALL merge in a small, well-known safe read-only command allowlist automatically when entries are missing, committed as part of the permission-hygiene stage. Any change that would broaden an existing permission, add a non-read-only command, or touch `permissions.deny`/hook configuration SHALL NOT be applied unless the user's own request explicitly asks for that specific broadening; absent that, the skill SHALL report the broader change as a recommendation in the PR description rather than applying it. For a harness without an established repo-committed permission mechanism (Cursor, as of this writing), the skill SHALL report that this pillar has no applicable action for that harness rather than fabricating a config format for it.

#### Scenario: Missing safe read-only entries for Claude Code
- **WHEN** `.claude/settings.json` (or `.claude/settings.local.json`, whichever convention the repo already uses) is missing common safe read-only allowlist entries and Claude Code is in scope
- **THEN** the skill adds them automatically as part of the permission-hygiene commit, without pausing

#### Scenario: A broader change would otherwise be useful
- **WHEN** the detection map suggests a permission change that would broaden access (e.g. a new non-read-only command the repo's own CI already runs)
- **THEN** the skill reports it as a recommendation in the PR description and does not apply it unless the user's request explicitly asked for that broadening

#### Scenario: In-scope harness has no established permission mechanism
- **WHEN** Cursor is the harness in scope and has no documented repo-committed permission convention
- **THEN** the skill reports this pillar as not applicable for Cursor and writes no permission file for it

### Requirement: MCP setup — specific supported tools get real installs via find-skills, everything else stays advisory
The skill SHALL perform real, non-interactive setup for exactly three specifically-supported tools — the codebase-memory MCP server, the Serena MCP server, and repomix — as entries in the supported-tools registry (see the extensible supported-tools registry requirement). For each, the skill SHALL discover the tool's corresponding skill using `find-skills`, install it via the `skills` CLI (`npx skills add <source>@<skill> -y`) if not already present under `.agents/skills/`, and follow that installed skill's own instructions to perform the tool's setup — rather than agentify-project hardcoding or duplicating that tool's install/registration commands itself. Registration of the codebase-memory MCP server and the Serena MCP server SHALL be **project-scoped only** (`.mcp.json` for Claude Code, `.cursor/mcp.json` for Cursor); the skill SHALL NOT perform global or cross-repo MCP registration automatically for either server under any circumstance, since that state lives outside the repo and can't be represented in a pull request — where cross-repo/global registration is what the user actually wants, the skill SHALL document the manual steps for it in the PR description instead of performing them. For any other MCP server the stack analysis suggests might help, the skill SHALL continue to note it as advisory text only (name and rationale) and SHALL NOT generate configuration, endpoints, or credentials for it.

#### Scenario: codebase-memory MCP not yet set up
- **WHEN** the detection map shows no codebase-memory MCP registration and no `codebase-memory-mcp-intelligence` skill installed for the target repo
- **THEN** the skill installs the `codebase-memory-mcp-intelligence` skill via `find-skills` + the `skills` CLI and follows its instructions to register the server via project-scoped MCP configuration only, committing the result as part of the MCP/repomix-setup stage, without pausing to ask

#### Scenario: Serena MCP not yet set up
- **WHEN** the detection map shows no Serena MCP registration and no `setup-serena-mcp` skill installed for the target repo
- **THEN** the skill installs the `setup-serena-mcp` skill via `find-skills` + the `skills` CLI and follows its instructions to register the server via project-scoped MCP configuration only, committing the result as part of the MCP/repomix-setup stage, without pausing to ask

#### Scenario: User wants cross-repo/global registration
- **WHEN** the target repo's intended usage is cross-repo indexing (the kind the reference repo itself uses, registered globally) for either the codebase-memory or Serena MCP server
- **THEN** the skill still only applies project-scoped registration automatically, and documents the manual global-registration steps in the PR description as a follow-up the owner can choose to do themselves

#### Scenario: repomix not yet set up
- **WHEN** the detection map shows no `repomix.config.json` (or equivalent) and no `repomix-explorer` skill installed in the target repo
- **THEN** the skill installs the `repomix-explorer` skill via `find-skills` + the `skills` CLI and follows its instructions for setting repomix up in this repo, merging around any existing partial config rather than overwriting it

#### Scenario: All three already set up
- **WHEN** the detection map shows codebase-memory MCP, Serena MCP, and repomix already configured (or their corresponding skills already installed)
- **THEN** the skill makes no changes for any of the three and reports all as satisfied

#### Scenario: An unrelated MCP server is suggested
- **WHEN** the detected stack suggests a different MCP server (e.g. for a database or ticketing system) would help
- **THEN** the skill writes only an advisory suggestion for it in the PR description, with no config, endpoint, or credential generated

### Requirement: Idempotent re-runs
Re-running the skill against a repository it has already agentified SHALL converge rather than duplicate content, re-run installs, or open a duplicate pull request — repeated runs on an unchanged repo SHALL propose no further changes and perform no further installs, across every pillar including the tool-setup and architecture-documentation pillars added by this revision. If a prior run's pull request is still open, a subsequent run SHALL add further commits to that same branch/PR rather than opening a second one.

#### Scenario: Second run on an already-agentified repo, no open PR
- **WHEN** the skill is run a second time against a repo it previously agentified (and that prior PR was already merged or closed) with no changes to the repo in between
- **THEN** the detection map shows every pillar (including tool setup and docs) already satisfied, and the skill opens no new PR (or opens one whose description states no changes were needed)

#### Scenario: A prior run's PR is still open
- **WHEN** the skill is run again while a previous agentify run's pull request is still open and unmerged
- **THEN** the skill pushes any new commits to the existing branch/PR rather than creating a second, competing pull request

### Requirement: Uniform pre-flight prerequisite gate
Before creating a branch or making any change, the skill SHALL verify three prerequisites, treated as equally hard: the git working tree is clean (no uncommitted changes), the `gh` CLI is present, authenticated, and the repository has a pushable remote, and Node/npm is available. If any of these checks fails, the skill SHALL stop immediately, make no changes of any kind, and report which check(s) failed along with a concrete recommended fix for each. The skill SHALL NOT proceed with a partial or degraded run when a prerequisite is missing.

#### Scenario: Working tree not clean
- **WHEN** the target repo has uncommitted changes at the start of a run
- **THEN** the skill stops before creating a branch or touching any file, and reports that the working tree must be committed or stashed first

#### Scenario: `gh` CLI missing, unauthenticated, or no pushable remote
- **WHEN** the `gh` CLI isn't installed, isn't authenticated, or the repo has no remote it can push to
- **THEN** the skill stops and reports the specific failure (e.g. "install and authenticate the `gh` CLI: `gh auth login`", or "configure a pushable `origin` remote") with no changes made

#### Scenario: Node/npm unavailable
- **WHEN** Node/npm is not available on the machine running the skill
- **THEN** the skill stops and reports that Node/npm is required (needed for the `npx`-based tools in the supported-tools registry), with an installation recommendation, and makes no changes

#### Scenario: All prerequisites satisfied
- **WHEN** the working tree is clean, `gh` is present/authenticated/pushable, and Node/npm is available
- **THEN** the skill proceeds to create the dedicated branch and run the staged pipeline

### Requirement: Staged pipeline delivered as a single pull request
The skill SHALL execute as an explicit, ordered pipeline of stages — tool setup, repo/stack analysis, harness scaffolding, skill wiring, permission hygiene, MCP/repomix setup, architecture documentation — after the pre-flight prerequisite gate and detection map have completed, and SHALL NOT pause mid-run to request confirmation at any stage. All changes SHALL be made on a dedicated branch created for the run, with each applicable stage committed separately; the skill SHALL then push the branch and open a single pull request (via `gh pr create`) whose description summarizes every stage's outcome, including any manual follow-ups the owner still needs to perform. The skill SHALL NOT write any pipeline output directly to the branch the invoking session started on.

#### Scenario: Full run start to finish
- **WHEN** the skill is invoked against a target repo and the pre-flight gate passes
- **THEN** it creates a dedicated branch, proceeds through all applicable stages in order without requesting any confirmation, commits each stage separately, pushes the branch, and opens one pull request summarizing every stage

#### Scenario: A stage finds nothing to do
- **WHEN** a given stage's detection-map facts show it's already fully satisfied
- **THEN** the skill produces no commit for that stage, notes it as satisfied in the PR description, and continues to the next stage without pausing

### Requirement: Extensible supported-tools registry
The tool-setup stage SHALL be driven by a registry of supported tools, where each entry declares: how to detect the tool is already set up, how to install/configure it if missing, whether its output is repo-scoped (a committed file or dependency, included in the pull request) or machine-global (a local prerequisite for running the pipeline, never a pull-request deliverable), and how a re-run verifies it's already done. The registry SHALL initially contain five entries — the OpenSpec CLI, the `skills` CLI, repomix, the codebase-memory MCP server, and the Serena MCP server — and SHALL be structured so that adding a further supported tool means adding a new entry in the same shape, without requiring changes to the pipeline's control flow. For the repomix, codebase-memory MCP, and Serena MCP entries specifically, the `install` definition SHALL be "discover the tool's corresponding skill via `find-skills`, install it via the `skills` CLI, then follow that skill's own instructions" rather than a bespoke command sequence — see the MCP setup requirement below for the full detail.

#### Scenario: A registry tool is missing
- **WHEN** a tool in the registry is not yet set up per its `detect` check
- **THEN** the skill installs/configures it per its `install` definition, including its output in the pull request if `scope` is repo-committed, or performing it as a local prerequisite step (not part of the PR diff) if `scope` is machine-global

#### Scenario: A registry tool is already set up
- **WHEN** a tool's `detect` check finds it already configured
- **THEN** the skill makes no changes for that tool and reports it as satisfied

#### Scenario: A new tool is added to the registry
- **WHEN** an entry for a sixth (or later) supported tool is added to the registry, following the same detect/install/scope shape as the initial five
- **THEN** the tool-setup stage applies the same handling to it without requiring any change to the pipeline's stage logic

### Requirement: Custom/local skill validation and drift enforcement
During the detection map's inventory of `.claude/skills/`, `.cursor/skills/`, and `.agents/skills/`, the skill SHALL classify each existing skill as registry-sourced (tracked in `skills-lock.json`) or custom/local (not tracked). Registry-sourced skills SHALL NOT be re-validated. Each custom/local skill SHALL be checked against `skill-standard`'s validation checklist. Mechanical violations (frontmatter field errors, directory-name mismatch with the skill's `name`, disallowed top-level frontmatter keys, host-specific absolute paths, description length over the limit) SHALL be fixed directly and committed as their own pull-request commit. Judgment-based violations (vague or low-quality description, non-actionable steps) SHALL be reported in the pull request description as a recommendation and SHALL NOT be rewritten automatically.

#### Scenario: Custom skill with a mechanical violation
- **WHEN** a custom/local skill has a structural violation (e.g. its directory name doesn't match its frontmatter `name`, or it uses a disallowed top-level frontmatter key)
- **THEN** the skill fixes the violation directly and commits the fix as its own commit in the pull request

#### Scenario: Custom skill with only a judgment-based issue
- **WHEN** a custom/local skill's only validation issues are judgment-based (e.g. a vague description)
- **THEN** the skill reports the issue as a recommendation in the pull request description and does not rewrite the skill's content

#### Scenario: Registry-sourced skill
- **WHEN** a skill present in the repo is tracked in `skills-lock.json`
- **THEN** the skill is not re-validated against `skill-standard`, regardless of its actual conformance

#### Scenario: Custom skill already compliant
- **WHEN** a custom/local skill already passes every item in `skill-standard`'s validation checklist
- **THEN** the skill makes no change and does not report it as an issue

### Requirement: Architecture documentation generation
The skill SHALL analyze the target repo (stack, service/component boundaries, data model, messaging topology if present) and generate or merge `docs/ARCHITECTURE.md` covering system context, a component/container diagram, a service or component catalog, any messaging topology present, cross-component sequences, the shared data model, and known issues — using Mermaid diagrams and Markdown tables, grounded strictly in what the analysis actually finds. For a target repo with more than one independently-versioned service or component, the skill SHALL additionally generate or merge `docs/services/README.md` (an index) and one `docs/services/<name>.md` per component (role, tech stack and entry point, API surface, data models, messaging in/out, known issues, citing real file locations). For a single-service repo, the skill SHALL produce only `docs/ARCHITECTURE.md` and SHALL NOT create a `docs/services/` tier with a single padded entry. Writes to existing docs SHALL merge — preserving hand-authored prose and updating only sections demonstrably stale relative to the current repo state. These writes are committed as their own stage in the pull request.

#### Scenario: No existing docs
- **WHEN** the target repo has no `docs/` directory
- **THEN** the skill generates `docs/ARCHITECTURE.md` (and `docs/services/*` if the repo has multiple components) grounded in the stack analysis, without inventing services or endpoints not found in the repo

#### Scenario: Existing architecture docs present
- **WHEN** the target repo already has a `docs/ARCHITECTURE.md`
- **THEN** the skill merges updates into it — for example adding a newly-found component to the service catalog — while preserving existing hand-authored sections

#### Scenario: Single-service repo
- **WHEN** the stack analysis finds only one independently-versioned service in the repo
- **THEN** the skill produces only `docs/ARCHITECTURE.md`, without a `docs/services/` directory
</content>
