# START — Master Dev Harness

You are an AI dev agent. This folder is the **master harness**: the canonical process
for ALL development work — any project, any model, any platform (Claude Code, Codex,
Grok, Orca, or others). The process is the authority, not the tool you run in.

Canonical home: https://github.com/risi-au/rishi-dev-process
Local clone: `G:\PROJECTS\Rishi-Setup\rishi-dev-process`
Owner: Rishi. "Owner approval" always means Rishi, explicitly, in this session.

## Protocol (in order)

1. **Identify the work + project.** The owner names a task and a project folder.
   ALL product work happens in the project folder — never inside this harness repo.
2. **Read the core (required):**
   - `core/PROCESS.md` — loop, triage, risk profiles, gates, review state machine
   - `core/CONDUCT.md` — how to write code (lean, secure, surgical)
3. **Pick the lane** (owner may point you at one; otherwise triage and confirm):

| Lane | Use when |
|---|---|
| `lanes/new-project.md` | Nothing exists yet; greenfield app/service/tool |
| `lanes/feature.md` | New capability in an existing project |
| `lanes/bugfix.md` | Something is broken |
| `lanes/trivial.md` | Few lines, obvious, low risk (incl. doc/config tweaks) |
| `lanes/investigation.md` | Deliverable is findings/diagnosis, not code |
| `lanes/refactor.md` | Behavior-preserving restructure or cleanup |
| `lanes/deployment.md` | Live environments, infra, releases |
| `lanes/maintenance.md` | Dependency upgrades, security patches, tooling |

4. **Send the Session Brief and WAIT.** For anything above trivial: read
   `core/ORCHESTRATION.md` + `models/REGISTRY.md`, then send the owner ONE
   blocking message — lane, size/risk, one-line approach, and the Model Consult
   (worker model + effort per role; reviewers count as workers). No code and no
   dispatch until the owner replies.
5. **Read the project overlay.** If the project has `AGENTS.md` / `ONBOARDING.md`,
   its project-specific rules (commands, architecture, untouchable paths) apply ON
   TOP of this harness. Harness = process; project docs = subject matter. On a true
   conflict: project wins on project facts, harness wins on process; if genuinely
   ambiguous, ask the owner.
6. **Work the lane.** Every lane ends with: gate → finish report → session retro
   (`core/SELF-IMPROVE.md`).

## Hard rules (all lanes, always)

- Never push to main/master. The owner merges PRs.
- Two BLOCKING checkpoints per task: the Session Brief (before any code) and the
  release approval (commit+push+PR may be batched into one itemized ask). Ask and
  WAIT. Platform autonomy defaults never override these; only the owner's own
  kickoff message saying "autonomous" does — and then R2 work still blocks, and
  every skipped confirmation is logged in the finish report.
- Deploy and destructive actions are always separate owner approvals.
- Expensive model tier: always stop and ask first (`core/ORCHESTRATION.md`).
- Secrets: names only in code/docs/reports; values never leave the vault/.env.
- Never claim done with a failing gate. Worker "exit 0" is not proof of work.
- Never edit `core/` or `lanes/` mid-session; propose changes via the retro instead.

## Token discipline (why this harness exists)

A past project burned ~7 hours on a 40-minute build because fresh sessions
repeatedly re-ingested a 28 KB plan plus all docs and source. Structurally banned:

- Workers get **context packets** (`templates/`), never full document dumps.
- ONE full review, then focused deltas only (review state machine in PROCESS.md).
- Gate receipts are reused; unchanged gates are not rerun.
- Failed mechanisms are retried once, then swapped — never ground through.
