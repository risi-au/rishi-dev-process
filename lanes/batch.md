# LANE: Batch (parallel sessions, one Release)
<!-- CAP: 110 lines -->

Prereqs: START.md + core read; project overlay read. Proven 2026-08-02 on `companyos-lexi`:
six sessions, fourteen issues, zero merge conflicts, one deploy.

Use when there are **several independent issues** and the deploy is expensive. On a repo where
one Release is an 18-minute ARM build, shipping fourteen fixes one at a time costs four hours of
build time for work that took one.

**Do NOT use for a single issue.** The coordination overhead exceeds the work.

## Step 0 — Route the request (do this on EVERY session, batch or not)

Before picking a lane, answer three questions out loud for the owner:

1. **Can it be verified locally?** Most app and UI work can. Genuinely prod-only: real
   credentials, real external services, the deploy environment itself, environment-signal bugs.
2. **How many independent pieces are there?** One → normal lane. Several with no shared files →
   batch. Several that all edit the same file → sequence them, do not parallelise.
3. **Does any piece need an owner decision?** Those go on hold and do not block the rest.

Then propose the route: **local-and-batch**, **single lane**, or **prod-only follow-up**. One
line each, then wait. Do not assume the batch is wanted because it is impressive.

## Step 1 — Triage every candidate before assigning any

**Verify each issue against current source, not its own text.** On the first run of this lane,
**four of the issues were already fixed** and nobody had checked — closing them with evidence was
the cheapest work of the night. `codegraph explore` per issue; a `STALE` verdict is a real result.

Classify each: `LOCAL` / `PROD-ONLY` / `DUPLICATE-OF-#N` / `STALE` / `ROADMAP`. Epics, RFCs and
milestone captures are **roadmap, not backlog** — leaving them open is correct.

## Step 2 — Group by FILE, not by theme

This is the whole trick. Sessions collide on files, not on subject matter.

- Run `codegraph callers` on each issue's target and build the real file set.
- Any file wanted by two issues decides the grouping: both issues go to **one** session.
- A file nobody can avoid sharing (a page every feature touches) means that issue runs **alone**,
  after the batch.

Look for changes that need no shared edit at all. On the first run, one issue appeared to need a
file another session owned; rewriting a component's internals while keeping its export name and
props meant the shared file was never touched. **That trick is worth hunting for.**

## Step 3 — Write the coordination issue

One GitHub issue holds the whole plan: rules, session table with exclusive file sets, owner
decisions already made, and the integration protocol. Template:
`templates/batch-coordination-issue.md`.

**It lives in the tracker, not in chat.** If the orchestrator session dies, any session can pick
up integration from the issue. Every session brief starts with "read issue #N first".

## Step 4 — Brief and launch

One session per group, `templates/packet-session.md`. Each brief carries the triage findings —
files, call paths, and the specific traps — so the session does not re-derive them.

**Say explicitly whether the session dispatches or self-implements.** Left unstated, sessions
self-implement, which silently keeps all load on one provider. Trivial work: implement directly.
Standard/Heavy: dispatch to OmniRoute (`core/ORCHESTRATION.md`).

Model per session by **how much judgement the brief left open**, not by issue count: a heavy tier
where a design decision remains inside the work, a cheap tier where triage removed the ambiguity.

## Step 5 — Watch for drift, do not wait for reports

Check what sessions actually touch, roughly every 20 minutes:

```
for w in <worktrees>; do (cd $w && git status --porcelain); done
```

This caught two real problems within minutes on the first run: a file collision, and two sessions
running `npm install` on a `pnpm` repo. **Both were briefing errors, not sessions misbehaving** —
which is the usual finding. Fix the brief, tell the session, move on.

## Step 6 — Integrate

1. Branch from `main`, merge every session branch, resolve conflicts.
2. **Run the gate on the MERGE**, never on the branches. CI tests the merge; a branch gate on a
   moving trunk diverges from it.
3. One PR, one merge, **one Release**.
4. Verify the Release, then the running instance — merging is not deploying.

## Lane rules

- **Never let a session merge to `main`.** Six merges is six deploys, which defeats the lane.
- **One migration at a time across the whole batch.** Two concurrent migrations collide on the
  journal and break the `prevId` chain. Everything else waits or defers that piece.
- **A session that needs another session's file stops and reports.** It never edits across the
  boundary and sessions never talk to each other.
- **Closing a verified-fixed or duplicate issue counts as work.** The goal is a smaller true list,
  not a larger diff.
- Expect the briefs to be wrong somewhere. Budget attention for corrections rather than assuming
  a clean run.
