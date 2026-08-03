# TEMPLATE: Batch coordination issue
<!-- File this as ONE GitHub issue before launching a batch. See lanes/batch.md.
     It lives in the tracker, not in chat: if the orchestrator session dies, any
     session can pick up integration from here. Every session brief opens with
     "read issue #N first".
     Below is the real issue from the first run (companyos-lexi #279, 2026-08-02) —
     six sessions, fourteen issues, zero conflicts. Replace the specifics; keep the
     shape, and keep the reasons attached to the rules. A rule with its reason
     survives contact with a session that thinks it knows better.

     VENDOR NOTE: the example names Claude Code, Orca and specific model tiers
     because that is what the first run used. This harness is platform-independent
     (README.md) and core/PROCESS.md actively prefers a DIFFERENT vendor for review
     than the one that implemented. Substitute whatever models/REGISTRY.md lists as
     active — the lane itself assumes nothing about vendor. Worktrees, file ownership,
     one-migration-at-a-time and gate-on-the-merge are git and CI facts, not model
     behaviour, so they hold for Codex, Grok, Cline or a mix. -->

Coordination issue for the overnight batch. **Any session may read this; only the orchestrator acts on the integration section.** If the orchestrator session dies, another can pick up integration from here — that is why this is an issue and not chat context.

## Rules every session follows

1. **Start in your OWN git worktree, never in the `main` checkout.** `main` is occupied by another
   session and git allows only one branch per checkout — taking it would yank the ground out from
   under that work. Each worktree also gets its own CodeGraph index, built automatically at session
   start (~4s):
   ```
   git worktree add ../night-s1 -b night/s1-scope-page main
   ```
   Expect a separate install per worktree, and give each session its own `PORT` (3001-3006).

   ⚠️ **This repo is `pnpm@11.1.3`. Use `pnpm install` — NEVER `npm install`.** Two sessions ran
   npm on 2026-08-02 and produced a stray `package-lock.json`, which resolves dependencies
   differently from `pnpm-lock.yaml` and must never reach a commit. If you see a
   `package-lock.json` in your worktree, delete it. (`npm run dev` is fine — it is `pnpm`'s
   install that matters.)

   **Running under Orca?** Orca creates the worktree for you (at
   `C:\Users\rishi\orca\workspaces\companyos\<task>`) — skip the `git worktree add` command above;
   everything else applies. Three Orca quirks worth knowing (`models/orca.md`): `orca` returns
   **exit 255 even on success** from PowerShell, so parse the JSON `.ok` field rather than the exit
   code; never run `check --wait` as a PowerShell background job (heartbeats trip
   `NativeCommandError` and it is killed around 600s — run it from Bash); and after
   `dispatch --inject`, **send a follow-up Enter**, because the prompt can land mid-boot and sit
   unsubmitted. A session that never started looks exactly like a session that is thinking.

   CodeGraph self-builds in an Orca worktree (the hook is global). **graphify does not** — worktrees
   never get `graphify-out/`, so any graphify query must use MAIN's absolute path.
2. **You own a branch and a file set. Do not touch files another session owns.** The groupings below exist because three issues all edit `apps/os/src/app/(app)/s/[...path]/page.tsx`; that file belongs to S1 alone.
3. **Never merge to `main`.** Never open a PR against `main`. Push your branch and stop.
4. **Never merge another session's branch.** Sessions do not talk to each other.
5. Follow `G:\PROJECTS\LEXI OS\rishi-dev-process\START.md`. Lane per issue label; the Session Brief to the owner is **not** required for these — the owner has already approved scope and worker (this issue is that approval).
6. **Use CodeGraph before reading files.** `codegraph explore "<task>"` gives allowed files, call paths and blast radius in one call. Check `codegraph status` looks like ~550 files first; if it is small, run `codegraph init .`.
7. **Use Serena for symbol-shaped edits** (`rename_symbol`, `replace_symbol_body`). Do **not** trust Serena's `find_referencing_symbols` for blast radius — it misses cross-package references through `@companyos/*` aliases. CodeGraph for scope, Serena for the edit.
8. **Verify locally.** Postgres is up (`companyos-postgres`, project `lexi-dev`, fully migrated 57/57). `DATABASE_URL=postgres://companyos:devpassword123@localhost:5432/companyos`. Run the app with your own port: `PORT=300N npm run dev`.
9. **Gate before reporting:** lint, typecheck, and the tests `codegraph affected <your changed files>` names. State exactly what you ran. Never claim the full suite unless you ran it.
10. **Report and stop.** Do not deploy, do not merge, do not touch the VPS.

## Worker routing (OmniRoute)

Live combos are `lexi-scout`, `lexi-implement`, `lexi-implement-hard`, `lexi-review`, `lexi-secure`. Preflight: `curl -s -m 8 -o /dev/null -w "%{http_code}\n" http://localhost:20128/api/v1/models` must return 200.

> Note: `rishi-dev-process/models/omniroute.md` documents `sprint-*` combo names. Those are **stale** — the router serves `lexi-*` and `auto/*`. Adapter needs updating.

### DISPATCH, do not self-implement (corrected 2026-08-02)

**You are an orchestrator, not the implementer.** `core/ORCHESTRATION.md`: *"Standard/Heavy →
dispatch by default. The orchestrator's tokens are the scarcest; its job is plans, packets,
verification, and integration — not bulk code writing."*

The first wave of this batch self-implemented because the briefs did not say otherwise. That was
an unstated waiver and it left OmniRoute idle while every session burned one provider's quota.
Do not repeat it.

- **Trivial cards** (a few lines, one file) — implement directly. Dispatch overhead exceeds the work.
- **Standard** — dispatch to `lexi-implement`.
- **Heavy** (migrations, auth, permissions, trust semantics) — dispatch to `lexi-implement-hard`,
  then a **different** worker on `lexi-review` before you accept the diff.

Use `templates/packet-implementer.md`. Each packet carries: task, success criteria, ALLOWED FILES,
the call paths from `codegraph explore`, forbidden actions, verify commands, and a return-size cap.
Never paste the full plan or a source dump — that pattern cost 35% of a past project's tokens.

**You still own verification. "exit 0 is not proof of work":** confirm a diff exists, that it
touches only allowed files, and run the gate yourself rather than trusting the worker's claim.

## Sessions

| # | Branch | Issues | Model | Owns exclusively |
|---|---|---|---|---|
| **S1** | `night/s1-scope-page` | #265, #186, #245 | **Opus** | `s/[...path]/page.tsx`, `tasks/service.ts`, `kernel/grants.ts`, `modules/members/`, **+ the four #186 consumers below** |
| **S2** | `night/s2-sidebar-chat` | #268, #270, #269 | Sonnet | `_components/Sidebar.tsx`, `modules/docs/DocsView.tsx`, `modules/agent/` |
| **S3** | `night/s3-health` | close #252, close #212, #152 slice | Sonnet | `modules/health/service.ts`, `lib/ops-health.ts` |
| **S4** | `night/s4-memory-eval` | #115, #111, #90 | Sonnet | `modules/memory/`, `modules/eval/` |
| **S5** | `night/s5-provisioning` | #250, #209 | **Opus** | `modules/provisioning/`, `db/schema/kernel.ts` |
| **S6** | `night/s6-onboarding` | #243, #241, #244 | **Opus** | `modules/first-run/`, `modules/invitations/`, `modules/admin/InvitationCreateForm.tsx` |

**Model choice rationale:** the session model is the *orchestrator* — it plans, dispatches to
OmniRoute workers, and judges the result. It is not writing the bulk of the code. So the question is
only how much judgement the brief left open. Opus where a real design decision remains inside the
work (S1 chooses a carrier shape across five call sites; S5 needs a new identifier plus a migration;
S6 invents a permission that does not exist yet). Sonnet where triage removed the ambiguity and the
task is execution against a named file list (S2, S3, S4). If a Sonnet session finds itself designing
rather than executing, that is the signal to stop and escalate, not to push on.

**S1 ownership extended 2026-08-02** to `packages/mcp/src/server.ts`,
`apps/os/src/modules/dashboards/DashboardGrid.tsx`, `packages/api/src/modules/digest/service.ts`
and `packages/api/src/modules/agent/service.ts`. Reason: S1's first commit added
`listTasksWithStatus` and rewired only `page.tsx`, leaving the other four #186 consumers still
calling the old `listTasks` — so a Plane outage still makes the digest state *"no open tasks"* as
fact. The original ownership row was too narrow; that was a briefing error, not S1 straying.
**No other session may touch those four files.**

**#248 (timestamps) is deliberately unassigned** — it touches 9+ files across several sessions' territory. It runs alone after the batch merges, or it will conflict with everything.

## Owner decisions already made (2026-08-02) — do not re-open these

- **#248** timestamps render in the **viewer's own timezone, with the zone shown**. One shared helper. No DB field, no admin screen.
- **#244** email verification **gets turned on**, then resend + auto-retry is built. Note `requireEmailVerification` is currently `false` and pinned by a test — that test changes.
- **#241** personal scope is **on by default** for everyone; project grants optional; **"can create projects" becomes its own permission**, independent of admin-on-root. See the rewritten issue.
- **#265** activity rows show a **friendly name plus a short sentence per event type**. Not raw payload.
- **#269** **restore Ask Lexi.** This knowingly reverses D13 (#31 / PR #91); record that on #31.
- **#242** wizard step, personal connections ticked by default, agents explicit. **Not in this batch** — see the rewritten issue.

## Integration protocol (orchestrator only)

1. Each session reports; the owner relays the report to the orchestrator.
2. Orchestrator creates `integration/2026-08-03-night` from `main`, merges each `night/s*` branch in the order they finish, resolving conflicts.
3. Run the full gate on the integration branch — **not** on the individual branches. CI tests the merge; a branch gate tests the branch, and on a moving trunk those diverge.
4. **Check no `package-lock.json` slipped in** before merging (see rule 1).
5. **One PR, one merge, one Release.** Do not merge session branches to `main` individually — each merge is an ~18-minute ARM build and a production deploy.
6. Verify the Release: all three jobs green, then check `lexi.risi.au` directly.
7. Update the product documentation (`Brissie-Digital-PTY-LTD/lexi-live-os-documentation`) per `CLAUDE.md`'s closing ritual, before declaring done.

## Outstanding, owner-only

- **#273** — VPS `~/lexi/.env` needs `BRAIN_CRON_ENABLED=true` and `COMPOSE_PROFILES=brain`, then recreate the sidecar. Code already deployed in #277. **In progress.**
- **#197** — revoke the four non-human root grants. Sequence **after** #209 lands, or the accumulation re-creates itself.



