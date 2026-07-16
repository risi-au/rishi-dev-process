# LANE: New project — greenfield
<!-- CAP: 80 lines -->

The highest-risk lane for process failure: a past project took ~7 hours for a
40-minute build because of an oversized plan and repeated broad reviews. This lane
exists to prevent a repeat.

## Steps

1. **Grill.** Interrogate the owner before any design: purpose, users, in/out of
   scope, data touched, where it runs, what "done" means, what it must NEVER do.
   Assume the first description is incomplete.
2. **Product contract** (`templates/product-contract.md`) — ONE page. Owner
   approves it. This page, not the plan, is what every worker and reviewer anchors to.
3. **Triage risk.** Greenfield is not automatically heavy. A small tool with no
   auth/deploy/data triggers is R1: plan-lite, one implementer, one reviewer, done.
   R2 triggers (auth, secrets, deployment, destructive potential, cross-user
   access) get plan-full with sections SEPARATED: core product | safety invariants
   | deployment | optional hardening (deferred to V1.1 by default) — so reviewers
   verify a bounded candidate instead of re-litigating the whole safety surface.
4. **Repo setup:** git init, .gitignore, README stub, and the gate tooling
   (types/lint/tests) BEFORE product code — feedback rate is the speed limit, so
   build the gate first. Add the harness pointer (`project-setup/README.md`).
5. **Model Consult**, then **implement V1** per plan — one implementer owns all edits.
6. **Gate + receipt**, then ONE `FULL_REVIEW -> FOCUSED_FIX -> FOCUSED_REREVIEW`.
   Two-cycle stop. Do NOT restart broad reviews after each fix.
7. **Deployment**, if any, runs as `lanes/deployment.md` with its own contract —
   a separate workstream, never mixed into the code-review loop.
8. **Wrap:** docs proportionate to the project (a small tool gets a small README),
   finish report, retro.

## Lane rules

- Target for a routine R1 tool: contract + plan ≤1 hour; first candidate in
  30–60 min; ≤3 orchestrated tasks; one review cycle. At 2× target: checkpoint
  report to the owner.
- V1 ships the contract, nothing more. Hardening and nice-to-haves go to a V1.1 list.
- The plan never exceeds its template cap; supporting detail links out, not in.
