# skills.sh.json schema (confirmed shape)

Source: https://skills.sh/schemas/skills.sh.schema.json — verified 2026-07-27. This is a snapshot, not a live check: if skills.sh changes this schema, re-fetch that URL to see what's drifted rather than trusting this file forever.

## Top-level fields

| Field | Required | Type | Notes |
|---|---|---|---|
| `groupings` | Yes | array, 1–50 items | Sections shown on the repo page |
| `$schema` | No | string (URI) | Preferred way to reference the schema, for editor validation |
| `schema` | No | string (URI) | Legacy alias for `$schema` — prefer `$schema` when both could apply |
| `notGrouped` | No | enum: `"top"` \| `"bottom"` | Where to place skills not listed in any grouping. Default: `"bottom"` |

## `groupings[]` entry shape

| Field | Required | Constraints |
|---|---|---|
| `title` | Yes | 1–120 characters |
| `skills` | Yes | 1–500 skill names/slugs |
| `description` | No | Up to 500 characters |

## Mechanical checks this skill performs

- `schema` present without `$schema` → migrate to `$schema` directly.
- `notGrouped` present but not `"top"`/`"bottom"` → report; don't guess which one was intended.
- A `groupings[]` entry with an empty `skills` array, or a `title` that's empty or over 120 characters → report.
- More than 50 groupings, or a single grouping listing over 500 skills → report. (This repo is nowhere near these limits today, but the check is mechanical and cheap either way.)
