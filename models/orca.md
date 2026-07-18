# ADAPTER: Orca (platform runtime, optional)
<!-- CAP: 120 lines. Quirks are dated; prune stale ones per core/SELF-IMPROVE.md. -->

Orca is a terminal/worktree manager — a RUNTIME the process can use, never the
process itself. Use it when the owner wants tracked multi-agent visibility or is
already working in it. A single-worker task does not need it.

## What it provides

- One worktree per task with UI cards/status.
- Terminals hosting any CLI worker (codex / grok / claude).
- task-create → dispatch → check --wait lifecycle tracking.
- Persistent history, usable for retrospectives.

## Worktrees vs code graphs

Orca checkouts are separate folders (e.g.
`C:\Users\rishi\orca\workspaces\companyos\<task>`), not only the main clone.
- **graphify:** still query MAIN's absolute `graphify-out\graph.json` — worktrees
  do not get that dir (`models/graphify.md`).
- **code-review-graph:** per worktree under `.code-review-graph/`; MCP must
  inherit the agent workspace cwd (no hard-coded main path). Build once in a
  new worktree if status is empty (`models/code-review-graph.md`).
- Product work runs in the worktree cwd; kickoff prompts should name both
  Project (canonical main path) and Workspace (open Orca path).

## Mechanics

- Create a worker terminal:
  `orca terminal create --worktree id:<wt> --title <name> --command "<worker cli>" --json`
- Tracked dispatch: `task-create` → `dispatch --inject` → `check --wait`.
- If the session has the `orca-cli` / `orchestration` skills available, prefer
  them over memorized commands — they carry the current command surface.

## Known quirks (dated — prune when stale)

- 2026-07: `orca` returns exit 255 even on success from PowerShell — parse the
  JSON `.ok` field; append `2>$null`.
- 2026-07: don't run `check --wait` as a PowerShell background job (heartbeats
  trip NativeCommandError, killed ~600 s) — run from Bash or poll
  `terminal list` lastOutputAt.
- 2026-07: after `dispatch --inject`, send a follow-up Enter — the prompt can
  land mid-boot, unsubmitted. On codex version prompts, send "2" (Skip).
- 2026-07: codex workers under Orca need the sandbox-bypass launch
  (`models/codex.md`) or worker_done never fires.
- Cost truth (2026-07 retrospective): Orca transport was <1% of tokens; the
  fresh full-context sessions attached to it were the cost. Optimize by running
  fewer, better-scoped tasks — not by avoiding Orca.
