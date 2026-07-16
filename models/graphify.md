# ADAPTER: graphify (code knowledge graph — tool, not a worker)
<!-- CAP: 120 lines. Quirks dated; prune per core/SELF-IMPROVE.md. -->

Local tree-sitter code graph (`pip install graphifyy`, CLI `graphify`). Indexing
code is FREE (no LLM). Proven in trial (companyos #42, 2026-07-17): one query
before any file reads replaced the blind exploration sweep (~10–20k tokens) and
returned fix sites with file:line.

## When to use

- FIRST, before reading any file, when locating code: "where is X implemented",
  "what calls Y", "what connects A to B". One or two queries usually suffice.
- NOT a substitute for pre-edit reads — editing still requires reading the real
  file content. Don't force extra queries once you have the map.

## Commands (work from ANY directory via --graph)

```
graphify query "<question>" --graph "<abs path>\graphify-out\graph.json" [--budget N]
graphify path "NodeA" "NodeB" --graph "<abs path>\graphify-out\graph.json"
graphify explain "Node" --graph "<abs path>\graphify-out\graph.json"
```

## The worktree rule (learned the hard way)

`graphify-out/` is gitignored, so it is ABSENT from fresh worktrees/checkouts.
Never assume a relative `graphify-out/` exists — always pass the absolute
`--graph` path to the project's main checkout (e.g.
`C:\dev\companyos\graphify-out\graph.json`). Kickoff prompts and worker packets
must carry that absolute path.

## Building / refreshing a project graph

- Build: `/graphify <project path>` (the skill) or the pipeline in the skill doc.
  Exclude junk dirs (session artifacts, .claude, legacy) and skip docs/video
  unless the owner approves the token cost — code-only is zero tokens.
- Refresh after merged changes: `/graphify <path> --update` (incremental, cached).
- Refresh cadence: before starting a task on a project whose graph is older than
  the last few merges. Stale graphs give stale line numbers — trust file:line
  from the graph as a locator, not as gospel.

## Known quirks (dated — prune when stale)

- 2026-07-17: gitignored graph absent in Orca worktrees; #42 session lost a query
  to `graph file not found` before locating the main checkout's graph by hand —
  hence the absolute-path rule above.
- 2026-07-16: companyos graph is code-only (347 files; docs/SQL layers deferred —
  docs need ~9 subagents of tokens, SQL needs `pip install "graphifyy[sql]"`).
