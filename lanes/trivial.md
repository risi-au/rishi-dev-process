# LANE: Trivial — few lines, obvious, low risk
<!-- CAP: 80 lines -->

For: typo fixes, config tweaks, doc edits, tiny obvious code changes, single-file
low-risk adjustments. If ANY risk trigger from `core/PROCESS.md` applies (auth,
data, deploy, destructive), it is not trivial — re-route.

## Steps

1. Confirm it's actually trivial: one area, few lines, no behavior ambiguity.
2. No plan, no dispatch, no Model Consult. The orchestrator implements directly.
3. Make the surgical change (`core/CONDUCT.md` still fully applies).
4. Gate: run the project's checks relevant to the change. Docs-only edits skip
   the source gate; verify rendering/links/commands instead.
5. Finish report (short form is fine) → owner approves commit → PR or branch push
   per project convention. Owner merges.
6. Retro only if something surprising happened; otherwise silent (no metrics
   needed for trivial work).

## Lane rules

- Time box ~30 minutes. If it's growing beyond that, it wasn't trivial — stop and
  re-triage with the owner.
- Batch related trivia into one PR rather than five.
- Zero scope expansion: this lane never touches "while I'm here" items.
