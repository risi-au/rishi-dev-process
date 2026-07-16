# Plan (full): <task>
<!-- For Heavy size or R2 risk. Owner approves before code.
     Keep the sections SEPARATED so reviewers verify a bounded candidate
     instead of re-litigating the whole safety surface every pass. -->

Contract: <path/link — the one page everyone anchors to>
Branch: | Issue:

## 1. Core product (keep small)
<what V1 actually does — should read close to the contract>

## 2. Architecture
<components, data flow, key decisions with one-line rationale each>

## 3. Safety invariants + risk triggers
<the R2 triggers; how each is neutralized; what is owner-approval-gated>

## 4. File-level steps
<ordered; grouped by phase if multi-phase>

## 5. Deployment (or "none")
<host, user, permissions, secrets by NAME only, smoke test — executed via lanes/deployment.md>

## 6. Optional hardening (deferred by default)
<V1.1 candidates — explicitly NOT part of the V1 review scope>

## 7. Test plan + acceptance

## 8. Estimate + checkpoints
