## ADDED Requirements

### Requirement: Agent-readiness audit
The `agentify-project` skill SHALL audit a target codebase, read-only, against a four-pillar agent-readiness checklist (agent instructions file presence/freshness, discoverable/relevant skill wiring, tool-permission hygiene where the in-scope harness has an established mechanism, candidate MCP server notes) before proposing or applying any change, and SHALL present the findings as a report even when no changes are ultimately applied.

#### Scenario: Fresh repo with no agent scaffolding
- **WHEN** the skill is run against a repository that has no agent instructions file, no wired skills, and no harness permission file
- **THEN** the skill reports all four pillars as gaps before proposing any fix, and does not write any file until the user reviews the report

#### Scenario: Already agent-ready repo
- **WHEN** the skill is run against a repository that already has current agent instructions, relevant skills wired, reasonable permission configuration for its harness, and no missing MCP opportunities
- **THEN** the skill reports each pillar as satisfied and makes no changes

### Requirement: Harness detection precedes harness-specific pillars
The skill SHALL determine which agent harness(es) are in scope for a run — at minimum distinguishing Claude Code and Cursor — before acting on the agent-instructions or skill-wiring pillars, by checking for existing harness-specific directories/files in the target repo (e.g. `.claude/` implies Claude Code, `.cursor/` implies Cursor). When no such signal is present, or more than one plausibly applies and the user hasn't said which they want, the skill SHALL ask rather than guess.

#### Scenario: Single harness signal present
- **WHEN** the target repo has a `.claude/` directory and no `.cursor/` directory
- **THEN** the skill targets Claude Code's conventions for the agent-instructions and skill-wiring pillars without asking

#### Scenario: No harness signal present
- **WHEN** the target repo has neither a `.claude/` nor a `.cursor/` directory
- **THEN** the skill asks the user which harness(es) to set up for before drafting any instructions file or wiring any skill

#### Scenario: Both harness signals present
- **WHEN** the target repo has both `.claude/` and `.cursor/` directories
- **THEN** the skill treats both harnesses as in scope and applies the agent-instructions and skill-wiring pillars for each, keeping their outputs consistent with each other

### Requirement: Agent instructions generation and update supersedes `init`
The skill SHALL generate the agent instructions file(s) for whichever harness(es) are in scope itself — `CLAUDE.md` for Claude Code, Cursor project rules (`.cursor/rules/*.mdc`, or the existing `.cursorrules` if that's what the repo already uses) for Cursor — and SHALL propose a diff-based update when any of them already exist, without delegating this responsibility to the built-in `init` skill. Overwriting or substantially restructuring an existing instructions file SHALL require explicit user confirmation of the shown diff before writing. When both harnesses are in scope, the skill SHALL keep their instructions content consistent with each other rather than letting one go stale.

#### Scenario: No existing instructions file for the in-scope harness
- **WHEN** the target repo has no `CLAUDE.md` and Claude Code is in scope (or no Cursor project rules and Cursor is in scope)
- **THEN** the skill drafts the appropriate file from its own template and writes it directly, since there is no existing content to lose

#### Scenario: Existing instructions file present
- **WHEN** the target repo already has an instructions file for an in-scope harness
- **THEN** the skill computes a diff against its proposed content, shows the diff to the user, and writes it only after the user confirms

#### Scenario: Both harnesses in scope with only one instructions file present
- **WHEN** the target repo has `CLAUDE.md` but no Cursor project rules, and both `.claude/` and `.cursor/` are in scope
- **THEN** the skill offers to draft Cursor project rules with equivalent content, rather than leaving Cursor without any instructions

### Requirement: Skill wiring is recommend-first and copy-on-confirm, per harness directory
The skill SHALL match the target repo's detected stack against skills discoverable in the current environment or a user-specified local skills source, present a shortlist with rationale, and SHALL copy a recommended skill's files into the target repo's harness-appropriate local skill directory — `.claude/skills/<name>/` for Claude Code, `.cursor/skills/<name>/` for Cursor — only after the user confirms that specific recommendation, wiring into each in-scope harness's directory when more than one harness is in scope. It SHALL NOT fabricate or invent skill content that does not already exist in an accessible source.

#### Scenario: Relevant skills found for a single in-scope harness
- **WHEN** the target repo's stack matches one or more discoverable skills not yet wired into the repo, and only Claude Code is in scope
- **THEN** the skill lists each match with a one-line rationale and copies only the confirmed ones into `.claude/skills/`

#### Scenario: Both harnesses in scope
- **WHEN** both Claude Code and Cursor are in scope and a recommendation is confirmed
- **THEN** the skill copies the confirmed skill into both `.claude/skills/<name>/` and `.cursor/skills/<name>/`

#### Scenario: No relevant skills discoverable
- **WHEN** no discoverable skill matches the target repo's detected stack
- **THEN** the skill reports "no skill recommendations found" as a valid outcome rather than substituting an unrelated or fabricated skill

### Requirement: Tool-permission hygiene is scoped to harnesses with an established mechanism
For a harness with an established, documented, repo-committed permission mechanism — currently Claude Code's `.claude/settings.json` `permissions.allow` — the skill SHALL merge a small, well-known safe read-only command allowlist in automatically when entries are missing, and SHALL require explicit confirmation with a shown diff before making any change that broadens an existing permission, adds a non-read-only command, or touches `permissions.deny` or hook configuration. For a harness without an established repo-committed permission mechanism, the skill SHALL report that this pillar has no applicable action for that harness rather than fabricating a config format for it.

#### Scenario: Missing safe read-only entries for Claude Code
- **WHEN** `.claude/settings.json` is missing common safe read-only allowlist entries (e.g. `git status`, `git log`, `git diff`) and Claude Code is in scope
- **THEN** the skill adds them automatically without a confirmation prompt

#### Scenario: Proposed change would broaden permissions
- **WHEN** a proposed `.claude/settings.json` change would widen an existing rule, add a non-read-only command, or modify `permissions.deny` or hooks
- **THEN** the skill shows the diff and requires explicit user confirmation before writing

#### Scenario: In-scope harness has no established permission mechanism
- **WHEN** Cursor (or another harness without a documented repo-committed permission convention as of this skill's authoring) is the harness in scope
- **THEN** the skill reports this pillar as not applicable for that harness and does not invent or write a permission file for it

### Requirement: MCP server suggestions are advisory only
The skill SHALL note candidate MCP servers as advisory text (name and rationale) in its report or agent-instructions output, and SHALL NOT generate MCP server configuration, endpoints, or credentials on the user's behalf, regardless of which harness is in scope.

#### Scenario: Stack suggests a useful MCP server
- **WHEN** the target repo's detected stack (e.g. a database, a ticketing system) suggests an MCP server would help
- **THEN** the skill writes a one-line advisory suggestion naming the server and why it would help, without creating any config file or credential, the same way regardless of harness

### Requirement: Idempotent re-runs
Re-running the skill against a repository it has already agentified SHALL converge rather than duplicate content — repeated runs on an unchanged repo SHALL propose no further changes, for any harness(es) previously agentified.

#### Scenario: Second run on an already-agentified repo
- **WHEN** the skill is run a second time against a repo it previously agentified with no changes to the repo in between
- **THEN** the audit reports all pillars satisfied and no diffs are proposed
