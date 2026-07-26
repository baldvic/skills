# Benchmark analyzer (complete procedure)

After `benchmark.json` exists, add qualitative **notes** that summary statistics hide. No vendor tooling required.

## Read first

- `run_summary` — per-configuration means, stddev, `delta`
- Each entry in `runs[]` — `expectations`, `notes`, `result`
- `feedback.json` from the same iteration (if present)
- `eval_feedback` inside individual `grading.json` files when patterns repeat

## Patterns to flag

| Pattern | Action |
|---------|--------|
| Expectation passes 100% in both configurations | Non-discriminating — tighten or replace |
| High variance on one eval across runs | Flaky prompt or environment — note; add runs |
| Pass rate up but time/tokens up sharply | Document cost tradeoff in `notes` |
| Baseline fails same expectation every time | Strong signal the skill adds value |
| Repeated grader suggestions | Feed into skill edits and `evals.json` |

## Write

Append short bullet strings to `benchmark.json` → `notes`. Each note one observable finding.

For version-vs-version comparisons, optionally add `analysis.json` beside the iteration:

```json
{
  "comparison_summary": {
    "winner": "with_skill",
    "comparator_reasoning": "Higher pass rate on multi-step evals with fewer workarounds"
  },
  "winner_strengths": [
    "Explicit validation step before write"
  ],
  "loser_weaknesses": [
    "Vague instruction led to three different approaches in transcript"
  ],
  "improvement_suggestions": [
    {
      "priority": "high",
      "category": "instructions",
      "suggestion": "Replace vague step with numbered substeps",
      "expected_impact": "Less variance on eval 2"
    }
  ]
}
```

Tie every suggestion to evidence from transcripts, outputs, or grading — not generic best practices.
