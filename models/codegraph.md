# ADAPTER: CodeGraph (structural code graph — tool, not a worker)
<!-- CAP: 120 lines. Quirks dated; prune per core/SELF-IMPROVE.md. -->

Local Tree-sitter → SQLite graph + MCP/CLI. Stores its index under `.codegraph/` in the
**checkout it describes** (worktrees each need their own). Built for blast radius,
call paths, and token-efficient retrieval.

**Replaced `code-review-graph` on 2026-08-02** after a measured bake-off; see
`core/GRAPH-TOOLS.md` for the comparison and the failure that ended CRG.

Sibling tools: **Serena** (`models/serena.md`) for symbolic *editing*; **graphify**
(`models/graphify.md`) for architecture and cross-document questions.
**One tool per question.**

## Install (per machine, once)

```
# Windows
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 | iex
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh

codegraph install -y      # register the MCP server into every detected agent
codegraph telemetry off   # anonymous usage stats are ON by default
```

`codegraph install -y` writes MCP config for **every agent it detects** — Claude Code,
Codex, Cursor, opencode, Kiro, Antigravity — not just the one you are in. Expected, but
know that it is broader than a single-agent install.

## When to use

- "What breaks if I change this?" — blast radius before touching shared code.
- "Who calls this / what does this call?" — precise, cross-package.
- Assembling an implementer packet: `explore` returns the allowed-file set, the call
  paths, and verbatim source in one call.
- Picking targeted tests: `codegraph affected <files...>`.
- NOT for "how does auth work" — that is graphify.
- NOT a substitute for reading the file you will edit.

## Commands

```
codegraph init [path]           # full build; one time per checkout (~4s / 550 files)
codegraph sync [path]           # incremental after edits (~0.7s)
codegraph status [path]         # file/node counts — the health check that matters
codegraph explore "<task>"      # source + call paths + blast radius (the main one)
codegraph callers <symbol>
codegraph callees <symbol>
codegraph impact <symbol>
codegraph affected [files...]   # test files affected by changed sources
codegraph query <search>        # find a symbol
codegraph node <name>           # one symbol's source + caller/callee trail
```

MCP tool names mirror these: `codegraph_explore` (primary), plus `codegraph_node`,
`codegraph_search`, `codegraph_callers`, `codegraph_callees`.

## Quirks (dated — verify before trusting)

- **2026-08-02, Windows: the `.cmd` shim breaks stdio MCP.** Registering
  `codegraph serve --mcp` fails with `-32000: Connection closed`, because the launcher
  is a `.cmd` wrapper. Register the vendored runtime directly:
  ```
  claude mcp add --scope user codegraph -- \
    "%LOCALAPPDATA%\codegraph\current\node.exe" --liftoff-only \
    "%LOCALAPPDATA%\codegraph\current\lib\dist\bin\codegraph.js" serve --mcp
  ```
- **2026-08-02: Git Bash cannot resolve `codegraph` on PATH** for the same reason —
  bash does not resolve `.cmd`. A hook written as `command -v codegraph || exit 0`
  silently no-ops **forever**. `.claude/hooks/codegraph.sh` in `companyos-lexi` falls
  back to the node runtime; copy that pattern rather than re-deriving it.
- **The file watcher only runs while an MCP server is live.** Edits made outside a
  session are not picked up until the next `sync`. The session-start hook covers this.
- **`init` is per checkout.** A fresh worktree has no index. The hook builds one; without
  the hook you must run `codegraph init .` yourself.

## The check that matters

`codegraph status` prints a file count. **Compare it to the size of the repo.** An index
covering a fraction of the tree still answers every query confidently, and an unindexed
symbol returns zero callers — indistinguishable from one that genuinely has none. That
single unchecked number is what made CRG unsound for weeks.
