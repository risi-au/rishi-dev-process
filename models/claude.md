# ADAPTER: Claude Code CLI
<!-- CAP: 120 lines. Quirks are dated; prune stale ones per core/SELF-IMPROVE.md. -->

Typical role: ORCHESTRATOR — the session the owner starts. Can also be a worker.

## Preflight (as worker)

```
claude -p "Reply with exactly: PREFLIGHT-OK"
```

## Spawn — headless worker

```
claude -p "<packet prompt>" --permission-mode acceptEdits [--model <model>]
```

Run from inside the worktree (or pass `--add-dir <worktree>`). Never use
bypass-permissions modes outside owner-approved, isolated worktrees.

## Notes

- Claude-as-worker consumes the same subscription that usually runs the
  orchestrator — state this in the Model Consult so the owner can weigh it.
- Claude Code auto-reads AGENTS.md/CLAUDE.md in the working directory; keep
  packets self-contained anyway so behavior matches other workers.
- Subagents (the Agent tool) inside an orchestrator session are cheaper than
  separate CLI spawns for read-only research — use them for exploration, not
  implementation, unless the owner opts in.
