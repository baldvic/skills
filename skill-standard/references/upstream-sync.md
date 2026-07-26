# Upstream sync pipeline

Agent procedure to refresh skill-standard when [upstream-sources.md](upstream-sources.md) repos evolve. **Does not change runtime behavior** unless bundled files are updated and reviewed.

## When to run

- User asks to sync, update, or refresh skill-standard from upstream.
- Periodic maintenance (e.g. quarterly) after checking upstream commits.
- After a release or spec change announced on [agentskills/agentskills](https://github.com/agentskills/agentskills).

Skip sync if the user is only authoring **other** skills — this pipeline maintains **skill-standard itself**.

## Prerequisites

- Network access to clone public GitHub repos.
- Git available (shallow clone is enough).
- Write access to the skill-standard tree and `upstream.lock.json`.

Use a **temporary clone directory** under the repo or system temp — never commit clone dirs into skill-standard. Delete clones when done.

---

## Pipeline (ordered)

Copy progress checklist:

```
Upstream sync:
- [ ] Step 1: Load lock + mappings
- [ ] Step 2: Fetch upstream snapshots
- [ ] Step 3: Diff mapped paths
- [ ] Step 4: Classify changes
- [ ] Step 5: Port into local files
- [ ] Step 6: Re-apply editorial layer
- [ ] Step 7: Validate + smoke aggregate script
- [ ] Step 8: Update lock + write changelog
- [ ] Step 9: User review
```

### Step 1: Load lock and mappings

1. Read [upstream.lock.json](../upstream.lock.json).
2. Read [upstream-sources.md](upstream-sources.md) for lineage context.
3. Note each source's `resolved_commit` (may be `null` on first sync).

**Done when:** you know every `mappings[]` entry: upstream path → local path → `merge_strategy`.

### Step 2: Fetch upstream snapshots

For each entry in `upstream.lock.json` → `sources[]`:

```bash
git clone --depth 1 --branch DEFAULT_REF REPO_URL tmp-upstream/SOURCE_ID
cd tmp-upstream/SOURCE_ID && git rev-parse HEAD
```

Record `HEAD` as candidate `resolved_commit`. If `default_ref` fails, try `main` then default branch from `git remote show origin`.

For `anthropics-skill-creator`, paths are under `skill_root` (`skills/skill-creator/`).

**Done when:** each source has a fresh tree and commit SHA.

### Step 3: Diff mapped paths

For each mapping:

1. If `resolved_commit` is set, optionally fetch old commit for three-dot diff; for first sync, treat entire upstream file as new baseline.
2. Compare upstream file to local file (read both; summarize semantic deltas, not only line count).
3. Flag mappings whose upstream file **missing or moved** — update `upstream.lock.json` paths in the same PR after confirming new location upstream.

**Done when:** you have a per-file change summary (none / minor / major / upstream deleted).

### Step 4: Classify changes

| Class | Action |
|-------|--------|
| **Format spec** | Merge into `agent-skills-format.md` / `validation.md` — must stay portable |
| **Eval schema** | Merge into `eval-artifacts.md`; keep JSON field names compatible with `aggregate_benchmark.py` |
| **Procedure** | Merge into `grader.md` / `analyzer.md` / `description-tuning.md` via `procedure_strip_harness` |
| **Script** | Merge into `scripts/aggregate_benchmark.py` via `script_stdlib_only` |
| **Harness-only** | **Drop** — subagents, vendor CLIs, browser viewers, `present_files`, product-specific branches |
| **local_only** | Never replace from upstream |

**Done when:** every upstream delta is tagged keep / merge / drop.

### Step 5: Port into local files

Apply merges file by file:

#### merge_strategy: `spec_text`

- Incorporate new or changed normative rules into the local markdown.
- Keep standalone tone (no "see GitHub").
- Preserve portability section and harness-agnostic framing.

#### merge_strategy: `eval_schema`

- Update JSON examples and field tables in `eval-artifacts.md`.
- If `aggregate_benchmark.py` expects specific keys (`configuration`, `result.pass_rate`, etc.), update script in the same pass when upstream schema changes.

#### merge_strategy: `validation_rules`

- Align checklist bullets with upstream spec/tests.
- Do not require external CLI validators in the checklist.

#### merge_strategy: `procedure_strip_harness`

- Port grading/analysis **steps and schemas**.
- Replace executor/subagent instructions with: separate session, `transcript.md`, disk artifacts.
- Remove product names unless quoting an upstream example being explicitly rejected.

#### merge_strategy: `script_stdlib_only`

- Three-way merge or careful copy from upstream script.
- Keep stdlib-only; no new third-party deps.
- Preserve harness-neutral docstring and relative-path examples.

**Done when:** all targeted local files edited; no partial mappings left unstated.

### Step 6: Re-apply editorial layer

Mandatory pass on every touched file:

- [ ] No absolute host paths in examples
- [ ] No secrets or credentials
- [ ] No requirement for a specific agent product
- [ ] `SKILL.md` still states bundled self-containment for **runtime**
- [ ] Upstream URLs only in `upstream-sources.md`, `upstream.lock.json`, and this sync doc — not in authoring procedure bodies

### Step 7: Validate

1. Apply [validation.md](validation.md) mentally to the skill-standard folder (all bullets).
2. Run:

```bash
python scripts/aggregate_benchmark.py --help
```

1. If a scratch workspace with sample `grading.json` exists, run aggregate once to verify no crash.

**Done when:** validation passes and script runs.

### Step 8: Update lock and changelog

1. Set `last_sync_at` to ISO-8601 UTC in `upstream.lock.json`.
2. Set each source's `resolved_commit` to SHA from Step 2.
3. Adjust `mappings[].upstream` if upstream renamed files.
4. Write `references/upstream-changelog.md` entry (append):

```markdown
## YYYY-MM-DD

- agentskills @ <short-sha>: <one-line summary>
- anthropics-skill-creator @ <short-sha>: <one-line summary>
- Local edits: <editorial notes>
```

Create `upstream-changelog.md` if missing.

### Step 9: User review

Present:

- Commits synced per source
- Files changed locally
- Harness-only upstream changes deliberately dropped
- Any breaking schema changes and script updates

Ask user to accept before treating sync complete. Do not commit unless user requests.

---

## Merge strategies (reference)

| Strategy | Use for |
|----------|---------|
| `spec_text` | Format, best practices, description tuning prose |
| `eval_schema` | evals.json, grading.json, benchmark.json shapes, workspace layout |
| `validation_rules` | Frontmatter and structure checks |
| `procedure_strip_harness` | grader.md, analyzer.md |
| `script_stdlib_only` | aggregate_benchmark.py |

---

## Optional: compare installed skill-creator

If the user has `anthropics/skills@skill-creator` installed under `.agents/skills/skill-creator`, diff that tree against `tmp-upstream` copy to detect install drift. Prefer **git upstream** as source of truth; installed copy is a convenience check.

---

## Failure handling

| Situation | Action |
|-----------|--------|
| Upstream file removed | Search repo for replacement; update mapping or document intentional omission |
| Schema break | Update `eval-artifacts.md` + `aggregate_benchmark.py` together |
| Conflicting spec vs skill-creator | Prefer **agentskills** for format; prefer **skill-creator** for eval JSON unless agentskills docs explicitly supersede |
| No upstream changes since lock | Report "Already up to date at \<sha\>"; still refresh `last_sync_at` only if user wants timestamp bump |

---

## Quick status (no full sync)

1. Read `resolved_commit` from lock.
2. `git ls-remote REPO REF` for each source.
3. If SHAs match, tell user sync not needed; else offer full pipeline.
