# LANE: Feature — new capability in an existing project
<!-- CAP: 80 lines -->

Prereqs: START.md + core read; project overlay (AGENTS.md / ONBOARDING.md) read.

## Steps

1. **Intake.** Get the feature request (GitHub issue preferred; else the owner's
   words). If requirements are vague, GRILL before coding: pointed questions —
   goal, users, edge cases, non-goals — until you can write acceptance criteria.
2. **Triage** size + risk (`core/PROCESS.md`). Confirm class with the owner if
   not obvious.
3. **Contract + plan.**
   - Standard: product contract + plan-lite, stored in the project's plan location
     (its `docs/tasks/` or `docs/plans/` if present).
   - Heavy or R2: plan-full; owner approves before code.
4. **Model Consult** (`core/ORCHESTRATION.md`) if dispatching.
5. **Branch** `task/<slug>` off the default branch (worktree if the platform
   supports it).
6. **Implement** — self (borderline-trivial) or dispatch an implementer packet.
   Affected tests run during implementation.
7. **Gate** the release candidate; write the gate receipt.
8. **Review** per the state machine: FULL_REVIEW (fresh vendor) → FOCUSED_FIX →
   FOCUSED_REREVIEW. Two-cycle stop.
9. **Release.** Finish report → owner approves commit → PR (`Fixes #N`) → owner merges.
10. **Retro** (`core/SELF-IMPROVE.md`).

## Lane rules

- **Verify the issue's stated starting point against the tree before writing any packet.**
  If it describes a type, capability or migration state, confirm that exists. Grep for
  duplicate copies of any type the card will change. (2026-07-26: an issue claimed a shape
  had been frozen by earlier work; it had not, five divergent copies existed, and the
  duplication *was* the bug. A packet written from the issue text would have been nonsense.)
- Acceptance criteria exist before implementation starts. No criteria, no code.
- Scope creep discovered mid-work: note it, finish the contracted scope, file the
  extra as a new issue. Don't absorb it.
- UI/UX decisions not covered by the contract are owner decisions — ask.
