## 1. Add the machine-specific data scan

- [x] 1.1 Add a new Process step: read-only, whole-repo sweep for absolute local paths, usernames/emails, and private/internal repo or system names used as examples — per `design.md` Decision 1
- [x] 1.2 Document the pushed-vs-not-pushed distinction for a found leak, via `git merge-base --is-ancestor`, and that a leak already on the remote's default branch is flagged separately since redacting the working tree doesn't remove it from history — per `design.md` Decision 2
- [x] 1.3 Document that a confirmed leak with an obvious generic replacement is fixed directly (the one write exception beyond `skills.sh.json`/`README.md`, requiring user confirmation first), while a leak needing a git-history rewrite is always reported, never auto-attempted — per `design.md` Decision 2
- [x] 1.4 Update frontmatter `description` to mention the machine-specific-data scan alongside the existing catalog/README sync behavior

## 2. Name and document drift enforcement

- [x] 2.1 Add a "Drift enforcement" section defining the mechanical vs. judgment-based buckets explicitly, cross-referencing `agentify-project`'s own `references/custom-skill-drift-enforcement.md` as the precedent — per `design.md` Decision 3
- [x] 2.2 Label each existing Process step's findings with which bucket they fall into (additive gap, stale reference, schema violations, README gaps/staleness, the new leak scan) so the concept reads as one consistent rule, not five separate ones

## 3. Keep both harness copies in sync

- [x] 3.1 Apply all changes to `.claude/skills/skills-catalog-housekeeping/SKILL.md`
- [x] 3.2 Mirror the identical content into `.cursor/skills/skills-catalog-housekeeping/SKILL.md`
- [x] 3.3 Confirm the two copies are byte-identical (the skill's own self-consistency check)

## 4. Apply the scan to this repo and remediate what it found

- [x] 4.1 Run the new scan against this repo; confirm it surfaces the private reference repo's name and a local absolute path referencing it (across `openspec/specs/agentify-project-skill/spec.md` and the archived `enhance-agentify-project-staged-multi-harness-setup` change's docs) and this repo's own local checkout path (in the archived `add-skills-catalog-housekeeping-skill` change's `design.md`)
- [x] 4.2 Redact the working-tree content directly (mechanical fix, per Decision 2), after user confirmation
- [x] 4.3 Per explicit user request, scrub the same strings from git history itself using `git filter-repo` (after `git filter-branch` was attempted first and produced a corrupted rewrite that was caught and safely reverted via its own backup refs), since the affected commits were already pushed to the public remote
- [x] 4.4 Verify no leaked strings remain anywhere in history (`git log --all -S'<string>'` returns nothing) and that the rewritten repo is otherwise intact (correct branch, no fabricated commits)

## 5. Sync and archive

- [x] 5.1 Sync this change's spec delta into `openspec/specs/skills-catalog-housekeeping/spec.md`
- [x] 5.2 Archive this change once the spec sync is confirmed
