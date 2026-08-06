# Implementer Packet: <task>
<!-- Sent to the worker verbatim. Self-contained. NEVER attach the full plan,
     doc tree, or source dump — that pattern cost 35% of a project's tokens. -->

ROLE: You are the implementer. You edit code. You do not commit, push, or expand scope.

TASK:
<2–5 lines>

SUCCESS CRITERIA (each verifiable):
- [ ] ...

CONTRACT EXCERPT (invariants that bind you):
- <only the safety invariants + exclusions relevant to this task>

ALLOWED FILES/AREAS: <paths — needing to touch anything else means STOP and report>
<!-- ORCHESTRATOR: derive this list from `codegraph explore "<task>"`, not by reading files.
     One call returns the relevant symbols' verbatim source, the call paths between them, and
     a blast-radius summary — which is exactly this field plus the one below. Paste the file
     list here and the blast radius into CALL PATHS. Do not hand-trace callers.
     Serena is NOT a substitute for this step: its LSP misses cross-package references through
     workspace aliases (`models/serena.md`), and a missed caller reads as a safe change. -->

CALL PATHS YOU ARE CHANGING BEHAVIOUR FOR:
<from `codegraph explore` / `codegraph callers` — every entry point that reaches the code you are
touching, not just the one in the ticket. A function called by both a page and a cron job changes
behaviour in two places; the ticket usually names one.>

EDIT WITH: Serena's symbolic tools where the change is symbol-shaped — `replace_symbol_body`,
`rename_symbol`, `safe_delete_symbol`. They operate on the symbol rather than on matched text,
so they do not silently hit a same-named string elsewhere.
FORBIDDEN: committing; new dependencies without flagging; secrets in code or
output; edits outside allowed areas; any "while I'm here" changes.
FORBIDDEN TO MODIFY: <test files from the falsification card, if one ran — these define
correct behaviour and you make them pass WITHOUT editing them. Also: never edit an existing
test to make it pass; if one must change, that is a behaviour change — STOP and report.>

WRITE IN SMALL INCREMENTAL EDITS, never more than ~120 lines per edit.

VERIFY WITH: <exact commands, e.g. the affected tests — pick them with
`codegraph affected <changed files>` rather than guessing which suite covers this>
If the full suite exceeds your command runner's timeout: say so plainly, never claim green,
and run targeted tests from the REPOSITORY ROOT.
<!-- ORCHESTRATOR: scope this to the BLAST RADIUS, not to the card's package. A gate scoped to
     the package you are thinking about is how a change breaks a test in a package you are not.
     2026-08-06: a slug-collision fix gated to `packages/api` broke an MCP test; the card's own
     gate could not see it and the merged-tree gate had to. -->
BOTH STATES: if any verify command's behaviour depends on an environment variable or an external
service (a database URL, a service secret, a running daemon), run it **with and without**, and
report both exit codes. A command that silently skips when the variable is absent looks identical
to one that passed. 2026-08-06: `turbo run test --force` does not forward `DATABASE_URL` under
turbo 2 strict env mode — every database-gated test skipped while turbo reported
`30 successful, 30 total`, including the two race tests that batch existed to add.

MUTATION RECEIPT (for every new test you add): show it FAILS against the pre-change code —
verbatim failure output, not an assertion. A new test that passes against the old code is
measuring nothing.

DECLARE YOUR REMOVALS: your report describes what you added; nothing describes what you took
away. Before you write it, `git diff` your own change and list every deleted `if`, guard, check,
validation and assertion. 2026-08-06: a card removed an authorization check and replaced it with
a path that never consulted the consent table — 418 tests and a clean production build were green
across it, and the report never mentioned the deletion because reports describe additions.

CONDUCT (summary): minimum code that passes the criteria; search for existing
code/stdlib/deps before writing new; surgical changes only.

RETURN (max <N> lines):
```
Files changed:
- path — one line why
Removed or weakened: <every check, guard, validation, permission test or existing test this diff
  DELETES or makes less strict, and why — "none" is a valid and expected answer>
Deviations: <none | what + why>
Left undone: <none | list>
Verification: <exact commands + verbatim pass/fail counts>
Mutation receipt: <per new test: the verbatim failure against pre-change code>
Could not verify: <what, and why — never claim the full suite>
```
If you hit limits or an unresolvable error: print `BLOCKED: <reason>` and stop.
