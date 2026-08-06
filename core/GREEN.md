# GREEN — why a worker's "tests pass" is not evidence, and what to do about it
<!-- HARD CAP: 90 lines. To add, remove (core/SELF-IMPROVE.md). -->

A worker saying "green" is a claim about a measurement it also authored. Five distinct
failure modes hide behind it, needing different cures — "review harder" covers none.

## The five ways green lies

| # | Failure | Why tests pass | Cure |
|---|---|---|---|
| S | **Scope blindness** — green is scoped to what it ran, never the full suite | Worker command runners cap out (~30s); each card sees one package | Orchestrator's own FULL uncached gate — **and prove that gate measured something** (below) |
| R | **Removal invisibility** — green because the check that would have failed was deleted | Reports describe additions; a deletion leaves no trace in one | Read the diff against what it replaced; require a `Removed or weakened:` field in every packet |
| M | **The test measures nothing** — asserts the wrong thing, or is passed by deleting code | Same mind wrote the bug and the test, encoding the same misunderstanding | Falsification-first (below) + mutation receipt |
| G | **No test exists for the guarantee** — the promise was never made executable | Nothing to fail | Falsification-first; adversarial review as backstop |
| Q | **Silent failure** — a guard rejects what it should accept, producing no error | A filter that drops everything still compiles and still passes absence tests | Silent-failure sweep (below) |

## Evidence (companyos-lexi, 2026-07-25 → 07-26, two consecutive R2 batches)

Nine real defects. **Adversarial review: 7** (all with passing tests and a green gate —
a bundle able to fabricate human confirmation, an intake path usable as an existence
oracle, TSV injection forging a `verified` flag). **Orchestrator's full gate: 2.**
**Orchestrator reading the code: 2** — one, a UI filter discarding every citation,
compiled cleanly and passed its package's tests. **Worker self-report: 1.**

Read that as a budget: review and orchestrator verification find the defects. Adding
more worker-run tests finds none of them.

**2026-08-06 repeat, nine again, same shape.** Review found 6 in one run (fresh session,
clean checkout, told which two doubts to attack); the merged-tree gate 1; reading the
diff the R-class one. **A second reviewer given that same diff returned 3 findings, all
false, and missed it** — reviewer quality is the variable, not reviewer presence.

## Prove the gate measured something (the hole in S's cure)

The orchestrator's full gate is evidence only if it ran the tests. `turbo run test
--force` does not forward `DATABASE_URL` under turbo 2 strict env mode: every
database-gated test skipped while turbo printed `30 successful, 30 total`, including the
two race tests that batch existed to add.

**Check the count, not the colour** — compare tests-run against what you expected and
grep the log for skip reasons. Run any env-dependent gate in both states once, so you
recognise its silent-skip output.

## Falsification-first (cures M and G)

For any card carrying a named guarantee, dispatch **two** cards, in order:

1. **Falsification card** (`templates/packet-falsification.md`) — writes ONLY tests, which
   MUST fail against current `main`. It never sees the implementation and never writes
   any. Its deliverable is red tests plus their verbatim failure output.
2. **Implementer card** — makes them green and **may not edit them**. List the test files
   as forbidden-to-modify in its packet.

The implementer can no longer pass by writing a weak test, because it cannot write the
test — the only mechanism here that removes the conflict of interest rather than
inspecting around it. Skip for trivial/R0; worth one extra dispatch on anything R2.

## Mutation receipt (cures M)

**Every new regression test must be shown to FAIL against the pre-change code.** Not
asserted — shown, with the verbatim failure. Cheapest form: stash only the source change
(keeping the test), run it, capture the failure, restore.

A new test that passes against the old code is measuring nothing, and this is common
enough to require a receipt rather than a promise. 2026-07-26: a widened sanitizer's test
produced 6 forged rows against the old regex versus 1 against the new — the only reason
that fix was trustworthy, since the orchestrator had written it.

## Silent-failure sweep (cures Q)

The cheapest high-yield question in the process, because it is *specific* and answerable,
unlike "review this diff". Dispatch to a cheap tier after implementation:

> List every filter, guard, early return, `continue`, `?.`, default value and catch block
> this diff adds or changes. For each: what happens when it rejects something that should
> have passed? Does anything observe the rejection? Name every one whose failure mode is
> silence rather than an error.

Then: **every filter needs a positive-partner test** proving the thing still appears when
it should (`core/CONDUCT.md`).

## What does NOT work

- More worker-run package tests — no effect on S, M, G or Q.
- A bigger implementer model — these defects are spec-comprehension and boundary
  problems, not capability problems.
- Trusting an honest worker. cline reports honestly and it does not help: honesty is not
  the failure mode, **scope** is. "Green for what I ran" is true and still misleading.
- Restricting the orchestrator's *reading* to save tokens. Defects here were found only by
  it holding the whole diff and the gate output at once. Deny its **writes**, never reads.
