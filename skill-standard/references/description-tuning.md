# Description tuning (complete procedure)

The `description` frontmatter field is the primary **discovery** signal. The body loads only after activation.

## Rules

- **What + when** live in `description` only — not duplicated as activation prose in the body.
- Describe **user intent**, not internal skill mechanics.
- Include paraphrases and implicit triggers (e.g. user says "spreadsheet" not "CSV").
- Stay within **1024 characters**.
- Prefer imperative activation phrasing: "Use when…"

Complex or multi-step user requests are easier to match than one-line generic prompts.

## trigger_evals.json

Store beside the skill or in its workspace (convention, not part of core skill format):

```json
[
  {"query": "realistic message that should activate the skill", "should_trigger": true},
  {"query": "near-miss: overlapping words but wrong task", "should_trigger": false}
]
```

Target ~20 queries: ~10 should trigger, ~10 should not.

### Should-trigger variety

- Formal vs casual phrasing
- Explicit domain words vs indirect need
- Short vs long prompts
- Multi-step requests where the skill applies to part of the work

### Should-not-trigger (important)

Prefer **near-misses** — shared vocabulary, wrong goal. Weak negatives (unrelated topics) do not test precision.

Example for a tabular-data skill: "convert this json file to yaml" should **not** trigger; "clean up this export and fix null emails" **should**.

## Procedure

1. Freeze the skill **body**; edit only `description`.
2. For each query, start a **fresh session** with the same skill catalog as production.
3. Record whether the skill **activated** (client loaded `SKILL.md` or equivalent).
4. Revise description; re-run misfires.
5. Hold out a few queries for final verification; do not tune against the holdout set.

## Under-triggering

- Add concrete nouns and workflows from real user messages.
- Broaden phrasing slightly; avoid single-keyword descriptions.

## Over-triggering

- Narrow verbs and deliverables.
- Add lightweight negative scope ("Use for X, not for Y") only when it reduces false positives.

## Substantive queries

Activation tests should use realistic, detailed prompts. Trivial one-liners may not activate any skill even with a perfect description — that is a harness limitation, not proof the description failed.
