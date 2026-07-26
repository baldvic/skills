# Eval and benchmark artifacts (complete schemas)

Harness-agnostic file contract. Any agent client that writes these paths and JSON shapes can run the same eval and benchmark loop.

## evals/evals.json (inside skill under test)

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "Realistic user message",
      "expected_output": "Human-readable success criteria",
      "files": ["evals/files/sample.csv"],
      "expectations": [
        "Output file outputs/result.csv exists",
        "Column profit_margin is present"
      ]
    }
  ]
}
```

| Field | Purpose |
|-------|---------|
| `skill_name` | Must match `SKILL.md` `name` |
| `evals[].id` | Stable integer |
| `prompt` | Task given to executor |
| `expected_output` | For humans and graders |
| `files` | Input paths relative to skill root |
| `expectations` | Verifiable checks; graded into `grading.json` |

Start with prompts only; add `expectations` after first runs.

## eval_metadata.json (per eval in workspace)

```json
{
  "eval_id": 1,
  "eval_name": "descriptive-slug",
  "prompt": "Same as evals.json entry",
  "expectations": []
}
```

## Workspace layout

Sibling of the skill directory (keeps eval fixtures out of distributable packages):

```
example-skill/
├── SKILL.md
└── evals/
    ├── evals.json
    └── files/

example-skill-workspace/
└── iteration-1/
    ├── eval-descriptive-slug/
    │   ├── eval_metadata.json
    │   ├── with_skill/
    │   │   └── run-1/
    │   │       ├── outputs/
    │   │       ├── transcript.md
    │   │       ├── timing.json
    │   │       ├── metrics.json
    │   │       └── grading.json
    │   └── without_skill/
    │       └── run-1/ ...
    ├── benchmark.json
    ├── benchmark.md
    └── feedback.json
```

Eval directories: prefix `eval-` **or** include `eval_metadata.json`.

### Configuration directories

| Name | Use |
|------|-----|
| `with_skill` | Skill loaded; agent follows `SKILL.md` |
| `without_skill` | Baseline for **new** skills — same prompt, skill not loaded |
| `old_skill` | Baseline for **improvements** — copy of previous version |

Use at least `run-1/`; add `run-2`, `run-3` for variance.

## transcript.md (required per run)

Plain-text log: eval prompt, steps taken, tools invoked (names only), errors, final outcome. Enables grading without proprietary trace formats.

## timing.json (optional)

```json
{
  "total_tokens": 0,
  "duration_ms": 0,
  "total_duration_seconds": 0.0,
  "executor_start": "2026-01-15T10:30:00Z",
  "executor_end": "2026-01-15T10:32:45Z"
}
```

Record only fields the harness provides; omit or zero unknown metrics.

## metrics.json (optional)

```json
{
  "tool_calls": {},
  "total_tool_calls": 0,
  "total_steps": 0,
  "files_created": [],
  "errors_encountered": 0,
  "output_chars": 0,
  "transcript_chars": 0
}
```

Keys under `tool_calls` are harness-specific labels.

## grading.json (required after grading)

Write in the run directory (parent of `outputs/`). Expectation objects **must** use `text`, `passed`, `evidence`.

```json
{
  "expectations": [
    {
      "text": "Output includes header row",
      "passed": true,
      "evidence": "outputs/result.csv line 1: id,name,value"
    }
  ],
  "summary": {
    "passed": 1,
    "failed": 0,
    "total": 1,
    "pass_rate": 1.0
  },
  "execution_metrics": {},
  "timing": {},
  "claims": [
    {
      "claim": "Result has three columns",
      "type": "factual",
      "verified": true,
      "evidence": "Header row lists three columns"
    }
  ],
  "user_notes_summary": {
    "uncertainties": [],
    "needs_review": [],
    "workarounds": []
  },
  "eval_feedback": {
    "suggestions": [],
    "overall": "No suggestions, evals look solid"
  }
}
```

Grading procedure: [agents/grader.md](../agents/grader.md).

## benchmark.json

Produced by `scripts/aggregate_benchmark.py` or authored manually. **Exact field names** below matter for aggregation and review tooling.

```json
{
  "metadata": {
    "skill_name": "example-skill",
    "skill_path": "example-skill",
    "executor_model": "unknown",
    "analyzer_model": "unknown",
    "timestamp": "2026-01-15T10:30:00Z",
    "evals_run": [1, 2],
    "runs_per_configuration": 1
  },
  "runs": [
    {
      "eval_id": 1,
      "eval_name": "eval-descriptive-slug",
      "configuration": "with_skill",
      "run_number": 1,
      "result": {
        "pass_rate": 1.0,
        "passed": 1,
        "failed": 0,
        "total": 1,
        "time_seconds": 0.0,
        "tokens": 0,
        "tool_calls": 0,
        "errors": 0
      },
      "expectations": [],
      "notes": []
    }
  ],
  "run_summary": {
    "with_skill": {
      "pass_rate": {"mean": 0.0, "stddev": 0.0, "min": 0.0, "max": 0.0},
      "time_seconds": {"mean": 0.0, "stddev": 0.0, "min": 0.0, "max": 0.0},
      "tokens": {"mean": 0, "stddev": 0, "min": 0, "max": 0}
    },
    "without_skill": {
      "pass_rate": {"mean": 0.0, "stddev": 0.0, "min": 0.0, "max": 0.0},
      "time_seconds": {"mean": 0.0, "stddev": 0.0, "min": 0.0, "max": 0.0},
      "tokens": {"mean": 0, "stddev": 0, "min": 0, "max": 0}
    },
    "delta": {
      "pass_rate": "+0.00",
      "time_seconds": "+0.0",
      "tokens": "+0"
    }
  },
  "notes": []
}
```

- `runs[].configuration` — use `with_skill` and `without_skill` (or consistent baseline names).
- `runs[].result` — metrics nested under `result`, not at run top level.

## history.json (optional, workspace root)

Tracks versions when comparing skill iterations:

```json
{
  "started_at": "2026-01-15T10:30:00Z",
  "skill_name": "example-skill",
  "current_best": "v2",
  "iterations": [
    {
      "version": "v0",
      "parent": null,
      "expectation_pass_rate": 0.65,
      "grading_result": "baseline",
      "is_current_best": false
    },
    {
      "version": "v2",
      "parent": "v1",
      "expectation_pass_rate": 0.85,
      "grading_result": "won",
      "is_current_best": true
    }
  ]
}
```

`grading_result`: `baseline`, `won`, `lost`, or `tie`.

## feedback.json (human review)

```json
{
  "reviews": [
    {
      "run_id": "eval-descriptive-slug-with_skill-run-1",
      "feedback": "Free-text review",
      "timestamp": "2026-01-15T12:00:00Z"
    }
  ],
  "status": "complete"
}
```

## analysis.json (optional, A/B comparison)

```json
{
  "comparison_summary": {
    "winner": "with_skill",
    "comparator_reasoning": "Brief summary"
  },
  "improvement_suggestions": [
    {
      "priority": "high",
      "category": "instructions",
      "suggestion": "Replace vague step with explicit sequence",
      "expected_impact": "Reduces inconsistent execution"
    }
  ]
}
```

Analyzer guidance: [agents/analyzer.md](../agents/analyzer.md).
