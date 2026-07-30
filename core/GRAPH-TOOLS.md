# GRAPH-TOOLS — use the map before you read the territory

Two graph tools exist in this setup. **They are not alternatives to each other.** They answer
different kinds of question, and using the wrong one wastes the time both were bought to save.

Both exist for one reason: **stop paying to re-read files you have already read.** An agent that
greps its way around a large repo burns tens of thousands of tokens rediscovering structure that is
already indexed. On `companyos-lexi` the indexed answer has repeatedly come in at **~90% fewer
tokens** than the file-reading equivalent.

---

## The split, in one line each

| | **code-review-graph** (MCP) | **graphify** (CLI skill) |
|---|---|---|
| Question type | *Structural.* "Who calls this? What breaks?" | *Conceptual.* "How does this area work? What connects to what?" |
| Covers | Code only — functions, classes, calls, imports, tests | Anything — code, docs, papers, images, video |
| Freshness | Live. Auto-updates on file change via hooks | Built on demand; `graphify update <path>` to refresh |
| Output | Precise facts, token-efficient snippets | Communities, narrative explanations, an HTML map, a wiki |
| Reach for it | Before Grep/Glob/Read, on any code question | Onboarding, architecture, cross-document synthesis |

**Rule of thumb:** if the answer is a *list* (callers, dependents, tests, changed symbols), that is
**code-review-graph**. If the answer is a *paragraph* (how this works, why these things relate), that
is **graphify**.

---

## code-review-graph — the structural index

Use it **first**, before Grep/Glob/Read, whenever the question is about code.

| Tool | Use when |
|---|---|
| `detect_changes_tool` | Reviewing a change — gives risk-scored analysis |
| `get_review_context_tool` | Need source snippets for review, token-efficiently |
| `get_impact_radius_tool` | "What breaks if I change this?" — before touching shared code |
| `get_affected_flows_tool` | Which execution paths a change touches |
| `query_graph_tool` | `callers_of` / `callees_of` / `imports_of` / `tests_for` |
| `semantic_search_nodes_tool` | Find a function or class by name or keyword |
| `get_architecture_overview_tool` | High-level structure; pair with `list_communities_tool` |
| `refactor_tool` | Planning renames, finding dead code |

**The moment it earns its keep most:** before changing anything shared — an auth check, a service
function, a type used across packages. `get_impact_radius_tool` answers in one call what hand-tracing
callers takes ten reads to get wrong. If you are about to hand-trace callers, you have already made
the mistake.

**It does not replace reading the file you are about to edit.** Read that. The graph tells you where
to look and what depends on it; it does not tell you what the code should say.

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
the codebase, run `graphify query` immediately. Do not rebuild. Do not detect. The graph is there —
use it.

**Reach for graphify when:**
- You are new to an area and need the shape of it before touching anything.
- The question spans code *and* documents — "what do we actually do about X?"
- You need an explanation to hand to a human, not a list of symbols.
- You suspect there is a connection you have not thought to look for. Community detection is the
  only tool here that answers questions you did not ask.

**Do not reach for graphify when** a single `query_graph_tool` call would answer it. Building or
querying a semantic graph to find out who calls one function is using a map of the country to find
the kitchen.

---

## Keeping them alive

- **code-review-graph** updates itself via hooks on file change. If it looks stale, that is a bug
  worth reporting, not a reason to abandon it for Grep.
- **graphify** does not. After a substantial change, run `graphify update <path>`. A stale conceptual
  graph is worse than none, because it reads as current.
- Neither is a reason to skip the canonical gate, and neither is evidence. They tell you where to
  look. `core/GREEN.md` still governs what counts as proof.

---

## The failure mode this file exists to prevent

An agent is told the graph exists, notes it, and then greps anyway — because grepping is the habit and
the graph requires one extra thought. That happened for an entire session on `companyos-lexi`: the
index updated faithfully on every commit while the agent hand-traced callers to decide whether an auth
change was safe. Exactly the question one `get_impact_radius_tool` call answers.

**If you catch yourself about to Grep the codebase, that is the signal to query the graph instead.**
