# GRAPH-TOOLS — use the map before you read the territory

Three code-intelligence tools exist in this setup. **They are not alternatives to each other.**
They answer different kinds of question, and using the wrong one wastes the time all three were
bought to save.

They exist for one reason: **stop paying to re-read files you have already read.** An agent that
greps its way around a large repo burns tens of thousands of tokens rediscovering structure that is
already indexed.

**Replaced code-review-graph, 2026-08-02.** CRG was measured against CodeGraph and Serena on a real
card (`companyos-lexi` #252) and lost on the only axis that matters: it had indexed **25 of 496
files** and nobody had noticed. See "Why CRG was removed" at the bottom — the failure is worth
understanding, because any indexed tool can repeat it.

---

## The split, in one line each

| | **CodeGraph** (MCP + CLI) | **Serena** (MCP, LSP) | **graphify** (CLI skill) |
|---|---|---|---|
| Question type | *Structural.* "Who calls this? What breaks?" | *Symbolic.* "Edit this symbol precisely" | *Conceptual.* "How does this area work?" |
| Backed by | Tree-sitter → SQLite index | Live language server | Semantic graph + communities |
| Covers | Code only | Code only | Anything — code, docs, papers, images |
| Output | Blast radius + call paths + verbatim source | Symbol bodies, references, safe edits | Communities, narrative, an HTML map |
| Reach for it | **First**, on any code question | When *editing* a symbol | Onboarding, architecture, cross-document |

**Rule of thumb:** if the answer is a *list* (callers, dependents, blast radius), that is
**CodeGraph**. If you are about to *change a symbol*, that is **Serena**. If the answer is a
*paragraph* (how this works, why these relate), that is **graphify**.

---

## CodeGraph — the structural index, and the default

Use it **first**, before Grep/Glob/Read, whenever the question is about code.

| Tool / command | Use when |
|---|---|
| `codegraph_explore` / `codegraph explore "<task>"` | **The one you want.** Relevant symbols' verbatim source + call paths + blast radius, in one call |
| `codegraph callers <sym>` / `callees <sym>` | Precise "who calls this" / "what does this call" |
| `codegraph impact <sym>` | What is affected by changing a symbol |
| `codegraph affected <files...>` | Which **test files** a change touches — use this to pick targeted tests |
| `codegraph query <search>` | Find a symbol by name or keyword |
| `codegraph node <name>` | One symbol's source + caller/callee trail |

**The moment it earns its keep most:** before changing anything shared — an auth check, a service
function, a type used across packages. `codegraph explore` answers in one call what hand-tracing
takes ten reads to get wrong. If you are about to hand-trace callers, you have already made the
mistake.

**It does not replace reading the file you are about to edit.** The graph tells you where to look
and what depends on it; it does not tell you what the code should say.

### Keeping it fresh — mostly automatic, one manual case

- **Per checkout, including every worktree.** The index lives in `.codegraph/` in the checkout it
  describes. A new worktree has none until one is built.
- **`.claude/hooks/codegraph.sh` handles this automatically** on `companyos-lexi`: at session start
  it builds a full index if none exists (~4s on a 550-file repo) and syncs if one does; after every
  Edit/Write it syncs (~0.7s). It prints a one-line file/node count at session start.
- **If you see that count look wrong — say, tens of files in a repo with hundreds — stop and
  rebuild.** `codegraph init <path>` does a full rebuild. Do not work off a partial index.
- On a repo without the hook, the manual equivalents are `codegraph init .` once, then
  `codegraph sync` after edits.
- Registering the MCP is **per machine, not per repo**: `codegraph install -y`.

---

## Serena — symbolic editing, and a sharp caveat

Serena is LSP-backed, so it reads live code with no index to go stale. Its tools include
`find_symbol`, `find_referencing_symbols`, `get_symbols_overview`, and the editing set:
`replace_symbol_body`, `insert_after_symbol`, `rename_symbol`, `safe_delete_symbol`.

**Use it when you are changing a symbol.** A rename or a body replacement through Serena is safer
than a text edit, because it operates on the symbol, not on matched characters. This makes it the
right tool for a dispatched **implementer**.

**Do NOT use it as the authority on "who calls this" in a monorepo.** Measured 2026-08-02 on
`companyos-lexi`: asked for references to `getOpsHealth`, Serena returned 3 files and **missed
`apps/os/src/lib/api.ts`**, which imports it at line 123 and calls it at line 483. The import is
`from "@companyos/api"` — a workspace package alias, which its language server did not follow
across the package boundary. In a pnpm monorepo where nearly every app→package call has that shape,
this is the common case, not an edge case.

**A missing reference reads exactly like a safe change.** For blast radius, ask CodeGraph.

Serena also ships a `write_memory` / `read_memory` layer. It is available; it is not currently part
of this process, and adding it needs a decision about how it relates to the handover documents that
already carry that knowledge.

Setup is per machine: `serena setup claude-code` and `serena setup codex`. Per project, create the
config non-interactively — auto-detection prompts for every language it finds and will hang a
non-interactive shell:

```
serena project create . --name <name> --language typescript --index
```

---

## graphify — the conceptual map

`graphify` turns a folder of *anything* into a persistent, queryable knowledge graph, with community
detection that surfaces connections nobody thought to ask about.

```
graphify <path>                 # build (or rebuild) the graph
graphify update <path>          # incremental re-extract, no LLM needed
graphify query "<question>"     # BFS traversal — broad context
graphify query "<q>" --dfs      # DFS — trace one specific path
graphify path "A" "B"           # shortest path between two concepts
graphify explain "<node>"       # plain-language explanation
graphify <path> --wiki          # agent-crawlable wiki, one article per community
```

**Fast path:** if `graphify-out/graph.json` already exists and the question is natural language about
the codebase, run `graphify query` immediately. Do not rebuild. Do not detect.

**Reach for graphify when:**
- You are new to an area and need the shape of it before touching anything.
- The question spans code *and* documents — "what do we actually do about X?"
- You need an explanation to hand to a human, not a list of symbols.
- You suspect there is a connection you have not thought to look for. Community detection is the
  only tool here that answers questions you did not ask.

**Do not reach for graphify when** a single `codegraph callers` call would answer it. Building or
querying a semantic graph to find out who calls one function is using a map of the country to find
the kitchen.

**graphify does not self-update.** After a substantial change, run `graphify update <path>`. A stale
conceptual graph is worse than none, because it reads as current.

---

## Neither is evidence

None of these is a reason to skip the canonical gate, and none is proof. They tell you where to
look. `core/GREEN.md` still governs what counts as green.

---

## The failure mode this file exists to prevent

An agent is told the graph exists, notes it, and then greps anyway — because grepping is the habit
and the graph requires one extra thought. That happened for an entire session on `companyos-lexi`:
the index updated faithfully on every commit while the agent hand-traced callers to decide whether
an auth change was safe.

**If you catch yourself about to Grep the codebase, that is the signal to query the graph instead.**

## Why CRG was removed (2026-08-02) — the lesson generalises

CRG was not a bad tool. It failed for a reason that can happen to **any** indexed tool, so the
detail matters more than the verdict:

- Its hook only ever ran `code-review-graph update` — *incremental*. A full `build` was never in the
  wiring. The index therefore only ever contained files edited since install: **25 of 496**, for
  weeks, while reporting itself healthy.
- Its `.mcp.json` pinned `cwd` to the **main checkout**, so in any worktree it silently answered
  about a different branch's code.
- `semantic_search_nodes_tool` was documented here as semantic search. It had **0 embeddings** and
  was doing keyword matching.

Each of these is invisible from the outside: **an unindexed symbol returns zero callers, which is
indistinguishable from a symbol that genuinely has none.** That is the shape of a bad merge, not a
missing feature.

Two rules follow, and they apply to CodeGraph exactly as much as they applied to CRG:

1. **Build full, never only incremental.** The session-start hook builds when the index is missing
   rather than quietly syncing nothing.
2. **Look at the file count at session start.** If it does not look like the size of the repo, the
   tool is lying to you and everything downstream of it is unsound.
