# ADAPTER: graphify (code knowledge graph — tool, not a worker)
<!-- CAP: 120 lines. Quirks dated; prune per core/SELF-IMPROVE.md. -->

Local tree-sitter code graph (`pip install graphifyy`, CLI `graphify`). Indexing
code is FREE (no LLM). Proven in trial (companyos #42, 2026-07-17): one query
before any file reads replaced the blind exploration sweep (~10–20k tokens) and
returned fix sites with file:line.

Sibling tool: **CodeGraph** (`models/codegraph.md`) — use that
for diff/PR blast radius, review risk, and test gaps. **One graph system per
question.** Project `AGENTS.md` may define the split; follow it when present.

## When to use

- FIRST, before reading any file, when locating code: "where is X implemented",
  "what calls Y", "what connects A to B". One or two queries usually suffice.
- Architecture overview, communities, god nodes, path between concepts.
- NOT for "what does this uncommitted diff / PR hit?" — that is CodeGraph.
- NOT a substitute for pre-edit reads — editing still requires reading the real
  file content. Don't force extra queries once you have the map.

## Commands (work from ANY directory via --graph)

```
graphify query "<question>" --graph "<abs path>\graphify-out\graph.json" [--budget N]
graphify path "NodeA" "NodeB" --graph "<abs path>\graphify-out\graph.json"
graphify explain "Node" --graph "<abs path>\graphify-out\graph.json"
```

## The worktree rule (learned the hard way)

`graphify-out/` is gitignored, so it is ABSENT from fresh worktrees/checkouts
(including Orca paths like `~/orca/workspaces/<project>/<task>`).
Never assume a relative `graphify-out/` exists — always pass the absolute
`--graph` path to the project's **main** checkout (e.g.
`C:\dev\companyos\graphify-out\graph.json`). Kickoff prompts and worker packets
must carry that absolute path. This rule is graphify-only; CodeGraph
stores per-worktree under `.codegraph/` (see its adapter).

## Building / refreshing a project graph

- Build: `/graphify <project path>` (the skill) or the pipeline in the skill doc.
  Exclude junk dirs (session artifacts, .claude, legacy) and skip docs/video
  unless the owner approves the token cost — code-only is zero tokens.
- Refresh is EVENT-driven, not scheduled (no cron — code goes stale on merges,
  not on a clock; rebuilds are free/seconds so the hook costs nothing):
  1. `graphify hook install` in the main checkout auto-rebuilds on commit and
     checkout; ALSO mirror post-commit → post-merge so `git pull` refreshes
     (installed on companyos 2026-07-17).
  2. Session-start check (backstop, e.g. hookless machines): if graph.json's
     mtime predates the newest main commit, pull + rebuild before querying.
- Stale graphs give stale line numbers — trust file:line from the graph as a
  locator, not as gospel. Branch-only code may be missing until main is rebuilt
  or you accept CodeGraph in the worktree for structure.

## Known quirks (dated — prune when stale)

- 2026-07-17: gitignored graph absent in Orca worktrees; #42 session lost a query
  to `graph file not found` before locating the main checkout's graph by hand —
  hence the absolute-path rule above.
- 2026-07-16: companyos graph is code-only (347 files; docs/SQL layers deferred —
  docs need ~9 subagents of tokens, SQL needs `pip install "graphifyy[sql]"`).
- 2026-07-18: companyos also runs CodeGraph; dual-graph routing lives in
  project AGENTS.md + this adapter pair — do not hard-code CodeGraph to main cwd.
