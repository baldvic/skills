# Custom/local skill validation and drift enforcement

Applies only to skills the detection map classifies as custom/local — not to registry-sourced skills. Read this before acting on the detection map's skill inventory.

## Classification: registry-sourced vs. custom/local

During the detection map's inventory of `.claude/skills/`, `.cursor/skills/`, and `.agents/skills/` (see [preflight-gate-and-detection.md](preflight-gate-and-detection.md)), classify every skill folder found:

- **Registry-sourced** — tracked as an entry in the root `skills-lock.json` (installed via `find-skills` + `npx skills add <source>@<skill> -y`, per [agent-readiness-checklist.md](agent-readiness-checklist.md)'s skill-wiring pillar). Trusted by construction. **Never re-validated**, regardless of its actual conformance to `skill-standard` — the lockfile entry is the trust boundary, not the skill's content.
- **Custom/local** — present in a skills directory but **not** listed in `skills-lock.json`. Hand-authored or vendored some other way. Subject to the validation below.

## Validation: mechanical vs. judgment-based

Check every custom/local skill against the full checklist in `skill-standard/references/validation.md` — don't duplicate that checklist here, read it directly. Split whatever it finds into two kinds:

**Mechanical** (deterministic, safe to fix without asking):
- Frontmatter field errors (missing/malformed `name` or `description`, invalid characters, length violations)
- Directory name ≠ frontmatter `name`
- Disallowed top-level frontmatter keys (anything other than `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`)
- Host-specific absolute paths in instructions or examples
- `description` over the length limit

**Judgment-based** (subjective, never auto-rewritten):
- Vague or low-quality `description` (doesn't state both capability and triggering context)
- Non-actionable steps (instructions that assume hidden project context)

## Handling each kind

- **Mechanical violations** — fix directly (rename the directory, correct the frontmatter, strip the disallowed key, genericize the absolute path, trim the description) and commit the fix as **its own PR commit**, separate from every other stage's commit. This keeps a mechanical fix independently reviewable and revertable without touching anything else the run did.
- **Judgment-based violations** — report as a recommendation in the PR description's "Judgment-based recommendations" section (see [pr-delivery.md](pr-delivery.md)). Name the skill and the specific issue. Never rewrite the skill's actual content to satisfy a subjective checklist item — that risks changing intent the skill's original author had, which is exactly the kind of call that shouldn't happen silently even under PR review (a rewritten description is much harder for a reviewer to sanity-check than a frontmatter fix).
- **Already compliant** — no change, no report. A custom/local skill that already passes every checklist item isn't mentioned as an issue anywhere.

## Edge cases

- **A custom/local skill has both kinds of violations** — fix the mechanical ones and commit them; still report the judgment-based ones separately. One doesn't block or subsume the other.
- **No custom/local skills found** (everything present is registry-sourced, or no skills directory exists at all) — report "no custom/local skills found" in the PR description; this is a satisfied state, not a gap.
