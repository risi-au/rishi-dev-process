# MODEL REGISTRY — the worker pool
<!-- Owner-maintained. Orchestrator: read before every Model Consult. -->
<!-- Status/quota notes are hints, not truth — the consult confirms with the owner. -->

| Worker | CLI / access | Status (2026-07-16) | Tier fit | Typical roles | Adapter |
|---|---|---|---|---|---|
| Codex | `codex` 0.144.4 (npm) | Installed; ChatGPT sub | Mid (default) → Expensive (high/xhigh) | Implementer, reviewer, rescue | `models/codex.md` |
| Grok | `grok` 0.2.101 | Installed | Cheap–Mid, fast | Implementer (mechanical/parallel), second opinion | `models/grok.md` |
| Claude | `claude` 2.1.211 | Installed; Claude sub | Mid–Expensive | Orchestrator (typical); reviewer. Worker use consumes the same sub that usually orchestrates — flag it in the consult | `models/claude.md` |
| DeepSeek | — | **No CLI installed yet** | Cheap-tier candidate | — | Add adapter when installed |

Platform runtime (not a model): Orca — `models/orca.md`.

## Rules

- This table is model-agnostic plumbing: add/remove rows as subscriptions change.
  Every row needs an adapter file with spawn + preflight commands.
- Tier meanings: Cheap = mechanical/exploration; Mid = default implementer;
  Expensive = ALWAYS owner-confirmed (`core/ORCHESTRATION.md`).
- Cross-vendor review: prefer reviewer vendor ≠ implementer vendor.
- Quota state changes daily — the Model Consult, not this file, decides.
