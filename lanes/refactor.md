# LANE: Refactor — behavior-preserving restructure
<!-- CAP: 80 lines -->

For: cleanup, dedup, renames, module splits, dead-code removal, and performance
work that doesn't change observable behavior.

## Steps

1. **State the goal + invariant.** What improves (readability / duplication /
   perf) and the invariant: observable behavior unchanged (or an explicit perf
   target, with a benchmark).
2. **Prove the safety net FIRST.** Run the full gate on the untouched code and
   record a baseline gate receipt. If coverage on the refactor target is thin,
   write characterization tests BEFORE moving anything.
3. **Triage.** Small mechanical refactor = Standard / R0–R1. Anything touching
   public interfaces, auth paths, data handling, or >~10 files = Heavy / R2 with a
   plan and owner approval.
4. **Refactor in reviewable steps.** Separate commits for mechanical moves/renames
   vs logic-touching edits. Never mix a refactor with a feature or bugfix.
5. **Gate after each step** against the baseline: same tests pass, no new warnings.
6. **Review.** The reviewer packet includes the baseline receipt so the reviewer
   verifies behavior-preservation, not just code taste.
7. **Release → retro** as usual.

## Lane rules

- No features ride along. If the refactor reveals a bug, file it (bugfix lane);
  fixing it here silently changes behavior and breaks the invariant.
- Deleted code must be provably dead — grep alone is not proof where reflection or
  dynamic dispatch exists; state how you proved it.
- If the gate can't prove preservation (no tests, no types), say so and get owner
  approval for the risk before proceeding.
