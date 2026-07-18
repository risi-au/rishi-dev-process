# ADAPTER: code-review-graph (review impact graph — tool, not a worker)
<!-- CAP: 120 lines. Quirks dated; prune per core/SELF-IMPROVE.md. -->

Local Tree-sitter graph + MCP/CLI (`pip install code-review-graph` or
`code-review-graph` on PATH). Stores SQLite under `.code-review-graph/` in the
**current checkout**. Built for blast radius, risk-scored review context, and
token-efficient impact queries — not as a full multi-modal knowledge map.

Sibling tool: **graphify** (`models/graphify.md`) — architecture / locate /
path/explain against MAIN `graphify-out/`. **One graph system per question.**
Project `AGENTS.md` may define the split; follow it when present.

## When to use

- Diff / PR impact: "what does this change touch?", test gaps, risk score.
- Review context: minimal file set for a change set (`detect-changes`,
  `get_review_context`, `get_impact_radius` via MCP or CLI).
- NOT first choice for pure "how does auth work / what connects A to B" when
  graphify is available — use graphify for architecture location.
- NOT a substitute for reading files you will edit.

## Commands (run from the open workspace / worktree root)

```
code-review-graph status
code-review-graph build          # full parse once per checkout if empty/stale
code-review-graph update         # incremental after local edits
code-review-graph detect-changes --brief
code-review-graph search "<query>"
code-review-graph impact --files <path>
code-review-graph serve          # MCP stdio (agents)
```

MCP tools (when server is connected): `list_graph_stats_tool`,
`detect_changes_tool`, `get_impact_radius_tool`, `get_review_context_tool`,
`build_or_update_graph_tool`, search/query variants, etc. Prefer MCP when the
client has the server enabled; otherwise CLI is equivalent.

## The worktree rule (opposite of graphify)

`.code-review-graph/` is **per checkout**, including Orca worktrees under
`~/orca/workspaces/<project>/<task>`. Each worktree needs its own build if the
DB is missing (status shows 0 files).

- MCP must **inherit the agent workspace cwd** — do **not** pin `cwd` to the
  main repo path in Cursor/Codex/Grok configs when using Orca worktrees.
- Optional: pass `repo_root` = open workspace when the client might spawn MCP
  from a wrong directory.
- Do **not** point CRG at MAIN via a hard-coded absolute graph path the way
  graphify requires — that re-attaches every worktree agent to main's graph and
  wrong git dirty state.

## Building / refreshing

- Once per new Orca worktree (or after large rebase): `code-review-graph build`
  from that worktree root (~20–40s for companyos-sized monorepos).
- Day-to-day: `code-review-graph update` or MCP incremental build after edits.
- Freshness is local: worktree graph tracks that branch's tree, not main's
  graphify-out.

## Install / agent wiring (machine-level)

- CLI: `pip install code-review-graph` (or pipx/uv tool).
- Register MCP without fixed main cwd, e.g. command `code-review-graph` args
  `serve` in Cursor/Codex/Grok MCP config; project `.mcp.json` may list it with
  no `cwd` so the host uses the open folder.
- Restart agent sessions after MCP config changes.

## Known quirks (dated — prune when stale)

- 2026-07-18: `code-review-graph install` wrote MCP `cwd` to the user home
  directory → empty graph (0 nodes). Fix: remove fixed cwd or set to open
  workspace; never leave home as cwd.
- 2026-07-18: pinning MCP cwd to MAIN broke Orca worktrees (agents saw main
  graph while editing feature branches). Inheritance of workspace cwd is the
  correct multi-worktree default.
- 2026-07-18: companyos also uses graphify; dual-graph routing in project
  AGENTS.md — architecture → graphify (MAIN abs path); review/impact → CRG
  (this workspace).
