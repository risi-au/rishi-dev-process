# LANE: Ship — the fast gear, DEFAULT for Lexi work
<!-- CAP: 80 lines -->

**Approved 2026-07-26 because the owner is currently Lexi's only user.** Revisit this
lane the day a real user other than the owner depends on Lexi daily — the trade this
lane makes (no per-card adversarial review) is only safe while one person feels every
regression immediately and can say so.

Ship gear is the DEFAULT lane for Lexi (`companyos-lexi`) work. Use Guarantee gear
(the full process: `lanes/feature.md` etc., falsification-first, per-batch adversarial
review) instead whenever a card touches any of:

- Auth or permissions
- Data deletion or retention
- Citation or trust semantics (anything a cited answer's honesty depends on)
- MCP public contract changes (`docs/CONSTITUTION.md`'s MCP-is-public-API rule)
- A DB migration
- Anything else carrying a named guarantee (a D-numbered decision, a documented
  invariant)

**When in doubt, Guarantee gear.** Ship gear is for the ordinary case: a bounded,
well-understood change with no load-bearing guarantee at stake.

## Steps

1. **Intake + triage** as normal (`core/PROCESS.md`). If any Guarantee-gear trigger
   above applies, stop — this is the wrong lane.
2. **One implementer card.** No separate falsification card. The packet still carries
   every standing trap (encoding, underscore-ignore, the repo's known packet rules in
   `V2 - LEXI/CARRIED-CONTEXT.md` §7) — Ship gear removes a review cycle, not the
   packet discipline that keeps a single pass from going red.
3. **Orchestrator reads the full diff.** Not a summary — the actual diff, checked
   against the packet's contract (missing-but-not-broken doesn't show up in a green
   gate; see `core/GREEN.md`).
4. **Orchestrator runs the full uncached gate once** (lint + typecheck --force +
   the full test suite from repo root). Record the receipt.
5. **Cheap-tier silent-failure sweep.** A cheap-combo dispatch whose only job is:
   does this diff silently drop a field, skip a filter, or relax an existing test's
   expectation to `expect.any(...)`. Not a full adversarial review — narrower and
   cheaper, catching the specific failure mode a green gate hides.
6. **Merge under the standing four-condition floor**, with the review condition
   satisfied differently: full gate green (uncached) — yes; CI green on the PR — yes;
   diff entirely in scope — yes; **independent adversarial review — deferred to the
   shot-end backstop below, not required per card.**
7. **No mid-flight scope changes.** A new owner decision that lands while a card is
   running goes to the NEXT card. Never re-scope a card already in flight — that's
   what turned "one implementer" into a moving target before.

## Shot-end backstop (mandatory, not optional)

Before any shot worked in Ship gear is declared done: **one batched adversarial
review** covering everything that shot merged under Ship gear — same adversarial
review this repo already runs for Guarantee gear, just run once against the whole
shot's diff instead of once per card. This is what makes skipping the per-card review
safe: nothing merges into a "done" shot without an adversarial pass having looked at
it, only later and batched instead of gating each merge.

If the backstop finds a blocking defect, it goes through Guarantee gear to fix,
regardless of what gear shipped the original card.

## Lane rules

- Ship gear does not change the merge authority or the deploy consequence — merging
  still deploys to production (`CARRIED-CONTEXT.md` §2). It changes review timing,
  not review existence.
- If a card's actual diff turns out to touch a Guarantee-gear trigger the intake triage
  missed, stop, do not merge, and re-run it through Guarantee gear.
