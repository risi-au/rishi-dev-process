# DISPATCH LEDGER — <project> <task> <YYYY-MM-DD>

<!-- Append a row AT DISPATCH TIME, not at retro time. See core/ORCHESTRATION.md. -->
<!-- Purpose: make "did the orchestrator actually delegate?" and "where did tokens go?" -->
<!-- checkable facts rather than end-of-session recollection. Adapted from fusion-audit. -->

| # | Card | Role | Combo @ effort | Timeout | Output file | Verdict | Files touched | Notes |
|---|---|---|---|---|---|---|---|---|
| 1 | | implementer \| falsification \| reviewer \| scout | | | | pending → landed \| rejected \| died | | |

Fill `#`, `Card`, `Role`, `Combo`, `Timeout`, `Output file` **before** the dispatch returns.
Fill `Verdict`, `Files touched`, `Notes` when you judge it against the working tree.

## Orchestrator's own edits (must stay empty on R1+ without an owner waiver)

| # | File | Why a packet was not worth it | Independently verified by |
|---|---|---|---|

Any row here is a deviation to name in the finish report and the retro. If the file is a
source file and no owner waiver was given in the Session Brief, it is a rule break — log
it honestly rather than omitting it.

## Totals (for the retro)

- Dispatches: <n> (implementer <n>, falsification <n>, reviewer <n>, scout <n>)
- Died / timed out: <n> — classify each: product | test | environment | worker-session | access
- Orchestrator source edits: <n> (target: 0)
- Full uncached gate runs: <n> — of which red: <n>
