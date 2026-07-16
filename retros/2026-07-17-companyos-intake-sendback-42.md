# Retro — companyos, issue #42 (wizard send-back silent no-op) — 2026-07-17

Lane: bugfix (promoted to R2 mid-task). Outcome: PR #85 open (owner merges);
follow-up #84 filed (idempotent provisioning side effects).

## Metrics

- Elapsed: ~1h50m wall clock (incl. background gates/reviews).
- Workers spawned: codex gpt-5.6-terra@high x5 dispatch attempts, of which 3
  produced review verdicts (FULL_REVIEW, inline FULL_REVIEW on R2 diff, inline
  FOCUSED_REREVIEW) and 2 died on sandbox infra; 1 codex-rescue plugin agent
  (also sandbox-blocked). Implementer: orchestrator self-implement (owner waiver).
- Review cycles: 3 real (R1 -> owner scope expansion; R2-01/R2-02 -> fencing fix;
  rereview -> residual accepted by owner). Two-cycle stop rule triggered twice;
  both resolved by owner decision, not by grinding.
- Gate runs: 4 full (1 fail: duplicate test import; 3 green, final 407 tests) +
  targeted vitest/meta-chain runs.
- Infra failures (environment/worker-session, NOT review cycles): 4 — codex
  read-only sandbox could not read a packet file outside the workdir (1), then
  CreateProcessAsUserW error 5 spawning pwsh inside the worktree (2, persisted
  AFTER the documented icacls repair), plugin rescue lane hit the same wall (1).
- Token hotspots: reviewer packets with full inline diffs (necessary — see below);
  five review dispatch attempts.

## Friction points (max 3)

1. codex sandbox on this machine currently cannot spawn pwsh in Orca worktrees at
   all (error 5), even post-icacls; file-reading review dispatches die. The only
   working shape was a fully inline packet + "do not run any tools" instruction.
2. The bugfix lane had no cheap off-ramp when a reviewer finding implies an
   architecture change: two separate owner stops (scope expansion, then residual
   acceptance) were needed; both were correct but each cost a blocking round-trip.
3. Reviewer packet claimed "delta since your FULL_REVIEW" but embedded a vs-main
   diff; harmless here, but packet templates assume file access reviewers may not have.

## Proposals (max 3)

1. models/codex.md (Known quirks, dated 2026-07): add "Orca worktrees: sandbox
   pwsh spawn can fail with CreateProcessAsUserW error 5 even after the icacls
   repair; fall back to a fully inline packet in the prompt with an explicit
   'run no tools' instruction — review-only dispatches then work reliably."
   (File is under its 120-line cap.)
2. models/codex.md (same quirks block): mark the standalone icacls repair line as
   "necessary but not always sufficient (2026-07-17)" so the next session does not
   burn a retry proving it again.
3. No core/lanes change proposed: the two-cycle stop + owner decision worked as
   designed both times.

Owner decision (2026-07-17): Proposals 1 and 2 APPLIED to models/codex.md (quirk
lane). Graphify follow-up audit (same session) confirmed the graph was used
first, successfully, and saved ~10-20k exploration tokens, but was ABSENT from
the Orca worktree (gitignored) — fixed by adopting graphify into the harness
(models/graphify.md: absolute --graph path rule) plus a kickoff-prompt line in
project-setup/README.md and a packet rule in core/ORCHESTRATION.md.
