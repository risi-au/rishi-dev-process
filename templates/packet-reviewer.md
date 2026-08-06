# Reviewer Packet: <task>
<!-- Fresh session; prefer a different vendor than the implementer. -->

ROLE: <FULL_REVIEW | FOCUSED_REREVIEW>. You review. You do not edit.

PLAN SUMMARY (≤20 lines): <what was supposed to happen — not the full plan>
RISK TRIGGERS: <from the contract/plan>
GATE RECEIPT: <summary or attached receipt>
KNOWN LIMITATIONS / OPEN DECISIONS: <list>

FOR FULL_REVIEW — DIFF: <complete release diff + stats>
FOR FOCUSED_REREVIEW — instead provide: the finding IDs under verification, the
fix diff, and the halo files. Check ONLY those + the updated receipt.

READ-ONLY SHELL IS PERMITTED AND EXPECTED: git diff/log/show, grep, lint, typecheck,
tests, `node -e`. Forbidden: editing files, any state-mutating git command. Report; don't fix.
(A blanket "no terminal commands" once made a reviewer verify zero gate numbers.)
**Re-run the gate yourself and report YOUR exit codes — do not accept the receipt's numbers.**
The receipt was written by the party being reviewed, and a gate can report success having
measured nothing (`core/GREEN.md`, mode S). Also check the receipt's own claims: 2026-08-06 a
report cited retained log files that were not in the tree, so it could not be reproduced from
the commit it described.

REVIEW AXES:
(a) **BREAK THE GUARANTEE** — not "does the diff match the plan". Name the guarantee in
    plain language and attack it. Conformance review has passed every real defect we've
    shipped. State the specific ways you suspect it fails, including your own suspicions
    of the orchestrator's reasoning — "the batch is sound" is an acceptable answer, style
    nits are not;
(b) **silent failure sweep** — list every filter, guard, early return, default and catch
    the diff adds. For each: what happens when it rejects something that should have
    passed, and does anything observe the rejection? (`core/GREEN.md`);
(c) safety invariants held; secrets, injection, permissions, destructive paths;
(d) **do the new tests discriminate?** Would any pass against the pre-change code?
(e) lean — flag overbuild and drive-by edits as findings;
(f) **REMOVALS — what does this diff DELETE?** Read it against what it replaced, not on its own.
    List every check, guard, validation, permission test and existing test the diff removes or
    weakens, and say what previously enforced it. **A removed check leaves no trace in a report
    that describes only additions**, so no report will point you at this. 2026-08-06: a revision
    deleted an authorization check and substituted a path that never read the consent table; the
    default account state meant a revoked connection would have kept working for up to 7 days.
    418 tests and a clean production build were green across it, and a reviewer given the same
    diff returned 3 findings, all false, and missed this one.

RETURN (max <N> lines):
```
VERDICT: APPROVED | REQUEST_CHANGES | NEEDS_REWORK
FINDINGS:
<ID> | BLOCKING|NON_BLOCKING|QUESTION | file:line | one-line issue | one-line suggested fix
```
FULL_REVIEW is your ONE broad pass — batch every finding now.
FOCUSED_REREVIEW must not open new broad scope.
