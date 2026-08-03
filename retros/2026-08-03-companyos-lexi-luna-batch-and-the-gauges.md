# Retro — 2026-08-03, companyos-lexi: the luna batch, and three gauges that could not fail

**Outcome:** seven fixes shipped and deployed in one Release (PR #303, `main` @ `f1c0273`). Five
parallel workers, one merge conflict and it was in an artifact file. Three new defects found and
filed to root cause. Product docs at v1.12.0.

---

## 1. The routing failure that inverted the whole experiment

The owner's goal was to spend GPT credits instead of Claude credits. The dispatch asked for
`--agent luna`. **`luna` is not an Orca agent** — Orca's registry is
`claude, codex, gemini, antigravity, opencode, pi, mimo-code, droid, grok, devin, omp`, and the
default is `claude`. So the run silently produced **four Claude workers**, spending the exact pool it
was built to protect, while an OpenAI orchestrator drove them.

`luna` is a **model** (`gpt-5.6-luna`), selected with `-m`. The agent and the model are different
axes and the naming does not signal it.

**Why it went unnoticed:** the request was accepted. Orca fell back to its default rather than
refusing an unknown agent, so the only visible signal was Claude-coloured tab icons in a UI nobody
was reading — we were supervising by `git status`, correctly, and `git status` cannot tell you which
model wrote the diff.

**Harness change proposed:** `models/REGISTRY.md` should carry the **agent-vs-model distinction
explicitly**, and every dispatch record in `templates/dispatch-ledger.md` should record *which model
actually ran*, not which was requested. Tonight's ledger said `luna-a`, `luna-b`, `luna-c` — role
nicknames — and was true about roles while silently wrong about spend.

**Generalisable:** when a runtime accepts an unrecognised value by falling back to a default, every
downstream record of intent becomes fiction. Prefer runtimes that refuse; where they do not, verify
the effect rather than the request.

---

## 2. Gate on the merge — vindicated on the first attempt

`lanes/batch.md` says gate the merge, never the branches. This run is the proof.

Seven cards, five workers, **every branch green**. The merged tree failed: one card reworded a
message and updated the test beside it in its own package, while a second assertion holding a
**copy of the same string** sat in another package, outside that card's declared file set and
invisible to its targeted test run.

No amount of per-branch rigour finds that. The rule earned its place, and the fix was to delete the
duplicated literal rather than update it — so the two cannot desynchronise again.

**Keep the rule. Strengthen the reasoning in the lane file with this example.**

---

## 3. The reviewer is evidence, not a verdict

Two adversarial review rounds through a different provider than the implementer produced
**3 real findings and 4 false ones**.

Real, and none detectable by a green test or a passing mutation receipt:
- A regression guard **loosened** (`toBeLessThan` → `toBeLessThanOrEqual`) so new code would pass. A
  mutation receipt cannot catch this — the receipt only exercises the *new* assertions.
- A budget reserve that **cancelled itself out algebraically** while reading as though it partitioned
  the budget.
- The cross-package string drift above.

False, and confidently graded BLOCKING:
- `estimateTokens(undefined)` alleged to throw or return NaN. Its signature is
  `(text: string | null | undefined)`. The reviewer wrote *"behavior unknown from this diff"* and
  then blocked on it anyway.

**Auto-merging on `VERDICT: CLEAR` ships defect 1. Auto-blocking on `VERDICT: BLOCKING` holds three
good cards.** Neither verdict is usable as a gate.

**What measurably improved the second round:** the prompt said *"speculation graded as a defect is
worse than silence — if you cannot verify a claim from the diff, say 'unverified'."* Round two said
"unverified" four times instead of inventing severity.

**Harness change proposed:** `templates/packet-reviewer.md` should require that instruction verbatim,
and `core/PROCESS.md`'s merge floor should say the review condition is met when *the orchestrator has
adjudicated every finding*, not when the reviewer returns CLEAR.

---

## 4. Slim briefs — the token lever, measured

| | Brief pointing at repo docs | Self-contained brief |
|---|---|---|
| Input tokens | **5,001,552** | **1,647,026** |
| CodeGraph calls | 6 | **9** |
| Files read wholesale | **18** | **2** |

Same model, same effort, same output quality, comparable card difficulty. The difference was
pointers to `START.md`, `CLAUDE.md`, `AGENTS.md` and a 158KB `BUILD-STATUS.md`.

`START.md` §"Token discipline" has said *"workers get context packets, never full document dumps"*
since the harness was written. **I violated it while quoting it** — the brief told the worker to read
the harness core, and it dutifully did, twice.

**Harness change proposed:** `templates/packet-*.md` should carry an explicit
**"do not read: START.md, CLAUDE.md, AGENTS.md, BUILD-STATUS.md, handoffs"** line. A rule that is
stated as a principle gets violated; a rule stated as a list does not.

---

## 5. A worker reset its own branch

One session un-committed four commits back into its working tree to reorganise them. Legitimate — but
the branch had **never been pushed**, so for a period those four commits existed only in that
worktree's reflog. A crash, or a scheduled worktree reap, would have destroyed a night's work.

**Harness change proposed:** `lanes/batch.md` lane rules should add **"workers push after every
commit"**. `origin` is then always the floor and the orchestrator never depends on a worktree
surviving.

---

## 6. Stale adapters cost real time

- `models/omniroute.md` documents seven `sprint-*` combo names. The router serves `lexi-*`. Its
  documented listing command greps `'"id":"sprint[^"]*"'` and **silently returns nothing**. Three
  files carry the stale naming.
- The global Codex `AGENTS.md` instructed agents to start with `code-review-graph`, **uninstalled
  since 2026-08-02**, and its fallback clause explicitly authorised reading files instead. Its
  PostToolUse hook was a no-op that still printed *"Updating code-review-graph"* on every tool call —
  a status line for work that could not happen. Corrected to CodeGraph + Serena, with a real index
  keeper installed.

**Generalisable:** an adapter that names a tool which no longer exists does not fail — it routes work
to the expensive fallback and reports nothing. Adapters need the same "what does this say when it has
measured nothing?" test as any other gauge.

---

## 7. The pattern, a third night running

Three defects found tonight, none reported by anyone, all the same shape — **something that fails or
succeeds without saying so**:

- A badge rendering a query's `.limit(50)` as a count of degraded components, **pegged at its own
  maximum for weeks**, so the only health signal in the persistent chrome carried no information and
  everyone learned to ignore it.
- Behind it, a brain failing on **any document containing a code block** since at least 2026-07-27,
  because a non-greedy fence regex sliced valid JSON at an inner fence.
- Its error message embedding **9,685 characters of its own input**, rendered 21 times, which was the
  entire 50KB page.

And a fourth, pre-emptively: the backup health check knows only that the job *reported a run*. Once
backups are enabled it will go green for a container that says "I ran" and uploads nothing.

**The question that found all four:** *what does this gauge say when it has measured nothing?* If the
answer is indistinguishable from "everything is fine", that is the bug.

**Harness change proposed:** add that question to `core/GREEN.md` as a fifth way green lies — not
"the test measures nothing" (already there as M), but **"the indicator cannot express failure"**.
