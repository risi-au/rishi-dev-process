# PROCESS — loop, triage, risk, gates, review
<!-- HARD CAP: 150 lines. To add, remove (core/SELF-IMPROVE.md). -->

## The loop (TRIP)

```
INTAKE -> TRIAGE -> PLAN (if required) -> IMPLEMENT -> GATE -> REVIEW -> RELEASE
```

## Triage: size

| Class | Criteria | Who codes | Plan |
|---|---|---|---|
| Trivial | Few lines, obvious, low risk, one area | Orchestrator self-implements | None |
| Standard | Multi-file or behavior change | Dispatched (self-implement only with owner waiver in the Session Brief) | plan-lite |
| Heavy | Schema/API/core, multi-module, security-touching | Dispatched per Session Brief | plan-full |

## Triage: risk profile (independent of size)

| Profile | Triggers | Extra requirements |
|---|---|---|
| R0 low | No auth/deploy/data/concurrency/destructive triggers | None. Checkpoint: 30 min |
| R1 routine | Small reversible feature/app; ~≤10 files / ~≤500 lines | 1 fresh reviewer. Checkpoint: 2 h |
| R2 elevated | Auth, secrets, deployment, migrations, cross-user perms, destructive potential, concurrency, novel architecture | Product contract + plan-full + owner plan approval + fresh final reviewer. Checkpoint: the plan's estimate |

List the exact triggers in the plan. If new triggers appear mid-work, promote the
profile and re-approve the plan — never silently expand the process.

## Plan rules

- Every non-trivial task anchors to a **one-page product contract**: purpose,
  in-scope, exclusions, safety invariants, acceptance checks, deployment boundary
  (`templates/product-contract.md`). **Do not duplicate what already exists** — where
  the GitHub issue already states scope, exclusions and acceptance criteria, that
  issue IS the contract: link it and note any delta. Writing a second copy is pure
  token cost and immediately drifts. For Standard / R0–R1 the contract is the
  CONTRACT section at the top of plan-lite — one file, not two.
- plan-lite (`templates/plan-lite.md`): hard cap 2 pages. Default for Standard / R0–R1.
- plan-full (`templates/plan-full.md`): Heavy or R2 only. Owner approves before code.
- **R2-small exception**: a Standard-sized R2 task (≤~3 files, no migration, deploy,
  or novel architecture) MAY use plan-lite-with-contract + Session Brief approval
  instead of plan-full + separate contract — *if* the fresh final reviewer and explicit
  owner approval still hold. Heavy, migration, or deploy R2 always needs plan-full.
- Plans are FILES in the project's plan location, not chat messages.
- Plans are never pasted to workers — workers get packets (`core/ORCHESTRATION.md`).

## Gate

- Gate = the project's configured checks (typecheck, lint, tests — see project docs;
  a new project sets these up before product code).
- Run affected tests during implementation; run the FULL gate on every release candidate.
- **A green gate does not mean the guarantee holds.** Across two consecutive R2 batches,
  7 of 9 real defects had passing tests AND a green gate; only an adversarial review found
  them. `core/GREEN.md` is the required reading on why, and on falsification-first tests
  (write the failing test in its own card, before the implementer, which may not edit it).
- **The orchestrator runs the real gate.** A worker's "green" is scoped to what it
  actually ran, which is never the full suite (see `models/cline.md` — its command
  runner caps at ~30s). This caught failures workers could not see three times in one
  session. Never accept a worker's gate claim as the gate.
- Record the gate where it will actually be read: **CI + the PR body is enough** for a
  normal release candidate. Write the gate-receipt file
  (`templates/gate-receipt.md`) only when reuse matters — an expensive or
  non-reproducible gate, or a handoff where CI cannot run it. Reuse requires the file;
  routine work does not.
- Reuse: a receipt stays valid while nothing it fingerprints changed. Handing work to
  another agent is NOT a reason to rerun an unchanged gate.
- Docs-only diffs skip the source gate unless they change commands, config, or
  operational instructions. Deployment smoke tests are never cached.

## Review state machine

```
FULL_REVIEW -> FOCUSED_FIX -> FOCUSED_REREVIEW -> release
```

1. **FULL_REVIEW** — one fresh session (different vendor than the implementer when
   possible) reviews the reviewer packet + the complete release diff. ALL findings
   are batched in one response, each with a stable ID and a status:
   `BLOCKING | NON_BLOCKING | QUESTION`.
   **Review the guarantee, not just the plan.** Conformance review ("does the diff
   match what we said?") is necessary but not sufficient: on companyos-lexi Shot 8 all
   four real defects *did* match the plan and would have passed it — including a route
   by which an imported bundle could mark knowledge human-confirmed that no human
   confirmed, the exact failure the batch existed to prevent. For anything carrying a
   correctness, safety, or privacy guarantee, point the reviewer at **breaking that
   guarantee**, name the specific ways you suspect it could fail, and tell it that
   "the batch is sound" is an acceptable answer and padding with style nits is not.
2. **FOCUSED_FIX** — one correction pass fixes all BLOCKING findings together.
   Rerun the full gate on the corrected candidate; update the receipt.
3. **FOCUSED_REREVIEW** — checks ONLY: the finding IDs, files changed by the fixes,
   their direct dependency/security halo, and the updated gate receipt. It does not
   hunt new scope. Broad re-reviews after every fix are what turned a 40-minute
   build into 7 hours — banned.

- A new FULL_REVIEW is required only if fixes changed architecture, public
  interfaces, auth, deployment behavior, dependencies, or >~25% of the reviewed diff.
- After TWO `REQUEST_CHANGES` cycles: STOP. Re-plan or ask the owner for a
  scope/risk decision. Review is a release gate, not an unlimited audit.
- Verdicts: `APPROVED | REQUEST_CHANGES | NEEDS_REWORK` (approach wrong → re-plan).
- Approval applies only to the reviewed diff. New commits invalidate it.

## Checkpoints and stop conditions

- Crossing a time checkpoint → short status to the owner: what expanded, what
  remains, revised approach. It does not auto-cancel the work.
- Infra failures (sandbox, auth, quota, cancelled/silent worker): retry the same
  mechanism ONCE, then swap worker/lane or stop and ask. Log these as
  infrastructure events in the retro — never as review cycles.
- One-strike remote rule: one failed SSH/connection attempt → stop; don't guess
  credentials or host state.

## Finish report (required whenever claiming done)

```
Files changed:
- path — one line why
Deviations from plan: <none | rationale>
Left undone: <none | list>
Gate: <receipt summary, e.g. lint ok | typecheck ok | tests 42 passed>
```

## Release

- Branch per task: `task/<slug>` or `fix/<slug>` off the default branch.
- PR links the issue (`Fixes #N`). Never push main directly.
- **The orchestrator merges its own PRs** (owner decision 2026-07-25) when ALL of the
  merge floor holds:
  1. the FULL gate is green, run **uncached** (never trust a cached lint/typecheck);
  2. CI on the PR is green — a green *local* gate is not a substitute;
  3. an independent adversarial review found nothing blocking;
  4. the diff is entirely in scope for the task.
  Any one failing → the PR waits for the owner, with a written reason in the PR body.
- Deploy is a separate approval whenever merging is not itself the deploy. Where merge
  auto-deploys, say so plainly before merging.
