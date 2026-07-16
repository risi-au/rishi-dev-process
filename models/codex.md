# ADAPTER: Codex CLI
<!-- CAP: 120 lines. Quirks are dated; prune stale ones per core/SELF-IMPROVE.md. -->

## Preflight (before first dispatch of a session)

```
codex exec --sandbox read-only -C "<project>" "Reply with exactly: PREFLIGHT-OK" < /dev/null
```

Passes if it prints PREFLIGHT-OK. On failure: check `codex login status`, retry
once, then swap worker (`core/ORCHESTRATION.md` failure handling).

## Spawn — headless implementer

```
codex exec --sandbox workspace-write -c model=<model> -c model_reasoning_effort=<effort> \
  -C "<worktree>" "<packet prompt>" < /dev/null
```

- `< /dev/null` is mandatory — codex hangs waiting on stdin otherwise.
- Via cmd.exe wrappers the prompt must not contain `"` or `%`.
- Effort ladder: minimal / low / medium / high / xhigh. xhigh = Expensive tier →
  owner confirmation required.

## Spawn — inside Orca (tracked)

```
orca terminal create --worktree id:<wt> --title <name> \
  --command "codex --dangerously-bypass-approvals-and-sandbox" --json
```

The bypass flag is needed because Orca's worker_done protocol requires the worker
to exec the `orca` CLI, which the workspace-write sandbox blocks (run hangs on an
approval prompt). Granular equivalent: `-s danger-full-access -a never`.
Only inside owner-approved, isolated worktrees.

## Review / rescue via Claude Code plugin

When orchestrating from Claude Code, the codex plugin lanes (`/codex:review`,
`/codex:adversarial-review`, `/codex:rescue`) are proven where raw
`codex exec --sandbox` hit write failures. Job state persists per workspace.

## Known quirks (dated — prune when stale)

- 2026-07: Windows sandbox needs ACLs granted on the worktree before codex can
  write: `icacls <worktree> /grant "CodexSandboxUsers:(OI)(CI)(M)" /t /q`
- 2026-07: codex cannot commit on Windows (sandbox denies `.git` writes) — the
  orchestrator commits.
- 2026-07: no package-manager binaries (e.g. pnpm) inside the sandbox; codex
  verifies with tsc/eslint/vitest directly. The orchestrator's root gate run is
  the real gate.
- 2026-07: three sandbox blocks (CreateProcessAsUserW) in one project before any
  preflight existed — hence the mandatory preflight above.
- 2026-07: include in packets: "On limits print LIMIT-ALERT: and stop." Monitor
  logs for `LIMIT-ALERT:|out of credits`.
