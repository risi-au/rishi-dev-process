# LANE: Deployment — live environments, infra, releases
<!-- CAP: 80 lines -->

For: deploying to VPS/production, service configuration, users/permissions, DNS,
reverse proxies, systemd, CI/CD changes. Live systems are always owner-approval
territory; treat this lane as R2 unless the owner says otherwise.

## Steps

1. **Deployment contract** — one page: what goes live, target host/user/paths,
   service boundaries, secrets needed (names only), rollback plan, smoke test.
   Owner approves BEFORE any live system is touched.
2. **Environment preflight.** Verify access (one attempt — one-strike rule), tool
   versions, disk/ports, and that the deploy user/permission model matches the
   contract. Late discovery of cross-user/ACL requirements was a major cost sink
   in a past project — surface these NOW, not during rollout.
3. **Stage changes** as scripts/config in the repo (reviewable) rather than ad-hoc
   live edits, wherever feasible.
4. **Deploy step-by-step**, checking each step's output. Never chain destructive
   steps unattended.
5. **Smoke test** per the contract, on the live system. Smoke tests are never
   cached or skipped — not even for "identical" redeploys.
6. **Live-only defects** use a narrow hotfix path: smallest fix, immediate smoke
   re-test, backport to the repo. Broad changes never happen live.
7. **Record:** deployed revision, time, config changes; update the project's
   deployment/ops doc. Finish report + retro.

## Lane rules

- Separate approvals: deploy approval ≠ commit approval ≠ plan approval.
- Secrets never appear in reports, logs, or (where avoidable) shell-visible args.
- Anything destructive to live data (drops, deletes, migrations) needs
  per-instance owner approval with a stated backup/rollback.
