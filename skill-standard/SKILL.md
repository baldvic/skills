---
name: skill-standard
description: Create, validate, and improve portable Agent Skills with structured evals and benchmarks. Use when authoring or editing SKILL.md, validating frontmatter, writing evals/evals.json, running with_skill vs without_skill baselines, producing grading.json or benchmark.json, or tuning descriptions for activation — in any agent harness.
license: MIT
---

# Skill Standard

Self-contained workflow for authoring skills in the **Agent Skills** open format and proving they help via **evals** and **benchmarks**. All format rules, JSON schemas, and grader/analyzer procedures live in this skill tree — read bundled files below at **runtime**; do not depend on external repos or product-specific CLIs when authoring other skills.

**Maintenance:** lineage and periodic refresh from upstream repos are documented in [references/upstream-sources.md](references/upstream-sources.md) and [references/upstream-sync.md](references/upstream-sync.md) (`upstream.lock.json` tracks last merged commits).

## Standalone reference map

| Need | Read |
|------|------|
| Full format spec | [references/agent-skills-format.md](references/agent-skills-format.md) |
| Validation checklist | [references/validation.md](references/validation.md) |
| Eval/benchmark files + JSON | [references/eval-artifacts.md](references/eval-artifacts.md) |
| Description / trigger tuning | [references/description-tuning.md](references/description-tuning.md) |
| Grade a run | [agents/grader.md](agents/grader.md) |
| Interpret benchmarks | [agents/analyzer.md](agents/analyzer.md) |
| Upstream repos + file lineage | [references/upstream-sources.md](references/upstream-sources.md) |
| Sync skill-standard from upstream | [references/upstream-sync.md](references/upstream-sync.md) |

Optional stdlib script: [scripts/aggregate_benchmark.py](scripts/aggregate_benchmark.py) — builds `benchmark.json` / `benchmark.md` from graded runs.

## Harness-agnostic rules

| Role | Definition |
|------|------------|
| **Executor** | Any agent session that runs the eval prompt with or without the skill available |
| **Grader** | Separate turn, human, or script that reads `transcript.md` + `outputs/` and writes `grading.json` |
| **Contract** | Files and JSON in [references/eval-artifacts.md](references/eval-artifacts.md) — not a vendor command |

- Parallel runs are optional; sequential execution is valid.
- Record only metrics the harness actually exposes in `timing.json` / `metrics.json`.
- Do not require subagents, proprietary trace formats, or bundled HTML viewers.
- Skills you write must use **relative paths** inside the skill tree; never embed host-specific absolute paths or secrets.

---

## Process

### 1. Capture intent

**Done when** you can answer:

1. What should the skill enable an agent to do?
2. When should it activate?
3. What does success look like?
4. Are outcomes objectively checkable or subjective?

Extract from conversation when the user says "turn this into a skill." Confirm gaps before drafting.

### 2. Draft the skill

Create `skill-name/SKILL.md` per [references/agent-skills-format.md](references/agent-skills-format.md):

- Directory name **equals** `name` (kebab-case, max 64).
- `description`: what + when, max 1024 chars — primary trigger.
- Body: how; under ~500 lines; `references/`, `scripts/`, `assets/` as needed.
- Allowed frontmatter: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`.
- Client-only flags → `metadata` string map, not extra top-level keys, when sharing skills.

### 3. Validate

Apply every item in [references/validation.md](references/validation.md). Fix before evals.

### 4. Define evals

Add `evals/evals.json` — schema in [references/eval-artifacts.md](references/eval-artifacts.md):

- 2–3 realistic prompts first; vary phrasing and edge cases.
- Fixtures under `evals/files/` when needed.
- Add `expectations` after first runs.

Improving an existing skill: copy current tree to `skill-snapshot/` in the workspace before edits.

### 5. Run the eval matrix

Workspace: `skill-name-workspace/` **sibling** to the skill folder.

For `iteration-N/` and each eval directory (`eval-<slug>/` or any dir with `eval_metadata.json`):

| Config | When |
|--------|------|
| `with_skill` | Skill available |
| `without_skill` | New skill baseline |
| `old_skill` | Improvement baseline (snapshot) |

Each config: at least `run-1/` containing `outputs/`, `transcript.md`, optional `timing.json` / `metrics.json`, later `grading.json`.

**Executor instructions** (same meaning in every harness):

- Complete the eval prompt.
- If skill active, follow `SKILL.md`.
- Write deliverables under `outputs/`.
- Write `transcript.md` (steps, tools by name, errors).
- Optional: `outputs/user_notes.md` for uncertainty.

Run baseline and skill configurations for each eval before drawing conclusions.

### 6. Grade

Follow [agents/grader.md](agents/grader.md). Write `grading.json` in each `run-*` directory.

Use scripts for structural expectations when possible.

### 7. Aggregate benchmark

From the directory that contains this skill's `scripts/` folder:

```bash
python scripts/aggregate_benchmark.py WORKSPACE/iteration-N --skill-name SKILL --skill-path SKILL
```

Use relative `skill_path` (skill folder name or path relative to repo root). Set `metadata.executor_model` in the JSON to whatever identifier the harness provides, or `unknown`.

Apply [agents/analyzer.md](agents/analyzer.md); fill `benchmark.json` → `notes`.

### 8. Human review

Show prompt, outputs, grading summary, config comparison. Collect:

```json
{
  "reviews": [{"run_id": "eval-slug-with_skill-run-1", "feedback": "...", "timestamp": "..."}],
  "status": "complete"
}
```

Subjective skills: prioritize human review over weak assertions.

### 9. Revise and repeat

Edit skill → `iteration-(N+1)/`. Stop when validation passes, expectations stable, user accepts quality.

### 10. Description tuning (optional)

If activation is wrong but execution is good, follow [references/description-tuning.md](references/description-tuning.md).

---

## Packaging

Ship the skill **folder**. Exclude from packages: `evals/`, workspace dirs, `__pycache__`, local benchmark artifacts.

---

## Minimal loop

1. Draft → validate → two eval prompts.
2. One `with_skill` and one `without_skill` run each → grade → review with user.
3. Revise once; expand evals only if the skill is shared or high-risk.

---

## Upstream sync (maintain skill-standard)

**Manual trigger only** — not part of the skill `description`. Invoke when you want to refresh this meta-skill from upstream (e.g. “run skill-standard upstream sync” or “follow upstream-sync for skill-standard”).

1. Follow the checklist pipeline in [references/upstream-sync.md](references/upstream-sync.md) end to end.
2. Use mappings in [upstream.lock.json](upstream.lock.json) and context in [references/upstream-sources.md](references/upstream-sources.md).
3. Append results to [references/upstream-changelog.md](references/upstream-changelog.md).
4. Preserve harness-agnostic editorial rules; drop harness-only upstream changes.

**Quick check:** compare `resolved_commit` in the lock file to `git ls-remote` on each upstream repo; run full pipeline only when SHAs differ or user requests merge.
