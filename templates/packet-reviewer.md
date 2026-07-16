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

REVIEW AXES:
(a) correctness vs the plan summary and contract;
(b) safety invariants held;
(c) security — secrets, injection, permissions, destructive paths;
(d) lean — flag overbuild and drive-by edits as findings.

RETURN (max <N> lines):
```
VERDICT: APPROVED | REQUEST_CHANGES | NEEDS_REWORK
FINDINGS:
<ID> | BLOCKING|NON_BLOCKING|QUESTION | file:line | one-line issue | one-line suggested fix
```
FULL_REVIEW is your ONE broad pass — batch every finding now.
FOCUSED_REREVIEW must not open new broad scope.
