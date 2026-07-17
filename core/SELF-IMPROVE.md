# SELF-IMPROVE — gated evolution of this harness
<!-- HARD CAP: 60 lines. -->

The old method — append learnings to process docs every session — bloated the
process until it failed on a new project. Replacement: retros are quarantined
data, proposals are capped, the owner gates every change, core files have hard caps.

## End-of-session retro (every session, all lanes except silent-trivial)

Write `retros/YYYY-MM-DD-<project>-<slug>.md` from `templates/retro.md`:
- Metrics: elapsed time, workers spawned (model@effort), review cycles, gate runs,
  infra failures by category, estimated token hotspots.
- Max 3 friction points, max 3 proposals. Each proposal names its exact target file
  and — if that file is at its cap — what text it would replace.

**A retro is data. It changes nothing by itself.**

## Change gate

1. Proposals live only in `retros/` until the owner explicitly approves them.
2. Approved `core/` or `lanes/` changes land as a commit the owner reviews (or a PR).
3. Machine/CLI quirks go to `models/<worker>.md` — never to core. Date every quirk.

## Size caps (hard)

| File | Cap |
|---|---|
| START.md | 100 lines |
| core/PROCESS.md, core/ORCHESTRATION.md | 150 lines each |
| core/CONDUCT.md | 80 lines |
| core/SELF-IMPROVE.md | 60 lines |
| each lanes/*.md | 80 lines |
| each models/*.md | 120 lines — prune, don't grow |

At cap, adding a line means removing a line. If a cap consistently hurts, the retro
proposal is "raise the cap" — owner-approved like any other change.

## Session close ritual (the work agent IS the auditor — no second agent)

In order, before signing off: (1) finish report; (2) retro file — its friction
section MUST list every harness rule you skipped or bent this session, honestly;
(3) present to the owner in chat: a 3-line metrics summary + the proposals;
(4) proposals the owner approves in chat: apply those exact edits to the harness
clone (respect caps), set the retro's owner-decision line, commit + push the
harness repo. The harness is the ONLY repo where pushing main is allowed, and
only for owner-approved changes in that same chat.

## Pruning

- models/ quirks unconfirmed for 90+ days: propose deletion in the next retro.
- Every ~10 retros the owner may request a compaction session: one agent reads all
  pending retros, extracts patterns, proposes core edits, and archives processed
  retros to `retros/archive/`.
