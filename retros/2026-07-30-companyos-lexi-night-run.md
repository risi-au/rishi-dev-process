# Session Retro: 2026-07-30 companyos-lexi night run (autonomous orchestrator)
<!-- Data only — changes NOTHING until the owner approves. CAP: 1 page. -->

## Metrics

- Lane: `ship.md` | Size/risk: 6 cards, three of them trust boundaries | Elapsed: overnight, single session
- Workers spawned: **12 dispatches across 6 cards** —
  `lexi-secure` ×7, `lexi-implement` ×3, `lexi-scout-grok` ×1, `lexi-review` ×1
- Review cycles: **1** (#176, the standing auth-boundary exception) → REQUEST_CHANGES → fix
- Gate runs: 6 full CI runs; 1 red (the composition break), 1 red-then-green after integration
- **Infra failures: 3** — `worker-session` ×2 (silent OOM kill; `API Error: Response stalled
  mid-stream`), plus 1 wasted pass caused by dispatching onto a stale branch (my error, classified
  `worker-session` only because the worker inherited it)
- Orchestrator source edits: **0**. Two integration actions taken directly (a `git revert` of a
  worker's stale edit, and `git merge origin/main` into a branch) — both integration, not
  implementation, but logged here rather than left implicit.
- Blocking checkpoint: **waived by the owner's kickoff** ("do not block on him"). Decisions taken
  unilaterally and stated: #135's trust gate, #134's scope cut, #183's fix shape.

**Shipped:** #180, #183, #176, #178, #135 merged, deployed and verified live. #134 in flight.
**Filed:** #183 (from the scout), #186. **Docs PRs:** #182, #187, #190.

## What worked

- **Incremental push discipline is the single highest-value packet line, again.** Two workers died
  mid-card; in both cases every pushed commit survived and only unpushed work was redone. The #176
  backend half was recovered intact from a worker that produced a 0-byte report.
- **Verifying the artifact caught three things a green signal would have hidden:** the docs named a
  *retired* container stack that is still running and answers queries with stale data; a green
  `release` job is not a deploy (it is one of three chained jobs); and migration `0051` needed
  `information_schema`, not the ledger, to confirm.
- **Naming the specific failure class in the review packet produced the only blocking finding** — the
  dynamic-grant test exercised `resolveAccess` rather than the MCP tool path. Same pattern as #93/#100:
  generic "review this" would not have found it.

## Friction

- **A branch gate and CI answer different questions, and nothing said so.** `feat/176` was cut from
  `9b8a00c` and never had `main` merged in. CI tests the *merge*, so it failed on a file the branch
  did not contain; a worker then "fixed" a stale copy. Cost: one wasted dispatch plus diagnosis.
- **Three concurrent workers each running the full suite OOM-killed the machine**
  (`Fatal process out of memory: Zone`). Killed one worker outright and forced every later packet to
  defer the full suite to CI. My scheduling error, not a worker fault.
- **My #176 packet named the monotonic-timestamp trap but not "generate, never hand-edit".** The
  worker hand-wrote the journal and shipped no snapshot. `meta-chain.test.ts` would have caught it,
  but only after a full CI cycle. #173's packet had the right line; it did not travel.

## Proposals (max 3)

| # | Target file | Exact change | If target at cap: what to remove |
|---|---|---|---|
| 1 | `core/ORCHESTRATION.md` → "Failure handling" | Add: **"Merge `main` into any long-running branch before its gate means anything — and always before judging a red CI run on it. A worker's gate tests its branch; CI tests the merge. On a fast-moving `main` these diverge silently, and the divergence looks exactly like a bad fix."** | Drop the Orca paragraph under "Platforms" — platform-neutrality is already stated in the opening line |
| 2 | `core/ORCHESTRATION.md` → "Proportion", trap list | Add a third trap: **"A card that adds a migration must be told to generate metadata with the drizzle generate command and never hand-edit the journal or a snapshot. Require the report to confirm the snapshot exists, `prevId` chains, and the journal `when` strictly increases."** | Merge the two existing trap bullets' preambles; the examples carry the meaning |
| 3 | `core/ORCHESTRATION.md` → "Context packets" | Add: **"At most ONE concurrently-dispatched worker may be told to run the full test suite; give the others targeted tests and let CI run the full one. Three parallel suites OOM-killed a worker on 2026-07-30."** | Same as #1 |

Owner decision: PENDING
