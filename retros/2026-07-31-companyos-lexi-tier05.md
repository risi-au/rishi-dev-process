# Retro — companyos-lexi Tier 0.5 (2026-07-31)

Shipped: #237 (`f422b96`) and #235 (`0a6fc41`), both deployed. Product doc v1.4.0 merged.
Ten defects found on #237. **None by a failing gate.**

---

## What caught what

| Defect | Caught by |
|---|---|
| A credential could reach every tool response | orchestrator reading the diff |
| The note forged TSV rows in `recall_memory` | an existing test the packet forbade touching |
| Elicitation could block a tool call for 60s | orchestrator reading the diff |
| Name recovered by regex-scraping rendered prose | orchestrator reading the diff |
| Pointer rode `ping` (no auth required) | orchestrator reading the diff |
| "Does not block" test ran with zero proposals | the worker itself, mid-run |
| `AGENTS.md` dropped the brief on the onboarding path | orchestrator reading the diff |
| Identity resolved twice per call (F1, R1) | adversarial review |
| The nudge never stopped; repeated verbatim | four-lens review, "what is missing?" |
| Identity guard left broken by a dead worker | a regression test the review had forced into existence |

`core/GREEN.md`'s budget held exactly: review and orchestrator reading find the defects; more
worker-run tests find none of them.

---

## Proposed changes to the harness

### 1. `core/GREEN.md` — name the mutation-receipt hazard

**Add:** *The mutation receipt deliberately breaks working code. If the worker dies between breaking
and restoring, it leaves the guard broken and everything green. After ANY interrupted worker,
verify every line its receipts claim to have restored — the gate cannot see a missing guard that
nothing else tests.*

This happened: a worker broke the identity guard, hit its quota, and died. Code compiled, typecheck
passed, every other test passed. One regression test caught it — and that test existed only because
a reviewer had found the same hole hours earlier.

### 2. `core/ORCHESTRATION.md` — a too-tight allowed-files list backfires

**Add:** *Scope the allowed-files list to where the work belongs, not to the smallest possible
surface. A worker locked out of the module holding the safe helper will re-implement it badly rather
than reach for it.*

Card A was restricted to `packages/mcp`. The sanctioned tool-inventory reader lives in
`packages/api`. The worker wrote its own, with no credential filtering, and nearly shipped a leak.
The restriction did not prevent a bad edit; it caused one.

### 3. `models/codex.md` — two dispatch failures

- **Inlining a diff breaks past ~30 KB on Windows.** `Argument list too long`, surfacing as a Codex
  error rather than a shell limit. Write the packet to a file and pass a pointer prompt.
- **"Run no commands" makes a reviewer refuse to read its own packet.** It returned
  `VERDICT: BLOCKED`. `V2 - LEXI/CARRIED-CONTEXT.md` §7 already says review packets must explicitly
  *permit* read-only shell; the adapter should repeat it, because the prohibition wording is the
  natural thing to write.

### 4. New adapter — `models/opencode.md`

`opencode run --model <provider>/<model> --auto --dir <path> "<prompt>"`. Authenticated separately
from OmniRoute; also fronted by OmniRoute as `opencode-go/*` and `oc/*`.

**Two hard facts:**

- **`TaskStop` does not kill it.** It kills the shell wrapper; the worker keeps editing. This
  session stopped a card, checked `git status`, concluded the tree was clean, and the worker
  subsequently wrote to seven files. Kill the PIDs and re-check the tree.
- **Quota exhaustion kills it mid-edit**, which is how the broken guard above happened. Check
  remaining quota before dispatching a card with mutation receipts in it.

### 5. `core/PROCESS.md` — four lenses, not four reviewers

**Add to the review section:** *Where budget allows, run several reviewers with DIFFERENT questions
rather than several copies of one. Redundancy finds nothing new; diversity finds what nobody thought
to ask.*

Four lenses ran: identity leakage, silent failure, do-the-tests-test-anything, what-is-missing.
Identity returned APPROVED. **"What is missing?" found the defect that mattered most** — the nudge
never stopped and repeated itself verbatim, which was the owner's explicit requirement and which no
conformance review would have caught, because the diff matched the plan.

---

## What I got wrong

- **I designed the always-on pointer before the owner gave the ladder requirement, then never
  reconciled them.** The review found it, not me. Writing a contract does not help if you do not
  re-read it after the requirements change.
- **I checked for the wrong artifact names** when verifying a stopped worker had left the tree
  clean, concluded it was, and proceeded. It had left seven files modified. Grep for what the card
  would plausibly have named things, or just read `git diff --stat`.
- **I nearly shipped on a green gate three times.** Each time the thing that stopped me was reading
  the diff, not the gate.

## What went right and is worth repeating

- **Recording a root cause as a hypothesis with its verification query attached.** The other session
  did this on #230. It cost one read-only query to disprove and prevented a wrong cause being
  written into the product doc as fact. Cheaper than being right by luck.
- **Decoupling the owner's blocking need from my in-flight work.** Onboarding needed #235, not
  #237. Merging #235 first unblocked him and meant my branch got rebased and re-gated on the
  combination — so the second merge was tested rather than assumed.
