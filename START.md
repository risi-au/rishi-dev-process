# START — Master Dev Harness

You are an AI dev agent. This folder is the **master harness**: the canonical process
for ALL development work — any project, any model, any platform (Claude Code, Codex,
Grok, Orca, or others). The process is the authority, not the tool you run in.

Canonical home: https://github.com/risi-au/rishi-dev-process
Local clone: `G:\PROJECTS\LEXI OS\rishi-dev-process`

> **This is the only clone.** `G:\PROJECTS\Rishi-Setup\rishi-dev-process` was retired 2026-08-03
> and deliberately emptied. Reading a harness from that path is reading nothing — use the path
> above, and do not "helpfully" restore it.
Owner: Rishi. "Owner approval" always means Rishi, explicitly, in this session.

## Protocol (in order)

0. **Check this clone is intact BEFORE following it.** `git status --porcelain` here; if
   files show deleted, `git restore .` — 2026-07-29 it was missing 42 of 49 files while
   START.md still pointed at core files not on disk.
1. **Identify the work + project.** The owner names a task and a project folder.
   ALL product work happens in the project folder — never inside this harness repo.
2. **Read the core (required):**
   - `core/PROCESS.md` — loop, triage, risk profiles, gates, review state machine
   - `core/CONDUCT.md` — how to write code (lean, secure, surgical)
   - `core/GREEN.md` — why a worker's "tests pass" is not evidence, and the cure per case
   - `core/GRAPH-TOOLS.md` — query the index before you Grep. **CodeGraph** for structural
     questions, **Serena** to edit a symbol, **graphify** for conceptual ones. Catching
     yourself about to Grep the codebase is the signal that you skipped this.
3. **Pick the lane** (owner may point you at one; otherwise triage and confirm):

| Lane | Use when |
|---|---|
| `lanes/ship.md` | **DEFAULT for Lexi (`companyos-lexi`) work** — bounded, well-understood change with no load-bearing guarantee at stake |
| `lanes/new-project.md` | Nothing exists yet; greenfield app/service/tool |
| `lanes/feature.md` | New capability in an existing project |
| `lanes/bugfix.md` | Something is broken |
| `lanes/trivial.md` | Few lines, obvious, low risk (incl. doc/config tweaks) |
| `lanes/investigation.md` | Deliverable is findings/diagnosis, not code |
| `lanes/refactor.md` | Behavior-preserving restructure or cleanup |
| `lanes/deployment.md` | Live environments, infra, releases |
| `lanes/maintenance.md` | Dependency upgrades, security patches, tooling |
| `lanes/batch.md` | **Several independent issues and an expensive deploy** — parallel sessions, one Release |

   **Before picking a lane, route the request** (`lanes/batch.md` step 0 — every session).
   One line each, then wait: **(a)** verifiable locally, or genuinely needs production?
   **(b)** one piece, or several independent ones with no shared files? **(c)** does any
   part need an owner decision — those hold without blocking the rest. Then propose: single
   lane, batch, or prod-only follow-up. A single issue never justifies batch overhead.

4. **Send the Session Brief and WAIT.** For anything above trivial: read
   `core/ORCHESTRATION.md` + `core/DISPATCH.md` + `models/REGISTRY.md`, then send the
   owner ONE blocking message — lane, size/risk, one-line approach, the Model Consult
   (worker model + effort per role; reviewers count as workers) **and the Dispatch
   Consult** (`core/DISPATCH.md`): which mode — native subagents, Codex headless,
   OmniRoute, or solo — with a one-line why, whose quota it spends, and the fallback if
   it is refused. **The mode is the owner's call, not yours**: the modes differ mainly in
   whose budget they spend, and only the owner knows what is left. No code and no
   dispatch until the owner replies.
5. **Read the project overlay.** `AGENTS.md` / `ONBOARDING.md` rules (commands,
   architecture, untouchable paths) apply ON TOP of this harness. Harness = process;
   project docs = subject matter. On conflict: project wins on project facts, harness
   wins on process; if ambiguous, ask.
6. **Work the lane.** Every lane ends with: gate → finish report → session retro
   (`core/SELF-IMPROVE.md`).

## Hard rules (all lanes, always)

- Never push to main/master directly. **The orchestrator merges its own PRs**
  (owner decision 2026-07-25) once the merge floor in `core/PROCESS.md` is met.
  Anything that fails the floor waits for the owner with a written reason.
- ONE BLOCKING checkpoint per task: the Session Brief (before any code). Ask and
  WAIT. Platform autonomy defaults never override it; only the owner's own kickoff
  message saying "autonomous" does — and then every skipped confirmation is logged
  in the finish report.
- **The owner's job is answering questions, not reviewing diffs** — see below.
- Deploy and destructive actions are always separate owner approvals.
- Expensive model tier: always stop and ask first (`core/ORCHESTRATION.md`).
- Secrets: names only in code/docs/reports; values never leave the vault/.env.
- Never claim done with a failing gate. Worker "exit 0" is not proof of work.
- Never edit `core/` or `lanes/` mid-session; propose changes via the retro instead.

## Asking the owner (the owner answers questions; they do not review diffs)

Frame every decision as a **real-world scenario with concrete examples**. Never make the
owner decode an issue number, a field name, or spec jargon to answer you.

- Bad: "D2 — which event classes count as business evidence for `sourceRefs[]` in #14?"
- Good: "You ask Lexi *why do we cap Meta CPA at $32?* — it should show why it believes
  that. Which things count as a footnote: decisions and docs you wrote, or also machinery
  like 'embeddings updated'? Here's what each looks like…"

Give 2–4 options with their real consequences, lead with a recommendation, say what you'll
do if they don't reply. Batch questions. If a decision can be made from the code, sensible
defaults, or a stated assumption — make it and say so.

## Judgment over scripts

This harness states **outcomes, constraints, and hard-won environment facts** — not a
script for how to think. Where a rule doesn't fit, say so and proceed with your reasoning
stated; don't perform ceremony that adds no safety. Every rule here caught a real defect;
if one stops earning its place, raise it in the retro rather than quietly following it.

## Token discipline (why this harness exists)

A past project burned ~7 hours on a 40-minute build: fresh sessions repeatedly
re-ingesting a 28 KB plan plus all docs and source. Structurally banned:

- Workers get **context packets** (`templates/`), never full document dumps.
- ONE full review, then focused deltas only (review state machine in PROCESS.md).
- Gate receipts are reused; unchanged gates are not rerun.
- Failed mechanisms are retried once, then swapped — never ground through.
