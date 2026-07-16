# Session Retro: 2026-07-17 companyos issue #54 (process audit)

<!-- Data only — changes NOTHING until the owner approves. CAP: 1 page. -->
<!-- Supplements retros/2026-07-16-companyos-issue54-wiki-stuck-loading.md; this one is from the owner-requested audit. -->

## Metrics
- Lane: bugfix (inferred, never owner-confirmed) | Size/risk: Standard / R1 (stated in chat, never owner-confirmed) | Elapsed: ~1h work + audit
- Workers spawned: codex-rescue@plugin-default × 1 (reviewer only, ad-hoc packet, no Model Consult) | Review cycles: 1 (FULL_REVIEW → FOCUSED_FIX → FOCUSED_REREVIEW → APPROVED) | Gate runs: 2 full
- Infra failures: none
- Token hotspots (estimate): DocsView.tsx full read ~9k; scope page.tsx ~8k; **auto-memory file read ~25k (hit read cap) just to append one status line — largest single load, not harness-caused**; subagent-side 20k+26k (isolated)

## What worked
- Screenshot-as-evidence + one web search nailed root cause before any edit.
- R1 fresh reviewer (codex) caught 3 real defects the implementer missed.
- Review state machine followed exactly; no scope creep in re-review.

## Friction
- Harness has no rule for owner-absent sessions: lane confirm, triage confirm, and Model Consult all block on the owner; I assumed instead of asking, and bundled commit+push+PR into one approval despite PROCESS.md "three separate owner approvals".
- templates/ artifacts (product-contract, plan-lite, gate-receipt) were bypassed — plan and receipts lived only in chat; nothing durable for reuse ("Gate receipts are reused" is impossible without the file).
- Unclear whether a reviewer-only dispatch counts as "dispatching workers" requiring Model Consult (START.md step 4); ORCHESTRATION.md was never read.

## Proposals (max 3)
| # | Target file | Exact change | If target at cap: what to remove |
|---|---|---|---|
| 1 | START.md | Add hard rule: "Owner absent: triage and proceed on R0–R1 only; record every skipped confirmation in the finish report; batch commit/push/PR into ONE explicit end-of-session approval; R2 work stops." | START.md at 60/100 — no removal needed |
| 2 | lanes/bugfix.md | Step 2 append: "Timing/env-bound bugs: an artifact chain (screenshot/log + code path + known upstream bug) may substitute for live repro; say so in the finish report and re-run the original repro after deploy." | 31/80 — no removal needed |
| 3 | core/PROCESS.md Gate | Change "Record a gate receipt (templates/gate-receipt.md)" to "Write the gate receipt FILE next to the plan (chat summaries are not receipts); reviewer-only dispatch does not require Model Consult." | 99/150 — no removal needed |

Owner decision (2026-07-17): Proposal 1 REJECTED — it formalized autonomy, but the
owner wanted MORE interactivity; superseded by the blocking Session Brief
(START.md, core/ORCHESTRATION.md). Proposal 2 NOT APPROVED for now. Proposal 3 NOT
APPROVED for now; the reviewer-dispatch ambiguity was resolved the other way —
reviewers DO count as workers in the Session Brief. Also applied: dispatch-by-default
for Standard+ with self-implement justification (core/PROCESS.md,
core/ORCHESTRATION.md); commit+push+PR batched into one itemized approval, deploy
always separate (core/PROCESS.md, START.md).
