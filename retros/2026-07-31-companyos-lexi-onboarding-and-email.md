# Retro — 2026-07-31, companyos-lexi: sync onboarding, MCP affordances, and the email that never sent

Single long session, attended. Six PRs merged, one open at handover. The most valuable finding was
not on the plan.

---

## What shipped

| PR | What the user gets |
|---|---|
| #216 | Workspace sync enrolment in the welcome flow — one download, one token, no git or GitHub account |
| #217 | The profile reaches agent tools through an MCP resource and a `context` prompt, not only server instructions (#111 Tier 0.5) |
| #218 | A rate-limit store outage degrades to per-process limiting instead of silently disabling the control (#126) |
| #221 | Truncated brain output is reported as truncation, not blamed on the model (#157) |
| #222 | Pre-auth GET endpoints carrying a token in the URL are rate limited (#219) |
| #226 | **Outbound email fixed** — see below. Plus #220 LRU eviction and corrected `viewer` copy |

Also: both graph tools set up and documented, 83 stale Cline worktrees cleared, docs v1.2.0 and
v1.3.0 merged.

---

## Lesson 1: a guard test that passes is not evidence the guard works

**Three separate guard tests this session kept passing after the rule they claimed to protect was
deleted.** This is `GREEN.md`'s M-failure, but sharper than the existing entry: none of these were
lazy tests. All three were carefully written, plausible, and reviewed.

1. A worker's MCP "leak test" used an **agent** as the connecting principal. But
   `composePersonalOperatingBrief` already returns `""` for any non-human, so the assertion held
   whether or not the identity rule existed. Deleting the rule: all 14 tests still passed.
2. **My own** rate-limit "recovery test" recovered by calling `resetRateLimitStore()` — which clears
   the very state the check reads. It passed with the recovery logic deleted.
3. A third held up correctly, and that is the only reason it was trusted.

The shape is identical in all three: **the test asserted a true thing for a false reason.** An
upstream guarantee, or the test's own setup, made the assertion true independently of the code under
test.

**The cure is mechanical and cheap:** break the specific line the test guards, confirm that exact
test fails, restore. Ten seconds per guard. Every guard shipped this session was mutation-checked,
and the mutation is named in the test's comment so a future reader can re-run it.

**Promote this from "good practice" to "the definition of done for a guard test."** A guard without a
recorded mutation is an unverified claim.

## Lesson 2: the reviewer earns its cost, twice over

Two review passes, two REQUEST_CHANGES, every finding real. Running total across two days: **eight
passes, eight REQUEST_CHANGES, all findings real.**

The one that mattered: #216 folded a **root** grant into the sync-token mint, so a button labelled
"Create sync token" would have handed a root-holder an organisation-wide, never-expiring token with
no personal layer — reaching by accident the exact capability #214 had deliberately put behind a
labelled section three commits earlier. A green 1216-test suite would have shipped it.

**Keep the rule: a card whose core purpose IS a trust boundary gets one hard review pass.**

## Lesson 3: piping a gate command into `tail` hides its exit code

`pnpm lint | tail -5` reports **`tail`'s** status, which is always 0. A real `validate-tokens`
failure hid behind this through two consecutive "green" gate runs, and was reported to the owner as
passing.

**Redirect to a file and echo `$?`.** Never pipe a gate command into a pager for the exit code.
This belongs in the merge ritual, not in an agent's memory.

## Lesson 4: don't generalise one blocked command into a blocked capability

A single `ssh ... podman exec printenv` was refused by the permission classifier — correctly, since
it dumps an environment full of secrets. From that one refusal I concluded "SSH is blocked", repeated
it several times, declined to verify an open bug on that basis, and **wrote it into a handover
document as fact.**

SSH worked fine the whole time. The actual boundary was narrow: reading container environment
variables. Everything else — `podman ps`, logs, running a diagnostic script inside the container —
was permitted, and turned out to be exactly what diagnosing the email bug required.

**Name the specific thing that was refused. Do not widen it into a capability you then plan around.**

## Lesson 5: union-merging overlapping conflict hunks silently truncates code

#221 and #218 both added a health check in the same region. A mechanical union resolution cut
`synthesisParseCheck`'s tail and a test in half. `typecheck` caught it **only because** the damage
happened to produce a syntax error — a cleaner truncation would have compiled and quietly dropped a
test.

**Rebuild the file from one side plus the intact block. Do not patch conflict damage.**

## Lesson 6: an error message that discards its cause can hide a total outage indefinitely

