# GREEN — why a worker's "tests pass" is not evidence, and what to do about it
<!-- HARD CAP: 90 lines. To add, remove (core/SELF-IMPROVE.md). -->

A worker saying "green" is a claim about a measurement it also authored. Four distinct
failure modes hide behind it. They need different cures — a single "review harder" does
not cover them.

## The four ways green lies

| # | Failure | Why tests pass | Cure |
|---|---|---|---|
| S | **Scope blindness** — green is scoped to what it ran, never the full suite | Worker command runners cap out (~30s); each card sees one package | Orchestrator's own FULL uncached gate. Non-negotiable |
| M | **The test measures nothing** — asserts the wrong thing, or is passed by deleting code | Same mind wrote the bug and the test, encoding the same misunderstanding | Falsification-first (below) + mutation receipt |
| G | **No test exists for the guarantee** — the promise was never made executable | Nothing to fail | Falsification-first; adversarial review as backstop |
| Q | **Silent failure** — a guard rejects what it should accept, producing no error | A filter that drops everything still compiles and still passes absence tests | Silent-failure sweep (below) |

## Evidence (companyos-lexi, 2026-07-25 → 07-26, two consecutive R2 batches)

Nine real defects across Shot 8 and #20. What caught them:

- **Adversarial review: 7.** Every one had passing tests and a green gate. Includes an
  imported bundle able to fabricate human confirmation, an audit event relabellable as
  human-confirmed evidence, an intake path usable as an existence oracle, and TSV
  injection able to forge a `verified` flag.
- **Orchestrator's full uncached gate: 2.** Both invisible to every worker.
- **Orchestrator reading the code: 2.** One of these — a UI filter silently discarding
  every record and event citation — **compiled cleanly and passed its package's tests.**
  No gate, no test and no worker report would ever have surfaced it.
- **Worker self-report: 1** (an honest disclosure of a dynamic-import workaround).

Read that as a budget: review and orchestrator verification find the defects. Adding
more worker-run tests finds none of them.

## Falsification-first (cures M and G)

For any card carrying a named guarantee, dispatch **two** cards, in order:

1. **Falsification card** (`templates/packet-falsification.md`) — writes ONLY tests, which
   MUST fail against current `main`. It never sees the implementation and never writes
   any. Its deliverable is red tests plus their verbatim failure output.
2. **Implementer card** — makes them green and **may not edit them**. List the test files
   as forbidden-to-modify in its packet.

The implementer can no longer pass by writing a weak test, because it cannot write the
test. This is the only mechanism here that removes the conflict of interest rather than
inspecting around it.

Skip for trivial/R0 work. Worth one extra dispatch on anything R2.

## Mutation receipt (cures M)

**Every new regression test must be shown to FAIL against the pre-change code.** Not
asserted — shown, with the verbatim failure. Cheapest reliable form: stash only the
source change (keeping the test), run the test, capture the failure, restore.

A new test that passes against the old code is measuring nothing, and this is common
enough that it must be a receipt rather than a promise. It caught a real gap on
2026-07-26: a widened sanitizer's test was confirmed to produce 6 forged rows against
the old regex versus 1 against the new — which is the only reason the fix was
trustworthy, since the orchestrator had written it.

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
- Restricting the orchestrator's *reading* to save tokens (cf. Devin Fusion's cost
  optimisation). Two defects here were found only by the orchestrator holding the whole
  diff and the gate output at once. Deny the orchestrator's **writes**; never its reads.
