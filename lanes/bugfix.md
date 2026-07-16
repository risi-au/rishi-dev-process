# LANE: Bugfix
<!-- CAP: 80 lines -->

Prereqs: START.md + core read; project overlay read.

## Steps

1. **Intake.** Symptom, environment, expected vs actual. Issue preferred.
2. **Reproduce.** A bug you can't reproduce is an investigation — switch lane.
   Capture the exact repro steps/command.
3. **Minimise.** Shrink the repro to the smallest failing case. The root cause
   usually reveals itself here — do not skip ahead to patching symptoms.
4. **Root cause.** State it in one sentence. If it was non-obvious, list the
   hypotheses you eliminated (cheap insurance against fixing the wrong thing).
5. **Triage** size + risk. Most bugfixes are Standard / R0–R1 → plan-lite or none.
   Auth, data, or migration bugs are R2.
6. **Fix surgically.** The smallest change that removes the root cause. No
   opportunistic cleanup — that's the refactor lane.
7. **Regression test.** Write the test that fails on the old code and passes on
   the new. A fix without a regression test is not done (if the project has no
   test infra, say so explicitly in the finish report).
8. **Gate → review → release → retro** per `core/PROCESS.md`. Obvious one-liners
   may run as the trivial lane instead.

## Lane rules

- Never claim fixed without re-running the original repro.
- If the fix grows past ~3 files or touches auth/data, stop and re-triage.
- If two fix attempts fail, stop: re-diagnose (investigation lane) rather than
  iterating blindly.
