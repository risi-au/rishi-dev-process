# rishi-dev-process — Master Dev Harness

The canonical, platform-independent process for all dev work (any project, any
model). Agents read `START.md` first, then work in the target project folder.

## Quickstart (human)

Start any agent session (Claude Code, Codex, Grok, inside Orca or not) with:

```
Read <harness path or repo URL>/START.md and follow it.
Lane: <lanes/*.md, or "triage it">. Project: <path>. Task: <description or issue #N>.
```

## Map

| Folder | What | Change policy |
|---|---|---|
| `START.md` | Single agent entry point | Owner-gated, 100-line cap |
| `core/` | Stable process: loop, orchestration, conduct, self-improvement | Owner-gated, hard line caps |
| `lanes/` | Per-task-type protocols (feature, bugfix, new-project, ...) | Owner-gated, 80-line caps |
| `models/` | Worker registry + per-CLI adapters (spawn, preflight, quirks) | Churn allowed; quirks dated + pruned |
| `templates/` | Contract, plans, context packets, gate receipt, retro | Owner-gated |
| `retros/` | Session retros land here; data only | Append-only; archived on compaction |
| `project-setup/` | How to point a project at this harness + new-machine bootstrap (`new-machine.md`) | Owner-gated |
| `history/` | Source reports that shaped this harness | Frozen |

## Design principles

1. **Process is king** — Orca, Claude Code, Codex are interchangeable runtimes.
2. **Orchestrator = whatever model the owner starts the session with**; worker
   models are chosen per session via the Model Consult (quota-aware, owner-approved).
3. **Token economy is structural**: context packets, one-full-review-then-deltas,
   gate receipts, preflight before expensive dispatch.
4. **Self-improving without bloat**: retros are quarantined data; core files have
   hard size caps; every change is owner-gated.