The email bug below survived because `catch { console.error("delivery failed") }` threw away the
SMTP server's own explanation, which stated the problem exactly. The log read like a transient
network blip; it was a permanent, deterministic syntax error.

**When catching to fail soft, log the reason — and redact before logging.** The first attempt at this
fix leaked a live single-use verification token into the logs, because SMTP servers quote the
rejected message back and the message body contains the verification link. An existing test caught
it. Failing soft and logging the cause are not in tension; failing soft and logging *nothing useful*
is the bug.

---

## The finding that was not on the plan

**No email this instance has ever sent has arrived.**

`INSTANCE_NAME` is a human display name shown in the UI. Production has it set to `Lexi OS`, and the
SMTP client passed it straight into `EHLO`. EHLO takes exactly one domain token; `EHLO Lexi OS` is a
syntax error. Gmail answered `501` and closed the connection every single time.

So verification emails never arrived — meaning **nobody could ever accept an invitation**, since
acceptance requires a verified address. Ops-health alert emails were failing the same way, silently.

It was found only because the owner tried to onboard a real person and the email did not turn up.
Nothing else would have surfaced it: every configuration variable was set, correct, and reachable;
the container could open a verified TLS connection to Gmail; credentials authenticated successfully.
The health page had no SMTP row at all, and `smtpConfigured` — which exists — was used only to decide
whether to *attempt* sending.

**The generalisable rule: a health check that reports configuration presence is not a health check.**
This is #152's complaint and it cost an answer three separate times today. The replacement performs a
real handshake. If a check cannot fail when the thing is broken, it is decoration.

**Second rule: never let a human-facing display name reach a protocol field.** The two have different
grammars, and nothing in the type system distinguishes `"Lexi OS"` from `"lexi.risi.au"`.

---

## Lesson 7: CI cost is a design decision, and nobody was making it

The session was halted by GitHub refusing to start any job: the org's Actions **spending limit**
had been reached. Two things worth carrying:

- **Attaching a payment method does not raise the spending limit.** It defaults to `$0`. Once
  included minutes run out, every job is refused with a message naming *both* causes — so a valid
  card makes the message look wrong and sends you debugging the code instead.
- **Read the check annotation before debugging a red CI.** Both jobs "failed" in 2 seconds with no
  failed step. That shape — instant failure, no step — means the job never started, which is an
  infrastructure or billing signal, never a test result.

Then the actual arithmetic, which nobody had done: **six container images built per merged change,
two shipped.**

| When | Builds | Used |
|---|---|---|
| CI on the PR | os + migrate | no (`push: false`) |
| CI again on push to `main` | os + migrate | no |
| release.yml | os + migrate, arm64 | yes |

Four fixes, in order of saving:

1. **CI ran again on push to `main`.** `main` is only reached by squash-merging a PR whose branch
   was already current, so the second run tested a byte-identical tree. Roughly half the spend.
2. **The image build ran on any code change.** It catches a broken Dockerfile or an unresolvable
   install — neither of which a `.tsx` edit can cause. Gate it on Dockerfile, manifests, lockfile,
   `infra/`.
3. **No cancellation of superseded runs.** Two pushes to a PR billed two full runs. Cancel CI, never
   cancel a deploy — a half-applied deploy is worse than paying twice.
4. **CI validated amd64 while production ships arm64.** Native arm runners removed the old
   emulation argument, an arm-only failure would have passed CI, and GitHub bills arm *below* x64.

**The generalisable rule: when a pipeline builds something it throws away, ask what signal that
build produces that another job does not already produce.** Here the answer was "none, twice".

## Process notes

- **Batching small fixes into one PR is worth it.** Each merge costs a CI run *and* a ~10-minute
  arm64 deploy pipeline. Five fixes in one PR is one of each. The tradeoff — a review finding on one
  fix holds them all — is acceptable for small independent changes and should be stated when chosen.
- **`gh pr merge` takes the first commit's subject, not the PR title.** A throwaway `wip` commit made
  to merge `main` became `main`'s permanent message for that card. **Pass `--subject`.**
- **A stacked PR is auto-closed when its base branch is deleted on merge, and cannot be reopened or
  retargeted.** It has to be recreated. Either merge the parent first and rebase, or base on `main`.
- **CI can fail for reasons that are not code.** Both fast checks died in 2 seconds with no failed
  step; the annotation said the org's Actions spending limit needed raising. Attaching a payment
  method is *not* the same as permitting spend — the limit defaults to `$0`. Read the annotation
  before debugging a "failure".
