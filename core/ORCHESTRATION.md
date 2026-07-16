# ORCHESTRATION — roles, model consult, dispatch, context packets
<!-- HARD CAP: 150 lines. To add, remove (core/SELF-IMPROVE.md). -->

## Roles

| Role | Who | Does |
|---|---|---|
| Orchestrator | The model the owner started the session with | Triage, plan, consult, dispatch, verify, commit, report |
| Implementer | Dispatched worker | Edits code per its packet; runs affected tests; never commits |
| Reviewer | Fresh session; different vendor than implementer when possible | FULL_REVIEW / FOCUSED_REREVIEW per PROCESS.md |
| Specialist | Optional; one named, non-overlapping question (e.g. security pass) | Answers only its question |

ONE implementer owns all edits for a task. The maker is never the final checker.

## The Session Brief (BLOCKING — before any code or dispatch above trivial)

There is no fixed model hierarchy. Subscriptions, quotas, and effort levels change;
the owner decides per session. After triage, send the owner ONE message and WAIT
for the reply. Platform autonomy defaults do not override this checkpoint:

```
Task: <one line> | Lane: <lane> | Size: <class> | Risk: <profile + triggers>
Approach: <one line>
Proposed workers (from models/REGISTRY.md — reviewers count as workers):
- Implementer: <model> @ <effort> — <one-line why>
  (or: self-implement — justify why doing it myself beats briefing a worker)
- Reviewer:    <model> @ <effort> — <one-line why>
- <specialists, if any>
Quota check: which of these have headroom today? Approve / edit?
```

The worker-selection part of the brief is the **Model Consult** the lane files
refer to. Sensible proposals: cheap tier for mechanical work, mid tier as
implementer default, a different vendor as reviewer. **Expensive tier** (xhigh/max
effort, multi-hour runs, parallel heavy dispatch) ALWAYS needs explicit owner
confirmation, every time, even after the brief. One brief per session; re-brief
only if scope or risk changes.

## Self-implement vs dispatch

- Trivial → the orchestrator implements directly (dispatch overhead exceeds the work).
- Standard/Heavy → dispatch by default. The orchestrator's own tokens are usually
  the scarcest; its job is plans, packets, verification, and integration — not bulk
  code writing. Self-implementing requires the owner's waiver in the Session Brief,
  with a justification for why doing it yourself beats briefing a worker (e.g. the
  fix is smaller than the packet needed to explain it).

## Preflight (before the first dispatch of each session)

For each chosen worker CLI, run its preflight line from `models/<worker>.md` — a
five-second no-op proving login is valid, the CLI executes, and the workspace is
accessible. A worker that fails preflight gets swapped BEFORE a full context is
burned on it. (Three unpreflighted sandbox failures cost real hours in a past project.)

## Context packets (the token rule)

Workers never receive the full plan, doc tree, or source dump. They receive a packet:

- Implementer packet (`templates/packet-implementer.md`): task, success criteria,
  contract excerpt, allowed files, forbidden actions, verify commands, return
  format + return-size cap.
- Reviewer packet (`templates/packet-reviewer.md`): ≤20-line plan summary, full
  release diff + stats, risk triggers, gate receipt, known limitations, findings
  format + cap.

Full docs stay available on demand — a worker may read specific files it needs —
but they are never pasted in by default. Cap every worker's return size.

## Verifying workers ("exit 0 is not proof of work")

After any worker completes:
1. Confirm a diff exists and touches only allowed files.
2. Run the gate yourself (or demand the receipt).
3. Read the finish report against the success criteria.
Silent no-ops are a known failure mode (see `models/grok.md`).

## Failure handling

Classify every failure: `product | test | environment | worker-session | access`.
- Retry the same failed mechanism ONCE. Then: swap worker model (one replacement
  max), or change lane (e.g. plugin route instead of raw CLI), or stop and ask.
- Sandbox/auth/quota failures are infrastructure events — count them in retro
  metrics, never as review cycles.

## Platforms (Orca, terminals, plugins)

The process is platform-independent. Orca is a supported runtime for tracked
multi-agent work (`models/orca.md`) — use it when the owner wants live visibility
or already works in it; a single-worker task doesn't need it. Never let a
platform's features add process steps the lane doesn't require. Evidence from the
last retrospective: Orca transport cost <1% of tokens; the fresh full-context
sessions attached to it were the cost.

## Commits

The orchestrator commits (with owner approval); workers never commit by default.
