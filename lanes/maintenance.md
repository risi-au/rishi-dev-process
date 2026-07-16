# LANE: Maintenance — dependencies, upgrades, patches, tooling
<!-- CAP: 80 lines -->

For: dependency upgrades, security patches, framework/tool version bumps, CI
maintenance, lockfile refreshes.

## Steps

1. **Scope.** What's being upgraded and why (security advisory, staleness,
   feature need). Security patches jump the queue; routine bumps batch together.
2. **Read the changelogs** for anything crossing a major version. List the
   breaking changes that touch this project before upgrading.
3. **Triage risk.** Patch/minor bumps of test-covered deps = R0/R1. Major
   versions, framework cores, auth/crypto libraries, anything with a migration
   guide = R2 (plan-lite + owner approval).
4. **Upgrade in isolation.** One PR per risky upgrade; routine minors may batch.
   Never mix upgrades with feature or bug work.
5. **Gate fully.** Upgrades are exactly where "tests pass" earns its keep. Note
   new deprecation warnings in the finish report as future maintenance.
6. **Review.** R2 upgrades get a fresh review focused on the breaking-change list.
7. **Release → retro.**

## Lane rules

- Pin resulting versions; commit lockfiles.
- New transitive dependencies with install scripts or postinstall hooks get
  flagged to the owner (supply-chain check).
- If the gate breaks and the fix isn't obvious within the time checkpoint, report
  options (pin back / fix forward / defer) instead of grinding.
