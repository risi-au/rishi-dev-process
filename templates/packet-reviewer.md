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
(e) lean — flag overbuild and drive-by edits as findings.

RETURN (max <N> lines):
```
VERDICT: APPROVED | REQUEST_CHANGES | NEEDS_REWORK
FINDINGS:
<ID> | BLOCKING|NON_BLOCKING|QUESTION | file:line | one-line issue | one-line suggested fix
```
FULL_REVIEW is your ONE broad pass — batch every finding now.
FOCUSED_REREVIEW must not open new broad scope.
