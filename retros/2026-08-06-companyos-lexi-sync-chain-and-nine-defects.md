# Session Retro: 2026-08-06 companyos-lexi + workspace-sync — the batch, then the sync chain

## Metrics
- Lane: batch, then deployment | Size/risk: R2 (auth, data-sync, two production deploys)
- Workers spawned: codex `gpt-5.6-luna@high` × 11 (5 batch cards, 6 reworks) + 1 fresh reviewer
- Review cycles: 2 (OmniRoute on the Lexi batch; fresh codex reviewer on workspace-sync)
- Gate runs: ~14 full/scoped | Infra failures: classifier refusals 0 (11/11 dispatches allowed)
- Token hotspots: **6 rework dispatches**, five of which trace to a missing sentence in the
  orchestrator's own packet rather than to worker error. Second hotspot: re-adjudicating each
  reworked branch from scratch.

## What worked
- **Fresh reviewer, clean checkout, named suspicions.** One run returned six real findings
  (`node_modules` blocked by a new scan, over-blocking independent repos, spelling-dependent URL
  matching, a stale user-facing README claim, an insufficient upgrade-race mitigation, and a
  pre-existing conflict-recovery path that could commit `.env`). Zero false positives survived
  checking. The packet named the two things the orchestrator was personally unsure about and told
  it to argue them.
- **Full uncached gate on the MERGED tree.** Caught a card breaking a test in a different package.
- **Adjudicating against the diff, not the report.** Caught a deleted authorization check.

## Friction
- **Five of six reworks were caused by the orchestrator's packet, not the worker.** Each was one
  absent instruction: gate scope narrower than the blast radius; no "verify in both environment
  states"; no "declare what you removed"; no "what legitimate cases does this now block"; no
  "trace the new config to the running process".
- **OmniRoute reviewed the batch diff containing a deleted auth check, returned 3 findings, all
  false, and missed it.** It cost verification time and contributed nothing. The fresh codex
  reviewer on a comparable diff returned 6-for-6 real.
- **The prescribed cure for scope blindness silently measured nothing.** `turbo run test --force`
  does not forward `DATABASE_URL` (turbo 2 strict env mode), so every database-gated test skipped
  while turbo reported `30 successful, 30 total`. Both race tests the batch existed to add skipped
  under it. `core/GREEN.md` names the orchestrator's full gate as the non-negotiable cure for S —
  and that gate was the thing lying.

## Proposals (max 3)
| # | Target file | Exact change | If target at cap: what to remove |
|---|---|---|---|
| 1 | `templates/packet-implementer.md` | Add a required `Removed or weakened:` RETURN field (every check/guard/validation/test removed, and why; `none` is a valid answer). Add to VERIFY WITH: gate must cover the codegraph blast radius, not the card's package; and any command whose behaviour depends on an env var or external service must be run in BOTH states with both exit codes reported. | No cap on templates/ |
| 2 | `templates/packet-reviewer.md` | Add review axis **(f) REMOVALS** — "what does this diff DELETE? A removed check leaves no trace in a report that describes only additions." Strengthen the shell paragraph: re-run the gate yourself, do not accept the receipt's numbers. | No cap on templates/ |
| 3 | `core/GREEN.md` | Add fifth failure mode **R — Removal invisibility** to the table, and amend S's cure so the orchestrator's gate must itself be shown to have measured something (env-dependent gates can skip silently while reporting success). | At 84/90. Costs ~5 lines; trim the Evidence section's per-category prose, keeping the counts. |

Also proposed for `models/omniroute.md` (not core, no gate needed per SELF-IMPROVE §3): record the
2026-08-06 result — 3 findings, 3 false, 1 real defect present in the diff and missed — and the
standing recommendation not to rely on it as the review of record for auth/data changes.

Owner decision: APPROVED in chat 2026-08-06 (all three proposals + the models/ note).

## Second request, same session — the Dispatch Consult (owner-initiated)

Owner, 2026-08-06: *"for every session, I want the agent to ask me which method to follow… the agent
at the beginning of the session has to propose which way to go forward, and based on that, we
proceed… I will check the limits and give my approval."*

The harness had a Model Consult (which model at which effort) but **no Dispatch Consult** (which
*mechanism*, and whose quota it spends). Four modes had been used across sessions with no written
comparison, so each session re-derived the trade-off from scratch — and 2026-08-04 spent a batch on
the owner's Claude quota that the owner would have routed elsewhere had he been asked.

Applied (owner-approved in the same chat):

| Target | Change |
|---|---|
| `core/DISPATCH.md` | **New file** (94/95). Four modes — native subagents, Codex headless, OmniRoute, solo — each with what it pays for, what it is good at, its measured failure modes, and its traps. Selection table. The blocking Dispatch Consult format. |
| `START.md` | Step 4 now requires the Dispatch Consult alongside the Model Consult, and states plainly that **the mode is the owner's call**, because only the owner knows what quota is left. |
| `core/SELF-IMPROVE.md` | Cap row for `core/DISPATCH.md`; **START.md cap raised 100 → 120.** |

**On that cap raise, honestly:** START.md was already **117/100** before this session — over cap for
weeks while the file itself instructs agents to respect caps. A cap being quietly ignored is worse
than an honest number. It was trimmed to 111 *while* gaining the Dispatch Consult, and the cap moved
to 120 to match reality. Flagged rather than done silently.

**Still over cap, deliberately not touched:** `core/ORCHESTRATION.md` at **156/150**. Not this
session's subject, and trimming a file mid-session that I had not read in full is how good content
gets lost. Next retro should either trim it or raise it to 160 — but decide, rather than leave a
stated rule being broken by the harness's own files.

Owner decision (dispatch consult): APPROVED in chat 2026-08-06.
