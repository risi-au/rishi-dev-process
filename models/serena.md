# ADAPTER: Serena (LSP symbolic editing — tool, not a worker)
<!-- CAP: 120 lines. Quirks dated; prune per core/SELF-IMPROVE.md. -->

MCP server backed by real language servers (40+ languages). Reads and **edits** code at
the symbol level rather than by matching text. Added 2026-08-02 alongside CodeGraph.

Sibling tools: **CodeGraph** (`models/codegraph.md`) for blast radius; **graphify**
(`models/graphify.md`) for architecture. **One tool per question.**

## Install (per machine, once)

```
uv tool install -p 3.13 serena-agent
serena init
serena setup claude-code
serena setup codex
```

Per project, create the config **non-interactively** — bare `serena project index`
auto-detects languages and prompts for each one, which hangs any non-interactive shell
(`Error: EOF when reading a line`, 2026-08-02):

```
serena project create . --name <name> --language typescript --index
```

## When to use

- **Editing a symbol**: `replace_symbol_body`, `insert_after_symbol`,
  `insert_before_symbol`, `rename_symbol`, `safe_delete_symbol`. Safer than a text edit
  because it operates on the symbol, not on matched characters.
- Reading one symbol precisely: `find_symbol`, `get_symbols_overview`.
- `get_diagnostics_for_file` — compiler/linter state for a file without running a build.
- This makes Serena the right tool for a dispatched **implementer**.

## When NOT to use — the caveat that matters

**Do not treat `find_referencing_symbols` as the authority on blast radius in a monorepo.**

Measured 2026-08-02 on `companyos-lexi`, asking for references to `getOpsHealth`:

| Tool | Result |
|---|---|
| CodeGraph | 5 files, including `apps/os/src/lib/api.ts` |
| Serena | 3 files — **missed `apps/os/src/lib/api.ts`** |

`api.ts` imports it at line 123 and calls it at line 483, via `from "@companyos/api"` —
a workspace package alias its language server did not follow across the package boundary.
In a pnpm monorepo nearly every app→package call has that shape.

**A missing reference reads exactly like a safe change.** For "what breaks if I change
this", ask CodeGraph. Use Serena to make the edit once CodeGraph has told you the scope.

## Output shape (where it beats CodeGraph)

`find_referencing_symbols` returns, per reference, the surrounding lines with the hit
marked — so you see the *call site in context*, not just a file and line:

```
>1391:    const result = await getOpsHealth(db, { ...input, sendAlerts: true }, ...);
```

Denser than CodeGraph for a targeted question: ~1,600 tokens vs ~3,400 on the same query.

## Testing it without an agent restart

MCP tools only load on session start. To exercise Serena mid-session, use its HTTP
project server:

```
serena start-project-server --port 24285
POST http://127.0.0.1:24285/query_project
  {"project_name": "<name>", "tool_name": "<tool>", "tool_params_json": "{...}"}
```

Read-only tools only; it refuses mutating ones on that route. `/heartbeat` for liveness.
There is no `/docs` or `/openapi.json` — those 404.

## Not yet part of this process

Serena ships `write_memory` / `read_memory` (a per-project note store). It is installed
and available. Whether it should carry any of the knowledge currently held in handovers
and `core/` is an open decision, not a default. Do not start writing to it ad hoc — two
memory systems that disagree are worse than one that is merely incomplete.
