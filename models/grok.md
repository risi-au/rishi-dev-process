# ADAPTER: Grok CLI
<!-- CAP: 120 lines. Quirks are dated; prune stale ones per core/SELF-IMPROVE.md. -->

## Preflight

```
grok -p "Reply with exactly: PREFLIGHT-OK" --always-approve --no-auto-update --cwd "<project>"
```

## Spawn — headless implementer

```
grok -p "<packet prompt>" -m <model> --always-approve --no-auto-update --cwd "<worktree>"
```

Always use `--cwd` rather than `cd <dir> && grok ...` — prefix-matched command
allowlists (and Orca dispatch rules) require the command to START with the CLI name.

## Known quirks (dated — prune when stale)

- 2026-07: WITHOUT `--always-approve`, headless grok reads code, prints
  "Implementing…", and exits 0 having written NOTHING — the classic silent no-op.
  Always verify the diff exists after any grok run.
- 2026-07: do NOT use the `agent` subcommand headlessly (doesn't write files).
- 2026-07: composer-2.5 models reject `--effort high` (HTTP 400) — omit effort.
- If a no-op recurs with the flag set: rerun once with `--debug-file <log>`,
  then swap worker.
