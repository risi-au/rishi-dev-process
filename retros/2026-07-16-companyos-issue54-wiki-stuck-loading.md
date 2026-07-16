# Session Retro: 2026-07-16 companyos issue #54 wiki stuck loading

<!-- Data only — changes NOTHING until the owner approves. CAP: 1 page. -->

## Metrics
- Lane: bugfix | Size/risk: Standard / R1 | Elapsed: ~1h
- Workers spawned: codex-rescue@default × 1 (reviewer only; orchestrator self-implemented) | Review cycles: 1 (FULL_REVIEW → FOCUSED_FIX → FOCUSED_REREVIEW → APPROVED) | Gate runs: 2 (full)
- Infra failures: none
- Token hotspots (estimate): none significant; issue screenshot download + one web search grounded the root cause cheaply

## What worked
- Reading the issue's screenshot as evidence (list rendered, only doc pane stuck) narrowed the fault to one state flag before any code changes.
- Root cause matched a known upstream bug (vercel/next.js#74246 — server action never settles when a navigation supersedes it), found with one targeted web search.
- Fresh codex reviewer caught 3 real findings (retry loop, stale tab href, undismissable error) that the implementer missed — R1's "1 fresh reviewer" rule earned its cost.

## Friction
- Staging-timing bug not locally reproducible; lane says "can't reproduce → investigation lane", but screenshot + code + upstream-bug evidence were sufficient. The lane text forces an awkward call.
- No React component test infra (no RTL) in companyos → regression test could only cover extracted pure guards, not the render behavior.
- Owner not present: dispatch requires Model Consult (blocked), so orchestrator self-implemented a Standard task; commit/push approval must be batched at end of session.

## Proposals (max 3)
| # | Target file | Exact change | If target at cap: what to remove |
|---|---|---|---|
| 1 | lanes/bugfix.md | Step 2 "Reproduce": add "…or establish the failure mechanism from artifacts (screenshot/log/known upstream bug) when the repro is timing/environment-bound; state this substitution in the finish report." | Lane at 31/80 lines — no removal needed |

Owner decision (2026-07-17): Proposal 1 NOT APPROVED for now (repro-substitution
clause offered as part of a batch; owner did not select it). May be re-proposed
with more evidence from future sessions.
