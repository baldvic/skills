# Grader (complete procedure)

Evaluate **expectations** against **transcript.md** and **outputs/**. No specific agent product or tool API is assumed — only files on disk.

## Inputs

- **expectations** — strings from `evals/evals.json` or `eval_metadata.json`
- **transcript_path** — `transcript.md` in the run directory
- **outputs_dir** — `outputs/` in the run directory

## Process

1. Read `transcript.md` completely (prompt, steps, tool names, errors, outcome).
2. List and inspect `outputs/`; open files relevant to each expectation; run helper scripts for binary or structured formats when needed.
3. For each expectation: search transcript and outputs; **PASS** or **FAIL**; quote or describe evidence. No partial credit.
4. Extract implicit **claims** from outputs; verify when possible (`factual`, `process`, `quality`).
5. If `outputs/user_notes.md` exists, summarize into `user_notes_summary`.
6. Critique the eval definitions: trivial passes, missing checks → `eval_feedback.suggestions` when warranted.
7. Copy optional `metrics.json` and `timing.json` from the run directory into the grading output.

## Verdict bar

- **PASS** — substantive evidence; not filename-only or coincidental matches.
- **FAIL** — missing, contradicted, superficial, or unverifiable.
- Uncertain → **FAIL**.

## Output location

Write `grading.json` in the **run directory** (same directory as `transcript.md`, parent of `outputs/`).

## grading.json schema

```json
{
  "expectations": [
    {
      "text": "The output includes the name 'Example User'",
      "passed": true,
      "evidence": "transcript.md step 4; outputs/summary.txt contains the name"
    },
    {
      "text": "The spreadsheet has a SUM formula in cell B10",
      "passed": false,
      "evidence": "No spreadsheet in outputs/; only summary.txt was created"
    }
  ],
  "summary": {
    "passed": 1,
    "failed": 1,
    "total": 2,
    "pass_rate": 0.5
  },
  "execution_metrics": {
    "tool_calls": {},
    "total_tool_calls": 0,
    "total_steps": 0,
    "errors_encountered": 0,
    "output_chars": 0,
    "transcript_chars": 0
  },
  "timing": {
    "total_duration_seconds": 0.0
  },
  "claims": [
    {
      "claim": "Output lists three items",
      "type": "factual",
      "verified": true,
      "evidence": "Counted three lines in outputs/list.txt"
    }
  ],
  "user_notes_summary": {
    "uncertainties": [],
    "needs_review": [],
    "workarounds": []
  },
  "eval_feedback": {
    "suggestions": [
      {
        "assertion": "The output includes the name 'Example User'",
        "reason": "Any file mentioning the name would pass — consider checking primary field matches input"
      }
    ],
    "overall": "Assertions check presence but not correctness."
  }
}
```

## Independence

Grade in a **separate** session from the executor when possible (different agent turn, human, or script). Same-session grading is acceptable only for quick smoke tests.

## Programmatic checks

When an expectation is purely structural (file exists, JSON field present, line count), prefer a small script over visual inspection; reuse scripts across iterations.
