# Session Retro: 2026-07-29 companyos-lexi org move + gate repairs
<!-- Data only — changes NOTHING until the owner approves. CAP: 1 page. -->

## Metrics
- Lane: ship (config/docs) + one deployment lane | Size/risk: medium, live prod | Elapsed: ~4h
- Workers spawned: `lexi-secure` × 3 (one died on launch, re-dispatched) | Review cycles: 0 | Gate runs: 6 CI + 3 local
- Infra failures: dispatch-launch × 1 (relative `-OutFile`); missing local clone files × 1 (42/49 absent)
- Token hotspots: the canonical gate. ~10 min per run, re-run per worker, and #172's flake forces reruns.

## What worked
- **Verifying the artifact, not the gate.** Every claim checked at its source: schema queried on the box
  rather than reading the migrate exit code; the prod GitHub token probed against the live API (422 vs 403)
  rather than trusting the health page; Syncthing's own index consulted to attribute a stray directory.
  This is what surfaced a *silently skipped migration* that four green deploys had hidden.
- **Cross-session corroboration beat single-session suspicion.** Two orchestrators hit the same flaky test
  on unrelated branches. Either alone reads as "probably my change"; together it is a filed, evidenced bug.
- **Fixing causes, not instances.** UTF-16 reports traced to the harness launching `powershell` (5.1's `>`
  writes UTF-16LE) and fixed by launching `pwsh`, rather than re-encoding files each time.

## Friction
- **Packet weight did not scale down.** Every packet demanded the full uncached suite plus a thorough
  evidence report, including for changes of a few lines. Workers complied — correctly — and each run cost
  ~10 minutes of gate. The discipline is right for migrations and auth; it is pure cost on a one-line edit.
- **A dispatch died silently on a relative path.** The dispatch script resolves `-Worktree` and `-Packet`
  to absolute paths but not `-OutFile`, so the child (cwd = worktree) read an empty prompt and exited.
  Zero-byte report, no error surfaced for ~20 minutes; two stray empty files in the worktree were briefly
  misread as progress.
- **The harness clone was 85% missing on disk** (42 of 49 files), while `START.md` still instructed agents
  to read `core/CONDUCT.md`, `core/GREEN.md`, `core/ORCHESTRATION.md`, `models/REGISTRY.md`,
  `templates/retro.md` — none present. Git had everything; `git restore .` fixed it. Any agent pointed here
  before that would have hit dead ends immediately and had no way to know why.

## Proposals (max 3)
| # | Target file | Exact change | If target at cap: what to remove |
|---|---|---|---|
| 1 | `core/ORCHESTRATION.md` | Add a "Proportion" section: packet weight and gate demands scale with card size; name the anti-pattern of demanding a full uncached suite and a thorough evidence report for a few-line change. | At 126/150 — room, no removal needed |
| 2 | `models/omniroute.md` | Record the dispatch quirk: pass ABSOLUTE paths for every dispatch argument including the output file; a relative one dies with an empty prompt and a zero-byte report. Watch commits, not file size, for liveness. | At 57/120 — room |
| 3 | `START.md` | Add a first-step integrity check: verify the referenced core files exist before following the protocol, and `git restore .` if not. | At 96/100 — replace two lines of the token-discipline block |

Owner decision: **ALL THREE APPROVED** in chat 2026-07-29 — "make sure you add your learnings about
the overkill for small line changes so the next agents pick that up", plus "i just want to point any
agent to this folder and they shd read that start file and do the right thing", which proposal 3
directly serves. Applied in the same commit:

- `core/ORCHESTRATION.md` → new "Proportion" section. 150/150, at cap.
- `models/omniroute.md` → new "Dispatch quirks (2026-07-29)". 74/120.
- `START.md` → new step 0, the integrity check. 99/100.
