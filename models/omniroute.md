# PLATFORM: OmniRoute (model router)

<!-- CAP: 120 lines. Not a model — a routing layer the workers speak to. -->

Local OpenAI-compatible router at `http://localhost:20128`. Workers (notably cline —
`models/cline.md`) target a **combo name** instead of a vendor model id; OmniRoute
resolves the combo to whichever underlying model currently has headroom.

Why it matters to the process: it decouples the Model Consult from vendor ids. The
consult chooses a **tier and role** (cheap mechanical / default implementer /
adversarial reviewer); OmniRoute picks the actual model. Quota state changes daily,
which is exactly the churn `models/REGISTRY.md` says not to hard-code.

## Health check (this IS the preflight for any OmniRoute-backed worker)

```
curl -s -m 8 -o /dev/null -w "%{http_code}\n" http://localhost:20128/api/v1/models
```

- `200` — up.
- `000` / connection refused — **down**. Every OmniRoute-backed worker will fail
  instantly with a connection error that reads like a bad packet. Stop and tell the
  owner; it is their local service to restart.

List the live combos:

```
curl -s -m 15 http://localhost:20128/api/v1/models | tr ',' '\n' | grep -oE '"id":"sprint[^"]*"'
```

## Combos (2026-07-25)

| Combo | Use for |
|---|---|
| `sprint-implement-r2` | Default implementer for contract / schema / correctness work |
| `sprint-implement` | Standard implementer |
| `sprint-mechanical-cheap` | One-line and mechanical fixes; don't burn a big model |
| `sprint-review-r2` | Adversarial reviewer (see PROCESS.md review state machine) |
| `sprint-rescue` | Second attempt after an implementer fails twice |
| `sprint-scout-cheap` | Exploration / locating code |
| `sprint-conformance-cheap` | Checklist / conformance verification |

Also exposes `auto/*` combos (best-coding, pro-reasoning, fast, …) and direct
vendor passthroughs. Prefer the `sprint-*` combos — they encode the tier intent.

## Failure handling

Per `core/ORCHESTRATION.md`, a dead router is an **infrastructure event**, not a
review cycle and not a worker fault. Observed 2026-07-25: OmniRoute died mid-card;
the running worker terminated after completing its edits but before its own gate run,
and the next dispatch failed in seconds. Both looked like worker problems.

Rule: **any worker death with a connection error → re-run the health check before
re-dispatching or blaming the packet.**

When it is down, the orchestrator can still run gates, review diffs, commit, and open
PRs — only dispatch is blocked. Say so plainly and keep the non-dispatch work moving.
