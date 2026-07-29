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
- **Enforce this mechanically where the runtime allows** — intent has failed in practice
  (2026-07-26). Deny the orchestrator's writes to source; never its reads
  (`models/claude-code-orchestrator.md`). Confirm the lock is live before dispatch one.

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
  release diff + stats, risk triggers, gate evidence, known limitations, findings
  format + cap. **A reviewer packet MUST explicitly permit read-only shell** (git
  diff/log, grep, lint, typecheck, tests). The standard hard-constraint block reads as
  "no terminal commands at all": 2026-07-25 a reviewer obeyed it literally, could not
  verify a single gate number, and its whole review was code-reading only.
- Every implementer packet states: **write in small incremental edits, never more than
  ~120 lines per edit** (`models/cline.md` — a card ignoring this died mid-write), and
  what to do when the full test suite exceeds the worker's command timeout (say so;
  never claim green; run targeted tests from the repo root).

Full docs stay available on demand — a worker may read specific files it needs —
but they are never pasted in by default. Cap every worker's return size.

If the project has code graphs, query BEFORE bulk-reading files to locate code.
Two tools may exist — do not use both for the same question:
- **graphify** (architecture / "where is X"): always pass the MAIN checkout's
  absolute `--graph` path; worktrees lack `graphify-out/` (`models/graphify.md`).
- **code-review-graph** (diff / PR impact / review risk): use the **open
  workspace** (per-worktree DB; MCP inherits cwd) — never hard-code MAIN as CRG
  cwd when in an Orca worktree (`models/code-review-graph.md`).
Put the relevant path(s) in every packet. Prefer project `AGENTS.md` when it
defines a graph split.

## Proportion — match the packet to the card (2026-07-29)

The evidence rules below are not free: a full uncached suite is minutes of wall clock per
run. Demanding it on a card that changes a few lines is the most common avoidable cost in
this harness — and one the worker cannot refuse, because the packet told it to.

Scale the demand to what is actually at risk:

| Card | Gate | Evidence |
|---|---|---|
| A few lines, no guarantee at stake, config/docs/copy | Targeted tests for the touched area + lint/typecheck | State what you ran. No fail-before/pass-after ceremony for a change with no behaviour to prove. |
| Ordinary bounded change | Full uncached gate once | Fail-before/pass-after on the specific behaviour changed |
| Migration, auth, permissions, trust semantics, MCP contract | Full uncached gate + verify the artifact, not the gate | Everything: pre-fix proof, schema queried not ledger, confirm the test RAN not skipped |

**Two traps worth naming in the packet itself:**

- **Pre-fix proof on a brand-new field proves nothing.** `expect(result.ok)` fails before the
  fix because `ok` does not exist yet — that demonstrates the field is new, not that behaviour
  changed. Demand the pre-fix assertion be on *behaviour that already had a shape*.
- **A known flake must be named as out of scope**, with the file and the reason. Otherwise a
  worker burns a pass chasing it, or worse, "fixes" it by loosening the test.

This is not permission to skip gates on risky work — proportion runs both ways, and the
heavy column is mandatory when `lanes/ship.md`'s triggers apply.

## Verifying workers ("exit 0 is not proof of work")

After any worker completes:
1. Confirm a diff exists and touches only allowed files.
2. Run the gate yourself (or demand the receipt).
3. Read the finish report against the success criteria.
Silent no-ops are a known failure mode (see `models/grok.md`).
**A worker's "green" is a claim about a measurement it authored.** `core/GREEN.md` names the
four ways it lies and the cure for each — read it before trusting any batch.
Log every dispatch to `templates/dispatch-ledger.md` **at dispatch time**, so delegation and
token hotspots are checkable facts rather than end-of-session recollection.

## Failure handling

Classify every failure: `product | test | environment | worker-session | access`.
- **A worker that dies with a connection error → re-run its preflight before blaming
  the packet.** A dead router/service makes every dispatch fail in seconds and looks
  exactly like a bad prompt (`models/omniroute.md`, 2026-07-25).
- Retry the same failed mechanism ONCE. Then: swap worker model (one replacement
  max), or change lane (e.g. plugin route instead of raw CLI), or stop and ask.
- When dispatch is blocked but the orchestrator is not, keep the non-dispatch work
  moving (gates, review, commits, PRs) and say plainly what is blocked and why.
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
