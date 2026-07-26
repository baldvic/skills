# Agent Skills format (complete)

Portable open format for extending agents with specialized instructions and bundled resources. This document is the **full format reference** bundled with skill-standard — no external spec required.

## Directory layout

```
skill-name/
├── SKILL.md          # Required: YAML frontmatter + Markdown body
├── scripts/          # Optional: executable code the agent may run
├── references/       # Optional: documentation loaded on demand
├── assets/           # Optional: templates, images, data for outputs
└── evals/            # Optional: evals.json + fixtures (development only; omit from packages)
```

The directory name **must equal** frontmatter `name`.

## Progressive disclosure

Agents should treat skills in three stages:

1. **Discovery** — load only `name` and `description` for each available skill.
2. **Activation** — when the task matches the description, load the full `SKILL.md` body.
3. **Execution** — read or run files under `scripts/`, `references/`, `assets/` only as the skill directs.

Keep `SKILL.md` under ~500 lines (~5000 tokens). Move depth to `references/` and link with **when to read** cues.

## SKILL.md structure

File order:

1. YAML frontmatter between `---` lines
2. Markdown body (instructions)

No other required structure for the body. Recommended content: ordered steps, input/output examples, edge cases.

## Frontmatter fields

| Field | Required | Constraints |
|-------|----------|-------------|
| `name` | Yes | 1–64 chars; lowercase `a-z`, digits `0-9`, hyphens `-` only; no leading/trailing `-`; no `--`; must match parent directory name |
| `description` | Yes | 1–1024 chars; non-empty; states **what the skill does** and **when to use it**; include keywords that help activation |
| `license` | No | Short license name or pointer to a bundled license file |
| `compatibility` | No | 1–500 chars if present; environment needs (runtime, packages, network) |
| `metadata` | No | Map of string keys to string values; use namespaced keys for client-specific flags |
| `allowed-tools` | No | Experimental: space-separated tool allowlist (support varies by client) |

### Minimal frontmatter

```yaml
---
name: skill-name
description: What this skill does and when the agent should use it.
---
```

### Full example

```yaml
---
name: pdf-processing
description: Extract PDF text, fill forms, merge files. Use when handling PDF documents or when the user mentions PDFs, forms, or document extraction.
license: Apache-2.0
compatibility: Requires a Python runtime if using bundled scripts.
metadata:
  author: example-org
  version: "1.0"
---
```

### Invalid `name` examples

- `PDF-Processing` — uppercase not allowed
- `-pdf` — cannot start with hyphen
- `pdf--processing` — consecutive hyphens
- `my_skill` — underscores not allowed

### Good vs poor `description`

Good:

```yaml
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
```

Poor:

```yaml
description: Helps with PDFs.
```

### `allowed-tools` example (experimental)

```yaml
allowed-tools: Bash(git:*) Read
```

## Body content

Write whatever helps the agent perform the task. Prefer imperative instructions.

**File references** — relative to skill root, one level deep from `SKILL.md`:

```markdown
See [API errors](references/api-errors.md) when a request returns a non-success status.

Run: scripts/validate.sh
```

## Optional directories

### scripts/

Executable code. Should be self-contained or document dependencies; clear errors; handle edge cases. Language depends on the agent implementation (common: Python, shell, JavaScript).

### references/

Extra documentation loaded on demand. Split by topic; keep files focused.

### assets/

Static resources: templates, images, lookup tables — not loaded into context unless needed.

## Portability rules (skills you author)

- Use **relative paths** only inside the skill tree; never embed host-specific absolute paths.
- Do not put secrets, credentials, or personal identifiers in skills.
- Put client-only behavior in `metadata` with documented keys, not non-standard top-level frontmatter, when sharing across clients.
- Do not rely on a specific agent product, CLI, or subagent API in the skill text unless scoped under `compatibility` or `metadata`.

## Authoring principles

- Ground content in real tasks, runbooks, or corrected traces — not generic filler.
- Add what the agent would get wrong without the skill; omit common knowledge.
- Scope one coherent unit of work; split skills when activation or step sequences conflict.
- Match prescriptiveness to fragility: rigid steps for brittle ops; goals when multiple valid approaches exist.
- Explain *why* when it improves judgment; avoid empty MUST spam.
- For multi-domain skills, use `references/` per variant and select in the body.
