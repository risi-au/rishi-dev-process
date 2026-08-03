# Session Brief: S<N> — <area>
<!-- Sent to a batch session verbatim. Self-contained. See lanes/batch.md. -->

Read <coordination issue URL> first — rules, constraints, and owner decisions you must not
re-open.

You are SESSION S<N> of the batch. Branch: `<branch>`. Model: <tier>. Issues: <#a, #b, #c>.

Set up:
```
git worktree add ../<dir> -b <branch> main
cd ../<dir> && <install command>          # name the package manager EXPLICITLY
```
<If under a worktree-managing runtime, say so and drop the command above.>

## You exclusively own

```
<paths>
```

**HARD CONSTRAINT:** do not edit `<files another session owns>`. If you believe you need one,
**stop and report** — do not edit across the boundary.

## Triage findings — confirm, do not re-derive

<Per issue: the verified defect with file:line, and the specific trap.
Traps are the highest-value part of this packet. State them as TRAP: with the consequence,
e.g. "TRAP: do not filter inside listGrants — two other consumers legitimately need agents."

If an issue may already be fixed, say so AND say "do not take this on trust — verify against
current source and quote what you found." A stale verdict is a real result.>

## Owner decisions already made — do not re-litigate

<decision + one line of why>

## Dispatch or implement

<Exactly one:
- "Implement directly — these are trivial cards and dispatch overhead exceeds the work."
- "DISPATCH to OmniRoute. You are an orchestrator, not the implementer: standard → lexi-implement,
  heavy → lexi-implement-hard, then a DIFFERENT worker on lexi-review before you accept the diff.
  Preflight the router first. You still own verification — exit 0 is not proof of work.">

Left unstated, a session self-implements. Say which you want.

## Tooling

Use CodeGraph before reading files. **Check its file count looks like the size of the repo first**
— a partial index returns zero callers, which is indistinguishable from a safe change.
Use Serena for symbol-shaped edits, but **not** for blast radius in a monorepo — its LSP misses
cross-package references through workspace aliases.

## Verify

```
<DATABASE_URL / PORT / commands>
```
Run lint, typecheck, and the tests `codegraph affected <changed files>` names. State exactly what
you ran. Never claim the full suite unless you ran it.

<For UI work: say whether a visual check is expected and how to reach a logged-in state. A session
that cannot verify visually should say so plainly rather than implying it did.>

## Mutation receipt (any security or behaviour fix)

Show the test that FAILS against pre-change code, **verbatim output**. Assert on behaviour that
already had a shape — a new field being absent proves the field is new, not that behaviour changed.
Prefer reverting only the fixed line while leaving any new column in place, so the failure cannot
be explained away.

## Forbidden

Merging; opening a PR against `main`; deploying; touching the VPS or production data; editing
files outside your set; creating a migration if another session is already migrating.

## Report and stop

```
Files changed: path — one line why
Issues closed: which, with the evidence
Deviations: <none | what + why>
Left undone: <none | list>
Verification: exact commands + verbatim pass/fail counts
Could not verify: <what and why — this is a real answer, not a failure>
```
Push `<branch>` and stop.
