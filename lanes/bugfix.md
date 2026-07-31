# LANE: Bugfix
<!-- CAP: 80 lines -->

Prereqs: START.md + core read; project overlay read.

## Steps

1. **Intake.** Symptom, environment, expected vs actual. Issue preferred.
2. **Reproduce.** A bug you can't reproduce is an investigation — switch lane.
   Capture the exact repro steps/command. Exception: for timing/environment-bound
   bugs, an evidence chain (screenshot/log + code path + known upstream bug) may
   substitute for live repro — declare the substitution in the finish report and
   re-run the original repro after deploy.
3. **Structural analysis** (R1+ and multi-file only; see `core/BUG-INTAKE.md`).
   Query the code graph BEFORE dispatch: `code-review-graph` for callers/tests/impact,
   `graphify` for conceptual context. Pre-fill ticket with: affected files, call paths,
   test gaps, impact radius, root cause hypothesis. Trivial/R0 bugs skip this.
4. **Minimise.** Shrink the repro to the smallest failing case. The root cause
   usually reveals itself here — do not skip ahead to patching symptoms.
5. **Root cause.** State it in one sentence. If it was non-obvious, list the
   hypotheses you eliminated (cheap insurance against fixing the wrong thing).
6. **Triage** size + risk. Most bugfixes are Standard / R0–R1 → plan-lite or none.
   Auth, data, or migration bugs are R2. Record risk profile in ticket frontmatter.
7. **Fix surgically.** The smallest change that removes the root cause. No
   opportunistic cleanup — that's the refactor lane.
8. **Regression test.** Write the test that fails on the old code and passes on
   the new. A fix without a regression test is not done (if the project has no
   test infra, say so explicitly in the finish report).
9. **Gate → review → release → retro** per `core/PROCESS.md`. Obvious one-liners
   may run as the trivial lane instead.

## Lane rules

- Never claim fixed without re-running the original repro (for substituted repros
  the re-check happens post-deploy — track it in the PR so it isn't forgotten).
- If the fix grows past ~3 files or touches auth/data, stop and re-triage.
- If two fix attempts fail, stop: re-diagnose (investigation lane) rather than
  iterating blindly.
