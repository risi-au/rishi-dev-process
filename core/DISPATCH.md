# DISPATCH — who does the work, and who pays for it
<!-- HARD CAP: 95 lines. To add, remove (core/SELF-IMPROVE.md). -->

**The owner chooses the dispatch mode every session, before any work starts.** The agent
proposes; the owner checks their limits and approves. This is not the agent's call: the
modes differ mainly in *whose quota they spend*, and only the owner knows what is left.

Wrong mode is expensive in a way no gate catches — a batch run on the wrong quota can
exhaust the owner's capacity mid-session and strand finished-but-unmerged work.

## The four modes

### A — Native subagents (in-session)
The session agent spawns Claude subagents (Haiku / Sonnet / Opus, tier per task) inside
the same account and session.

- **Pays:** the owner's Claude quota. This is the entire trade-off.
- **For:** no external dependency, nothing to health-check, fastest start, shared session
  context, cheap tiering per card.
- **Against:** every worker token is the owner's own. 2026-08-04 ran a full batch this way
  and the owner's next instruction was *"claude's limit is fresh and i dont want to waste
  it"* — that is the failure mode: not breakage, just a budget spent where it need not be.
- **Trap (2026-08-04):** a worker returned *"delegated to a background agent"* with zero
  commits. Packets MUST say **"do the work YOURSELF — do NOT spawn subagents"**, and
  supervision is by disk (`git status --porcelain` per worktree), never by report.
- **Best for:** small contained fixes; anything urgent; when B is refused or unavailable.

### B — Codex headless (Claude orchestrates, Codex implements)
`codex exec` per card, one worktree each. Exact command and load-bearing flags:
`models/codex.md`.

- **Pays:** Codex quota, not Claude's. The reason this mode exists.
- **For:** handles genuinely hard work — 2026-08-05/06 `gpt-5.6-luna@high` reworked a
  concurrency lock, rewrote a race test onto a driver-neutral harness, and reasoned
  correctly about an OAuth authentication boundary.
- **Against:** the permission classifier refuses dispatches **non-deterministically**.
  2026-08-04 it refused the identical command three times and issue #302 was never
  dispatched at all; 2026-08-05/06 it allowed 11 of 11. **If it refuses, tell the owner
  and move on — do not hunt for workarounds.**
- **Trap:** `--agent luna` does not exist. luna is a *model* (`-m gpt-5.6-luna`); asking
  for it as an agent silently gives you a Claude worker on the owner's quota — mode A
  while believing you are in mode B.
- **Best for:** batches, and anything substantial when Claude quota matters.

### C — OmniRoute
`lexi-*` combos, a different provider from the implementer. Health check first:
`curl -s -m 8 -o /dev/null -w "%{http_code}" http://localhost:20128/api/v1/models` → 200.

- **Pays:** whatever the routed provider costs; cheap.
- **For:** a genuinely independent perspective, because it is not the same vendor.
- **Against:** as a reviewer it runs roughly **5 real : 10 false**. 2026-08-06 it returned
  3 findings on a diff, all false, **and missed the real defect in that same diff** — a
  deleted authorization check. It has also died silently mid-session before.
- **Best for:** a cheap extra opinion. **Never the review of record** for auth,
  permissions, deletion or a security boundary (`models/omniroute.md`).

### D — Solo (the orchestrator does the work)
- **Pays:** session tokens only; no packets, no dispatch overhead, full context.
- **Against:** it collapses the implementer/reviewer separation that `core/GREEN.md`
  rests on. The orchestrator marking its own homework is the conflict of interest the
  whole harness is built to remove.
- **Best for:** trivial edits, config, docs, and orchestration glue. **Not product code
  above R1** — if it feels borderline, it is mode A or B.

## Choosing (propose this, do not decide it)

| Situation | Propose |
|---|---|
| Trivial / docs / config | D |
| One contained fix, Claude quota healthy | A |
| Several independent cards, expensive deploy | B |
| Claude quota low or being conserved | B |
| Anything touching auth, permissions, deletion, migrations | B or A **plus** a fresh reviewer (never C alone) |
| B refused by the classifier | Say so, propose A, let the owner decide |

Modes mix: B for implementation and a fresh reviewer for review is the strongest
combination measured so far (2026-08-06: one reviewer run, six real findings, zero false).

## The Dispatch Consult (blocking, part of the Session Brief)

Send with the Session Brief (`START.md` step 4) and **wait**:

```
Task:            <one line>
Lane / size:     <lane> / <R0-R3>
Proposed mode:   <A|B|C|D> — <one line why>
Workers:         <n> × <model@effort>, one per card
Review:          <fresh reviewer | C as extra opinion | none, and why>
Quota impact:    <whose budget this spends>
If refused:      <the fallback, so the owner approves it in the same breath>
```

The owner replies with the mode. Do not dispatch before that reply. Record the chosen
mode and any mid-session change in the retro — that is how this file stays accurate.
