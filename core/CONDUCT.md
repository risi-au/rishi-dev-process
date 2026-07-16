# CONDUCT — how to write code here
<!-- HARD CAP: 80 lines. To add, remove (core/SELF-IMPROVE.md). -->

## Four rules (always on)

1. **Think before coding.** State assumptions; surface confusion; when multiple sane
   approaches exist, present options with a recommendation instead of silently choosing.
2. **Simplicity first.** If a senior engineer would call it overcomplicated, simplify.
3. **Surgical changes.** Touch only what the task requires. No drive-by edits, no
   opportunistic refactors, no abstractions for single uses. Remove only dead code
   that YOUR change orphaned.
4. **Goal-driven execution.** Verifiable success criteria exist before code is
   written. If you can't state how it will be checked, you aren't ready to write it.

## The lean ladder (run before writing anything new)

1. Does this need to exist at all? (YAGNI)
2. Does the codebase already do it? (search first — be lazy about writing,
   never about reading)
3. Does the stdlib do it?
4. Does the platform/framework already provide it?
5. Does an already-installed dependency do it?
6. Can it be a few lines instead of a new component/module?
7. Only then: write the minimum implementation that satisfies the criteria.

New dependencies need a stated reason and owner awareness. Fewer lines that pass
the gate beat more lines that impress.

## Security baseline (every lane, every worker)

- Secrets: names only in code, docs, logs, and reports. Values live in the
  vault/.env and never appear in output, commits, or worker packets.
- Never copy .env values, credential files, or user-data directories anywhere.
- Validate and parameterize at boundaries: no string-built SQL or shell, no
  unsanitized input into paths, HTML, or commands.
- Least privilege: services get their own users/keys; never widen permissions to
  make an error disappear without owner approval.
- Destructive operations (delete data, reset git state, drop tables, force-push,
  overwrite user files) require explicit owner approval, per instance.
- Dependencies: pin versions; prefer well-maintained, widely-used packages; flag
  anything with install scripts or build-time network access to the owner.

## Repo hygiene

- Match the project's existing style, naming, comment density, and encoding.
- Comments only for constraints the code can't show. No "fixed per review" narration.
- Tests prove the contract: a bugfix ships with the regression test that would have
  caught it; a feature ships with tests for its acceptance criteria.
