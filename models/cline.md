# ADAPTER: cline (headless, via OmniRoute)

<!-- CAP: 120 lines. Quirks are dated; prune stale ones per core/SELF-IMPROVE.md. -->

Headless worker CLI. Routes through OmniRoute combos (`models/omniroute.md`), so the
combo name — not a vendor model id — is what you pass. Proven at scale on
companyos-lexi Shot 8 (2026-07-25): ~15 dispatched cards, one PR per card group.

## Preflight (before first dispatch of a session)

```
curl -s -m 8 -o /dev/null -w "%{http_code}\n" http://localhost:20128/api/v1/models
```

`200` = OmniRoute up. **`000` = down** — cline will die instantly with
`Cannot connect to API ... ConnectionRefused`, which looks exactly like a bad packet.
Re-run this check whenever a worker dies with a connection error, not only once per
session (2026-07-25: OmniRoute went down mid-card; the next dispatch failed in
seconds and the cause was not obvious from the worker output).

## Spawn — headless implementer

```
cline -P "openai-compatible" -m "<combo>" --thinking high --auto-approve true \
  -c "<project-abs-path>" --timeout <seconds> "<packet prompt>"
```

- Always dispatch through the harness `run_in_background`; runs are minutes-long.
- cline **buffers all output to the end** — an in-progress task's output file reads
  0 bytes. That is normal, not a hang.
- Do NOT pipe the invocation through `tail -N`: it truncates the worker's final
  report, which is where its self-reported gaps and baselines live (2026-07-25 —
  lost an eval baseline report this way).

## Combos (OmniRoute) — see models/omniroute.md for the live list

`sprint-implement-r2` (default implementer for contract/schema work) ·
`sprint-implement` · `sprint-mechanical-cheap` (one-line/mechanical fixes) ·
`sprint-review-r2` (adversarial reviewer) · `sprint-rescue` ·
`sprint-scout-cheap` · `sprint-conformance-cheap`

## Quirks (dated)

- **2026-07-25 — 30-second command cap.** cline's internal command runner kills any
  shell command at ~30s, so a worker can NEVER complete a full `pnpm test` on a
  medium repo. Tell it in the packet: say so explicitly rather than claim green, and
  run targeted tests **from the repository root**. **The orchestrator runs the real
  gate** — this caught failures workers could not see three times in one session.
- **2026-07-25 — oversized single writes kill the run.** A card whose deliverable was
  a ~400-line file died mid-write with `error: terminated`. Every packet must say:
  *"write in small incremental edits, never more than ~120 lines per edit."* Every
  card that carried that line landed first time. Split any deliverable over ~300 lines
  into separate cards.
- **2026-07-25 — the worker's own compliance sentence trips the constraint grep.**
  "No ... orchestration ... action was performed" matches a forbidden-token check.
  Read the match before treating it as a violation.
- **2026-07-25 — cwd matters for tests.** Running vitest from inside a package can
  resolve migration paths to the wrong checkout. Always instruct: from the repo root.
- Auto-approve is on, so the packet's **forbidden actions** list is the only guard.
  Always include the hard-constraint block (`templates/packet-implementer.md`).

## Verifying a cline worker

`exit 0` is not proof of work, and neither is its report. Per
`core/ORCHESTRATION.md`: confirm the diff touches only allowed files, run the gate
yourself, read the finish report against the success criteria. cline reports honestly
about what it ran — but "green" is scoped to what it ran, which is never the full suite.
