# LANE: Investigation — deliverable is findings, not code
<!-- CAP: 80 lines -->

For: diagnosing a problem without fixing it yet, feasibility checks, performance
analysis, retrospectives, architecture questions, "why is X happening?"

## Steps

1. **Define the question.** One or two sentences, agreed with the owner, PLUS what
   a sufficient answer looks like — the stop condition. Investigations without stop
   conditions are token sinks.
2. **Read-only by default.** No product code changes. Temporary instrumentation or
   repro scripts go in a scratch area and are removed (or flagged) before finishing.
3. **Choose depth with the owner** if workers would be dispatched: quick look
   (self, ≤30 min) vs multi-angle sweep (workers per Model Consult).
4. **Investigate.** Prefer evidence over inference: logs, git history, repro runs,
   measurements. Label every finding `CONFIRMED` (evidence attached) or
   `PLAUSIBLE` (reasoning only).
5. **Report.** Findings doc containing: question, method, findings (confirmed vs
   plausible), and a recommended next action — often a bugfix/feature/refactor
   lane task with a ready plan seed. Store where the owner says (project docs/
   usually; ask if unclear).
6. **Retro** if workers were dispatched.

## Lane rules

- Never silently morph into implementation. Recommend; the owner picks the lane.
- If the agreed depth is exhausted without an answer, report what was ruled out
  and stop — that IS a valid finding.
