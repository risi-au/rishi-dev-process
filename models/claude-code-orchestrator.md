# ADAPTER: Claude Code as orchestrator — mechanical write-lock

<!-- CAP: 120 lines. Quirks are dated; prune stale ones per core/SELF-IMPROVE.md. -->

Makes `core/ORCHESTRATION.md`'s "dispatch by default" a **tool-layer fact** instead of an
advisory rule. Adapted from opencode-fusion (2026-07): *"The main agent's file editing is
mechanically denied. Its only way to change a file is to hand a spec to the sidekick."*

Rationale, dated 2026-07-26: on companyos-lexi #20 the orchestrator self-implemented twice
without the owner waiver the harness requires. Each edit was individually defensible —
smaller than the packet needed to explain it — which is exactly how the rule erodes. Intent
did not hold; a deny rule does.

## Deny writes, never reads

Adopt the **write** restriction only. Devin Fusion also has the main agent read as little
as possible, for cost. **Do not copy that.** On the same two batches, two defects were
found *only* by the orchestrator reading code and holding the whole diff plus gate output
at once — including a UI filter that compiled cleanly while silently discarding data. Our
constraint is correctness, not token cost.

- Orchestrator: **no writes to source. Unlimited reads.**
- Workers: write freely inside their packet's allowed files.

## Project `.claude/settings.json`

Place in the project repo (not the harness). Adjust globs per project layout.

```json
{
  "permissions": {
    "deny": [
      "Edit(packages/**)",
      "Write(packages/**)",
      "Edit(apps/**)",
      "Write(apps/**)"
    ]
  }
}
```

Allowed by omission, and deliberately so: `docs/**` (plans, handovers), the packet
directory, `retros/**`, and every read/search tool. Add source roots as a project gains
them — a new top-level `services/` or `lib/` is a gap until it is listed.

**Verify after any settings change**, or a silent typo leaves the lock off: attempt one
trivial `Write` to a denied path and confirm it is refused. A deny rule that was never
exercised is not known to work. Verified 2026-07-26 on companyos-lexi — refused with *"File
is in a directory that is denied by your permission settings"*, **effective immediately, no
session restart needed**.

## What this lock does NOT stop (verified, 2026-07-26)

**Bash bypasses it.** `printf 'x' > packages/api/src/f.txt` succeeded with the deny rules
active — the deny list governs the `Edit`/`Write` tools, not shell redirection.

So this is a **guardrail against drift, not a sandbox**. That is still the right trade:
both real violations on #20 were `Edit` calls made because the fix felt smaller than the
packet, and the guardrail makes exactly that path impossible. A determined bypass via shell
is a decision, not a slip.

If the owner wants it airtight, the real mechanism is a `PreToolUse` hook on `Bash` that
rejects redirections and `sed -i`/`tee` into source globs (opencode-fusion uses a bash
allowlist limited to verification commands: lint, `git diff`, tests). Not implemented —
propose it in a retro if drift recurs through the shell.

## The escape hatch stays the owner's

Unchanged from `core/ORCHESTRATION.md`: self-implementing needs the **owner's waiver in
the Session Brief**. When granted, note in the brief which paths it covers. Record every
orchestrator edit in the dispatch ledger's own-edits table
(`templates/dispatch-ledger.md`) and name it in the finish report.

Genuinely trivial exceptions worth asking for up front: a one-line lint/type fix on a
worker's output, and doc-only edits. Both were the actual cases on #20.

## Quirks (dated)

- **2026-07-26 — settings-file precedence.** Project `.claude/settings.json` merges with
  user settings; a broad user-level `allow` does not override a project `deny` (deny wins),
  but confirm on the machine rather than assuming.
- **2026-07-26 — do not write control or separator characters into source via the Edit
  tool.** Typed U+2028/U+2029/U+0085 were normalised in transit, and a `node -e` attempt
  wrote a real newline inside a string literal and broke the file. Build them from code
  points (`String.fromCharCode(0x2028)`) so the source stays ASCII. Recovery, if a staged
  batch exists: `git checkout -- <file>` restores from the index.
- **2026-07-26 — `sleep`-then-check is blocked** in the Bash tool. Wait on the background
  task notification, or use an until-loop; do not chain short sleeps.

## Preflight

None needed — this is the session's own runtime. But **confirm the write-lock is live
before the first dispatch**, alongside the worker preflight, by attempting one denied edit.
