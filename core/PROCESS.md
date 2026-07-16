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

- Every non-trivial task starts with a **one-page product contract**
  (`templates/product-contract.md`): purpose, in-scope, exclusions, safety
  invariants, acceptance checks, deployment boundary. It is what every worker and
  reviewer anchors to.
- plan-lite (`templates/plan-lite.md`): hard cap 2 pages. Default for Standard / R0–R1.
- plan-full (`templates/plan-full.md`): Heavy or R2 only. Owner approves before code.
- Plans are never pasted to workers — workers get packets (`core/ORCHESTRATION.md`).

## Gate

- Gate = the project's configured checks (typecheck, lint, tests — see project docs;
  a new project sets these up before product code).
- Run affected tests during implementation; run the FULL gate on every release candidate.
- Record a **gate receipt** (`templates/gate-receipt.md`): revision, diff hash,
  commands, results, durations, tool versions.
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
- PR links the issue (`Fixes #N`). The owner merges. Never push main.
- Commit, push, and PR may be batched into ONE itemized approval. Deploy is
  always a separate approval.
