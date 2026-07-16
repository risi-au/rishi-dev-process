# Session Report: CompanyOS Debug Setup & Durable Work Process

**Date:** 2026-07-15  
**Workspace:** `C:\Users\rishi\orca\workspaces\companyos\Debug-Setup`  
**Branch:** `feature/debug-setup`  
**Repo:** `https://github.com/risi-au/companyos`  
**Owner:** Rishi (risi-au)  
**Report location:** `G:\PROJECTS\Rishi-Setup\rishi-dev-process\`

---

## 1. Session purpose

Establish an **easy, reusable, secure** way to work on CompanyOS (COS) for:

- New features
- Bug fixes
- Growing queues of work (SaaS-style backlog)

Constraints and goals:

- Process must be **platform / agent independent** (Orca, Claude Code, Codex, Grok, etc.)
- Agents should decide **self-implement vs orchestrate**
- External agents used mainly for **usage / multi-subscription** economics
- Expensive models (e.g. codex xhigh) require **owner confirmation**
- Final UX: open a chat, point the agent at a markdown entry file (and optionally a GitHub issue), and the agent knows the next steps
- Board for features + bugs (queue, triage, work one-by-one)

Starting process material you provided: **TRIP**  
(`plan -> implement -> gate -> review -> release`) plus CompanyOS hard rules and finish-report requirements.

---

## 2. Chronology of the session

### 2.1 Why PR creation failed in Orca (start of session)

**Symptom:** Unable to create a pull request from Orca in this worktree.

**Investigation:**

| Check | Result |
|--------|--------|
| Branch | `feature/debug-setup` |
| Working tree | Clean |
| Remote | `https://github.com/risi-au/companyos.git` |
| `gh` auth | Logged in as `risi-au`, scopes include `repo` |
| Orca status | Running; worktree registered as "Debug Setup" |
| Compare to `main` | **Identical** — `ahead_by: 0`, `behind_by: 0`, status `identical` |

**Root cause:** The branch had **no unique commits** (and no uncommitted changes) vs `main`. GitHub/Orca had nothing to open a PR about. Not an auth or Orca registration bug.

**Lesson:** Create/push at least one differing commit before expecting Create PR to work.

---

### 2.2 Process design (plan mode, Q&A)

You requested: read the repo process, review TRIP, design a durable workflow, ask many questions, short responses, one step at a time.

#### Decisions locked

| Topic | Decision |
|--------|----------|
| Work board | **GitHub Issues** |
| Relation to existing docs | Process becomes **canonical**; open to improving TRIP (not freeze a parallel system) |
| Where process lives | **Both:** canonical docs in main repo; Debug-Setup can be a personal ops checkout |
| Default agent topology | **Decide per task** (self vs orchestrate) |
| Entry file | **Rewrite `ONBOARDING.md`** as sole start |
| Expensive models | **Confirm only** expensive tier |
| OpenWiki | **Skip for now** (see §4.1) |
| Skills / external repos | **Fold patterns in-repo**; optional Claude Code plugin; not full plugin installs |
| Commits | **Coordinator commits**; workers never by default |
| Human merge | Owner merges PRs to `main` |

#### OpenWiki recommendation (agreed)

COS already has:

- Per-module `AGENTS.md`
- `docs/DESIGN.md`, `docs/CONSTITUTION.md`, primer
- Brain / wiki surfaces in the product

