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

List the exact triggers in the plan. If new ones appear mid-work, promote the profile and
re-approve — never silently expand the process.

**Gear (Lexi only, `lanes/ship.md`):** Ship gear is the R0/R1 fast path for work with no
Guarantee-gear trigger (auth/permissions, data deletion/retention, citation/trust
semantics, MCP contract changes, DB migrations, anything carrying a named guarantee) —
one implementer card, review deferred to a mandatory shot-end batched pass. Guarantee
gear (the process below, in full) is mandatory for R2 and any card matching a trigger
regardless of size. When in doubt, Guarantee gear.

## Plan rules

- Every non-trivial task anchors to a **one-page product contract**: purpose, in-scope,
  exclusions, safety invariants, acceptance checks, deployment boundary
  (`templates/product-contract.md`). **Do not duplicate what exists** — where the issue
  states scope, exclusions and acceptance criteria, that issue IS the contract: link it,
  note the delta. For Standard / R0–R1 it is the CONTRACT section of plan-lite, not a
  second file.
- plan-lite (`templates/plan-lite.md`): hard cap 2 pages. Default for Standard / R0–R1.
- plan-full (`templates/plan-full.md`): Heavy or R2 only. Owner approves before code.
- **R2-small exception**: a Standard-sized R2 task (≤~3 files, no migration, deploy or
  novel architecture) MAY use plan-lite-with-contract + Session Brief approval — *if* the
  fresh final reviewer and explicit owner approval hold. Heavy, migration or deploy R2
  always needs plan-full.
- Plans are FILES in the project's plan location, never chat messages, and are never
  pasted to workers — workers get packets (`core/ORCHESTRATION.md`).

## Gate

- Gate = the project's configured checks (typecheck, lint, tests — see project docs; a
  new project sets these up before product code). Run affected tests during
  implementation; the FULL gate on every release candidate.
- **A green gate does not mean the guarantee holds.** Across two R2 batches, 7 of 9 real
  defects had passing tests AND a green gate; only adversarial review found them.
  `core/GREEN.md` is required reading on why, and on falsification-first tests (the
  failing test written in its own card, before the implementer, who may not edit it).
  **A fixture tidier than reality is how they pass anyway** — 2026-08-01: identical
  timestamps a database never produces; a scope that did not exist.
- **The orchestrator runs the real gate.** A worker's "green" covers only what it ran,
  never the full suite (`models/cline.md`: runner caps at ~30s). Caught failures workers
  could not see three times in one session. Never accept a worker's gate claim.
- Record the gate where it is read: **CI + the PR body is enough** normally. Write the
  receipt (`templates/gate-receipt.md`) only when reuse matters — an expensive or
  non-reproducible gate, or a handoff CI cannot run. It stays valid while nothing it
  fingerprints changed; a handoff alone is no reason to rerun.
- Docs-only diffs skip the source gate unless they change commands, config or
  operational instructions. Deployment smoke tests are never cached.

## Review state machine

```
FULL_REVIEW -> FOCUSED_FIX -> FOCUSED_REREVIEW -> release
```

1. **FULL_REVIEW** — one fresh session (different vendor to the implementer where
   possible) reviews the reviewer packet + the complete release diff. ALL findings batched
   in one response, each with a stable ID and `BLOCKING | NON_BLOCKING | QUESTION`.
   **Review the guarantee, not just the plan.** Conformance ("does the diff match what we
   said?") is necessary, not sufficient: on Shot 8 all four real defects matched the plan,
   including a route letting an imported bundle mark knowledge human-confirmed that no
   human had. For anything carrying a correctness, safety or privacy guarantee, point the
   reviewer at **breaking that guarantee**, name how you suspect it fails, and say "sound"
   is acceptable while style nits are not. 2026-08-01: caught a permissions hole in a diff
   whose own falsification test was passing for the wrong reason.
2. **FOCUSED_FIX** — one correction pass fixes all BLOCKING findings together; rerun the
   full gate on the corrected candidate and update the receipt.
3. **FOCUSED_REREVIEW** — checks ONLY the finding IDs, files the fixes changed, their
   direct dependency/security halo, and the updated receipt. It does not hunt new scope.
   Broad re-reviews after every fix turned a 40-minute build into 7 hours — banned.

- A new FULL_REVIEW is required only if fixes changed architecture, public interfaces,
  auth, deployment behavior, dependencies, or >~25% of the reviewed diff.
- After TWO `REQUEST_CHANGES` cycles: STOP. Re-plan or ask the owner for a scope/risk
  decision. Review is a release gate, not an unlimited audit.
- **A PARTIAL finding is not a new cycle** (owner decision 2026-07-26). Finishing an
  incomplete fix is not a send-back — but ONE attempt, verified by someone other than its
  author, then STOP and go to the owner. Without that cap "it's the same finding" never
  terminates, which is how it was used on 2026-07-26.
- Verdicts: `APPROVED | REQUEST_CHANGES | NEEDS_REWORK` (approach wrong → re-plan).
  Approval covers only the reviewed diff; new commits invalidate it.

## Checkpoints and stop conditions

- Crossing a time checkpoint → short status to the owner: what expanded, what remains,
  revised approach. It does not auto-cancel the work.
- Infra failures (sandbox, auth, quota, silent worker, a registry unreachable mid-CI):
  retry the same mechanism ONCE, then swap worker/lane or stop and ask. Log as
  infrastructure events in the retro — never as review cycles. One-strike remote rule:
  one failed SSH/connection attempt → stop; don't guess credentials or host state.

## Finish report (required whenever claiming done)

```
Files changed: path — one line why
Deviations from plan: <none | rationale>   Left undone: <none | list>
Gate: <receipt summary, e.g. lint ok | typecheck ok | tests 42 passed>
```

## Release

- Branch per task: `task/<slug>` or `fix/<slug>` off the default branch.
- **Never push the default branch directly** — always a branch and a PR, even for one
  line, even when you hold the merge decision. Where CI runs `on: pull_request` only, a
  direct push is never linted, typechecked, tested or image-built before it deploys.
- `Fixes #N` only for a complete fix; a partial one uses `Refs #N` and narrows the issue.
  Auto-closing a half-fixed ticket hides the remainder and the docs then disagree.
- **The orchestrator merges its own PRs** (owner decision 2026-07-25) when the whole merge
  floor holds: FULL gate green run **uncached**; CI on the PR green (a green *local* gate
  is no substitute); an independent adversarial review finding nothing blocking; the diff
  entirely in scope. Any one failing → the PR waits for the owner, reason in the PR body.
- Deploy is a separate approval unless merging IS the deploy; where it is, say so plainly
  before merging.

**After merge — green is not deployed, and deployed is not working** (owner decision
2026-08-01: where the platform cannot enforce, discipline and detection replace it):

- **Check the deploy run, not PR CI**, then **exercise the capability** — call it, load
  it, read the real output. Green proves only the jobs that *ran* passed, and the
  path-filtered job that skipped is often the one that mattered: a release failed for a
  day while every PR stayed green. Every defect found that day came from looking at the
  live thing, none from the gate.
- **An unenforceable rule must be detectable.** Convention without a detector is not a
  control: give it a check writing a durable signal the next agent trips over (an issue,
  not an email), and confirm the detector itself fires. For CI/workflow config, local
  validation proves syntax only — valid YAML ≠ valid workflow, so plan a live run.
