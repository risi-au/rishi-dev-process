# Retro — companyos-lexi, bug batch → session attribution → main-red guard

**Date:** 2026-08-01 · **Lane:** bugfix → feature (R2) · **Orchestrator:** Claude Opus 5
**Owner approvals:** batch scope; merge-own-PRs; lift a worktree edit lock; plan-full for
the R2 attribution work; autonomous run while away; add these rules to core.

## Metrics

- Shipped: 7 commits, PRs #249 #254 #255 #256 #257 #258 #259, docs PRs #7 #8. All merged
  and deployed; production verified live after each.
- Workers: 6 × sonnet (3 implement, 3 adversarial review). Orchestrator ran every gate.
- Review cycles: 1 REQUEST_CHANGES (2 BLOCKING) → focused fix → APPROVED. Within cap.
- Infra failures: 1 (Docker Hub unreachable mid-CI, retried once, passed).
- Gate runs: 5 full uncached. Final: 1334 passed / 9 skipped.

## Friction (max 3)

1. **Three separate pre-existing failures surfaced as "my" CI failures**, costing three
   diagnosis rounds before any was attributable. Two were real (`main` un-deployable, a
   date-dependent test); one was infra. Nothing told the session `main` was already red.
2. **Path-based edit locks in `.claude/settings.local.json` do not work.** `Edit(apps/**)`
   blocks the edit tool only; two subagents wrote the same bytes via `node -e`, a third
   correctly refused. A lock that does not lock is worse than none because it is trusted.
3. **Two self-inflicted CI bugs on one file**, both passing local validation. The second
   (a workflow naming itself in `workflow_run`) silently disabled the guard entirely —
   the fix for silent failure failed silently.

## Proposals (max 3)

1. **DONE, owner-approved in session** — `core/PROCESS.md` §Release + §After merge: never
   push default branch; `Fixes #N` only for complete fixes; check the deploy run then
   exercise the capability; an unenforceable rule must be detectable; workflow config
   needs a live run. File held at its 150-line cap by compressing existing prose
   (contract, gear, merge floor, review, checkpoints) — no cap raise.
2. **`core/GREEN.md`** — add "a fixture tidier than reality" as a named failure mode.
   Two falsification tests passed for the wrong reason this session: one set two
   timestamps to the same object (a DB never produces that; production differed by 1 ms),
   one named a personal scope that did not exist so the vulnerable branch never ran.
   *Target:* `core/GREEN.md`, currently under cap. Not yet applied — owner gate.
3. **`START.md` is 103 lines against a 100-line cap** and was already over before this
   session. Flagging rather than trimming: it is the most-read file and edits there are
   high-blast-radius. *Target:* `START.md`. Owner decision.

## What actually caught the defects

Not the gate. Every real defect came from an adversarial reviewer briefed to break a named
guarantee, or from reading live production output:

- Root-tier agents could forge session attribution to any human (#197 says four non-human
  principals hold root) — found by review, missed by a test passing for the wrong reason.
- "Went quiet" on a session that never checked in — found by reading the real digest.
- A dead watchdog — found by looking at the run, not the file.

The gate was green for all three.