Adding [langchain-ai/openwiki](https://github.com/langchain-ai/openwiki) would create a second doc tree (`openwiki/`) that can **drift**. Not justified at ~14 modules / ~250 TS files yet. Filed as future issue if agent navigation becomes painful.

---

### 2.3 Implementation (approved plan)

Shipped as docs/process on branch `feature/debug-setup`.

**PR:** https://github.com/risi-au/companyos/pull/51  
**Title:** Docs: TRIP work process for features and bugfixes  

#### Files created or rewritten

| Path | Role |
|------|------|
| `ONBOARDING.md` | Sole agent entry: triage, board, gates, routing, memory routing |
| `docs/ORCHESTRATION.md` | TRIP loop, roles, review verdicts, finish report, brief template |
| `docs/MODEL-POLICY.md` | Cheap / mid / expensive; confirm expensive |
| `docs/CONSTITUTION.md` | Added §11 Lean ladder, §12 Agent conduct (Karpathy-adapted) |
| `docs/SUBAGENTS.md` | Linked triage + MODEL-POLICY; orchestrator steps updated |
| `AGENTS.md` | Points to ONBOARDING first |
| `docs/tasks/_TEMPLATE-feature.plan.md` | Non-trivial feature plan template |
| `docs/tasks/_TEMPLATE-bugfix.plan.md` | Repro → minimise → fix → regression |
| `docs/ops/COCKPIT.md` | Day-to-day short path |
| `docs/OPTIONAL-CLAUDE-CODEX.md` | Optional Claude Code + Codex plugin |
| `.github/ISSUE_TEMPLATE/feature.yml` | Feature issue form |
| `.github/ISSUE_TEMPLATE/bug.yml` | Bug issue form |
| `.github/ISSUE_TEMPLATE/config.yml` | Blank issues enabled |

#### GitHub labels applied on `risi-au/companyos`

- Type: `feature`, `bug`
- Triage: `needs-triage`, `ready`, `blocked`
- Size: `trivial`, `standard`, `heavy`
- Process: `needs-plan`, `in-progress`, `in-review`

#### Future issue filed

- https://github.com/risi-au/companyos/issues/52 — evaluate OpenWiki if agent navigation becomes painful

---

### 2.4 Merge blocked in Orca (conflicts)

**Symptom:** Orca PR panel: "Conflicts block this PR" — conflicting file `ONBOARDING.md`, 2 commits behind main.

**Cause:** `main` advanced with:

- #49 — FIX-bell-badge-refresh (NotificationBell)
- #50 — memory-routing rule for all agents + promote private-memory learnings to shared docs

**Resolution:** Merged `origin/main` into `feature/debug-setup`, rewrote `ONBOARDING.md` to keep TRIP **and** fold in:

- Full env / verify-bot / PGlite / Plane gotchas from main
- Dispatch allowlist rule (commands must start with `grok` / `codex` / dispatch script)
- § memory routing (shared layer first; private memory is a cache)

**Result:** PR became **MERGEABLE**; CI `check` may run after push.

---

### 2.5 Where to open Orca for future work

**Answer given:**

- **Default:** main **companyos** Orca project (not Debug-Setup forever)
- **Per ticket:** new worktree / branch (`task/<slug>` or `fix/<slug>`)
- **Prompt:** `Read ONBOARDING.md. Do issue #N. Confirm expensive models with me.`
- **Debug-Setup:** optional long-lived checkout for process/docs experiments — **not** required for normal feature/bug work
- **Board** = GitHub Issues; **workspace** = one worktree per ticket

---

### 2.6 Loop engineering article (Karpathy method)

Reviewed: [@0xCodila](https://x.com/0xCodila/status/2072329149520232639) — *Loop Engineering: The Karpathy Method - and the workflow that just made it 5x better*.

#### Article thesis

- Most people still use AI like 2005 Google (prompt → answer → prompt).
- High leverage is **loops**: goal, agent works until done, not human as engine each turn.
- Three critical parts: **verifier**, **state**, **stop condition**.
- **Karpathy Loop** (AutoResearch): agent may touch train code; evaluator is frozen; metric gates keep/rollback; run overnight.
- Five building blocks: automation heartbeat, skills, sub-agents (maker ≠ checker), connectors, verifier.
- **Bilevel** meta-loop: outer loop improves how inner loop searches (paper claim ~5× on a training metric).
- Honest risks: **comprehension debt**, **cognitive surrender**.

#### Fit to COS

| Idea | Already in TRIP? | Recommendation |
|------|------------------|----------------|
| Verifier | Yes — typecheck/lint/test | Keep non-negotiable |
| State | Plans, briefs, handoffs, issues | Tighten attempt/status lines |
| Stop condition | Review cap 5, LIMIT-ALERT | Optional: max implement rounds (e.g. 3) then stop |
| Maker ≠ checker | Orchestrator / implementer / fresh review | Keep |
| Skills | ONBOARDING, CONSTITUTION, module AGENTS | Keep; no parallel skill zoo |
| Overnight AutoResearch | No product single metric | **Skip** for features/bugs |
| Bilevel meta-loop | — | **Skip** for now |

**Good COS loops later:** implement → gate → fix (max N); CI red repair; mechanical doc/encoding sweeps.  
**Bad loops:** product design, multi-module architecture, anything without a machine check, unattended overnight product code.

---

## 3. The TRIP process (canonical summary)

```text
INTAKE (GitHub Issue)
  -> TRIAGE (trivial | standard | heavy; self vs orchestrate)
  -> PLAN (required if non-trivial; docs/tasks/)
  -> IMPLEMENT (branch task/* or fix/*; implementer usually)
  -> GATE (pnpm typecheck && pnpm lint && pnpm test)
  -> REVIEW (fresh model for non-trivial; verdicts below)
  -> RELEASE (PR to main; human merges; never push main)
```

### 3.1 Triage

| Class | Criteria | Who codes | Plan file? |
|-------|----------|-----------|------------|
| Trivial | Few lines, obvious, low risk, one module | Same agent may implement | Optional |
| Standard | Multi-file or behavior change | Dispatch implementer | Required |
| Heavy | Schema/API/kernel, multi-module, security | Dispatch; mid/expensive per policy | Required |

### 3.2 Review verdicts

- `APPROVED`
- `REQUEST_CHANGES`
- `NEEDS_REWORK`  
Cap **5** rounds. Critical/Major block. Drive-by edits / overbuild = Major.

### 3.3 Finish report (required when claiming done)

```text
Files changed: (one line each)
Deviations from plan: ...
Left undone: ...
Gate: lint: ... | typecheck: ... | tests: N passed
```

Never claim done with a failing gate.

### 3.4 Commits and merge

- Default: **orchestrator commits**; implementers do not
- PR links `Fixes #N` / `Closes #N`
- **Owner merges**

### 3.5 Model policy (tiers)

| Tier | Use | Confirm? |
|------|-----|----------|
| Cheap | Mechanical, docs, simple fix, explore | No |
| Mid | Default implementer (e.g. codex medium / gpt-5.5 medium) | No |
| Expensive | xhigh / max effort / long multi-agent rescue | **Yes — stop and ask** |

Details: `docs/MODEL-POLICY.md` in the repo.

---

## 4. External repositories and references reviewed

### 4.1 [langchain-ai/openwiki](https://github.com/langchain-ai/openwiki)

- CLI that generates/maintains agent wikis for codebases (and personal mode).
- Code mode writes under `openwiki/`, can inject markers into `AGENTS.md` / `CLAUDE.md`.
- **Decision:** skip now; file #52 for later evaluation.
- **Why:** COS already has structured agent docs + product wiki/brain; second tree risks drift.

### 4.2 [mattpocock/skills](https://github.com/mattpocock/skills)

- Engineering skills: grill-with-docs, triage, to-spec, implement, TDD, diagnosing-bugs, code-review, etc.
- Issue-tracker aware (GitHub/Linear/local).
- **Decision:** do **not** install whole plugin set.
- **Adopted as patterns:** triage via GitHub Issues; grill/assumptions before coding; bugfix repro→minimise→test; dual-axis review ideas folded into TRIP/plans.

### 4.3 [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)

- "Lazy senior" ladder: YAGNI → reuse → stdlib → deps → minimum.
- Claims large LOC/cost reductions while keeping safety.
- **Decision:** do not install full multi-agent plugin.
- **Adopted:** **Lean ladder** as CONSTITUTION §11 (always-on, platform-agnostic text).

### 4.4 [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc)

- Claude Code plugin: `/codex:review`, `/codex:adversarial-review`, `/codex:rescue`, status/result/cancel.
- **Decision:** optional only when using Claude Code.
- **Documented in:** `docs/OPTIONAL-CLAUDE-CODEX.md`.
- Process itself stays CLI-agnostic (`docs/SUBAGENTS.md` for `codex exec` / grok).

### 4.5 [multica-ai/andrej-karpathy-skills — karpathy-guidelines](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/skills/karpathy-guidelines/SKILL.md)

Four behaviors (MIT):

1. Think before coding (surface assumptions)
2. Simplicity first
3. Surgical changes (no drive-by)
4. Goal-driven execution (verifiable checks)

**Decision:** fold into CONSTITUTION §12 **Agent conduct** (not vendor whole skill pack).

### 4.6 [trendshift.io](https://trendshift.io/)

- Surfaced as a discovery surface for trending agent tools.
- Session shortlisted only the repos above; no broad trendshift adoption.

### 4.7 [@0xCodila loop engineering / Karpathy loop](https://x.com/0xCodila/status/2072329149520232639)

- See §2.6.
- Adopt mindset of verifier + state + stop; skip overnight product AutoResearch and bilevel meta-loops for now.
- References Karpathy AutoResearch-style setup and a "Bilevel Autoresearch" paper concept.

### 4.8 Existing COS process (pre-session baseline)

Already in repo before this session:

| Doc | Role |
|-----|------|
| `docs/CONSTITUTION.md` | Hard engineering rules |
| `docs/ORCHESTRATION.md` | Architect + implementer (milestone-era) |
| `ONBOARDING.md` | Agent start (always-orchestrate bias) |
| `docs/SUBAGENTS.md` | Grok/codex failure modes, dispatch |
| `scripts/dispatch-codex.ps1` | One-command worktree + dispatch |
| `docs/tasks/*` | Many milestone/UX briefs |

Session **evolved** these into TRIP + Issues board + per-task triage rather than inventing a second process tree.

---

## 5. CompanyOS hard rules (carried into process)

Violations = rejected work (from your brief + CONSTITUTION):

- Modules never import each other's service files; logic in `packages/api`
- Every write emits an event
- Migrations: drizzle-kit; never hand-edit journal; plain SQL; no live DB migrations from tasks; enum caveats
- Untouchable: `USER DATA/`, `legacy/`, `.env*`, `vps-login.txt`
- Secrets: names only; values in credential vault
- UI: design tokens only; server actions for mutations; `@/lib/api` wrappers
- Windows: plain ASCII in source string literals; no BOMs
- Do not invent scope structure / parallel process docs; ask owner when decision is product-level

---

## 6. How you run day-to-day after merge

1. **File work** on GitHub Issues (`feature` or `bug` template).
2. Open Orca on **main companyos project**.
3. Create a **worktree** for the ticket (`task/...` or `fix/...`).
4. Start agent with:

```text
Read ONBOARDING.md. Do issue #N. Confirm expensive models with me.
```

5. Agent should:
   - Triage class + self vs orchestrate
   - State model tier (ask if expensive)
   - Write plan if non-trivial
   - Implement or dispatch
   - Run gates
   - Adversarial review if non-trivial
   - PR; **you merge**

Short path doc in repo: `docs/ops/COCKPIT.md`.

---

## 7. What was explicitly out of scope this session

- Installing OpenWiki
- Full mattpocock or ponytail plugin installs
- Product runtime code (except merge brought in #49 NotificationBell from main)
- New orchestration runtime inside the COS product
- AutoResearch / bilevel overnight loops as platform infrastructure

---

## 8. Artifacts and links

| Artifact | URL / path |
|----------|------------|
| Process PR | https://github.com/risi-au/companyos/pull/51 |
| OpenWiki later | https://github.com/risi-au/companyos/issues/52 |
| Session worktree | `C:\Users\rishi\orca\workspaces\companyos\Debug-Setup` |
| This report | `G:\PROJECTS\Rishi-Setup\rishi-dev-process\2026-07-15-companyos-debug-setup-session-report.md` |

### Canonical files after merge (on main)

- `ONBOARDING.md`
- `docs/ORCHESTRATION.md`
- `docs/MODEL-POLICY.md`
- `docs/CONSTITUTION.md` (§11 lean, §12 conduct)
- `docs/ops/COCKPIT.md`
- `docs/tasks/_TEMPLATE-feature.plan.md`
- `docs/tasks/_TEMPLATE-bugfix.plan.md`
- `.github/ISSUE_TEMPLATE/*`

---

## 9. Optional next steps (not done this session)

1. Merge PR #51 if not already merged (resolve CI if needed).
2. Add a short **implementer loop** paragraph to ORCHESTRATION (gate → fix max 3 → stop) from the Karpathy-loop article.
3. Optional GitHub Project board columns (labels may be enough).
4. Optional install of codex-plugin-cc only on Claude Code machines.
5. Revisit OpenWiki only if issue #52 criteria hit.
6. Smoke test: new chat with only ONBOARDING + a real issue number.

---

## 10. One-paragraph executive summary

This session turned a broken "empty branch" Orca PR situation into a **durable, agent-independent CompanyOS work system**: GitHub Issues as the board, rewritten `ONBOARDING.md` as the single agent entry, TRIP as the standard loop (with per-task self-vs-orchestrate triage), model cost policy with confirmation for expensive runs, plan templates for features and bugs, lean + Karpathy conduct in the constitution, and optional Claude↔Codex docs. External tools (OpenWiki, mattpocock skills, ponytail, codex-plugin-cc, Karpathy guidelines, loop-engineering article) were reviewed; patterns were folded into in-repo docs rather than installing competing platforms. Merge conflicts with main's memory-routing work were resolved. Future features and bugs should start from the main companyos project as **one worktree per issue**, not a permanent Debug-Setup-only workflow.

---

*End of report.*
