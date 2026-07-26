# Validation (complete checklist)

Apply before evals and before publishing. All rules are defined in this skill bundle; no external validator is required.

## Structure

- [ ] Skill root is a directory whose name equals frontmatter `name`
- [ ] `SKILL.md` exists at skill root
- [ ] File begins with `---`, valid YAML frontmatter, closing `---`
- [ ] Markdown body follows frontmatter

## Frontmatter — `name`

- [ ] Present
- [ ] Length 1–64
- [ ] Characters: lowercase letters, digits, hyphens only
- [ ] Does not start or end with `-`
- [ ] Does not contain `--`
- [ ] Equals parent directory name

## Frontmatter — `description`

- [ ] Present, non-empty after trim
- [ ] Length ≤1024
- [ ] Describes both capability and triggering context (what + when)
- [ ] Avoid angle brackets `<` `>` if your client/parser is strict (recommended)

## Frontmatter — optional fields

- [ ] Only allowed top-level keys: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`
- [ ] `compatibility` ≤500 characters if set
- [ ] `metadata` values are strings if set

## Body

- [ ] Steps are actionable without hidden project context
- [ ] Every linked file exists under the skill root
- [ ] Reference links state when to read them
- [ ] Main file ≤~500 lines; overflow in `references/`
- [ ] No host-specific absolute paths in instructions or examples

## Bundled resources

- [ ] Scripts document dependencies and failure modes
- [ ] No secrets or credentials in the tree

## Evals (if present)

- [ ] `evals/evals.json` is valid JSON
- [ ] `skill_name` matches frontmatter `name`
- [ ] Each `files[]` path exists relative to skill root
- [ ] Expectations are objectively verifiable where used

## Security

- [ ] Instructions match stated intent; no misleading or harmful content

## Automated validation (optional)

You may implement checks with a short script or YAML parser. Third-party CLI validators compatible with the Agent Skills format may be used but are **not** required for this workflow.
