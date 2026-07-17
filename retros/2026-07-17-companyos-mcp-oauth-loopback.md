# Session Retro: 2026-07-17 companyos mcp-oauth-loopback (#90 follow-up / #93 / PR #94)
<!-- Data only — changes NOTHING until the owner approves. CAP: 1 page. -->

## Metrics
- Lane: Investigation → Bugfix | Size/risk: Standard / R2 (auth) | Elapsed: long, single session
- Workers spawned: Codex reviewer @ medium ×3 (1 FULL_REVIEW + 2 FOCUSED_REREVIEW). No Grok
  (owner waived → self-implement).
- Review cycles: 1 full + 2 focused (F1–F3 → userinfo → APPROVED). Gate runs: full vitest ×3
  (430→433→434) + many targeted/typecheck/lint.
- Infra failures: codex `rmcp AuthRequired` at review startup ×3 (the bug under test; non-fatal,
  review still completed); 2 self-corrected SSH command errors (busybox `find` quoting;
  classifier blocked one `information_schema` query); browser-history read denied by classifier
  (approach abandoned). All environment, not review cycles.
- Token hotspots: the prolonged hunt for the exact `127→localhost` rewrite line — many SSH
  round-trips + bundle greps + component-elimination probes — after the root cause was already
  confirmed behaviorally. This was the single biggest spend.

## What worked
- Evidence-first diagnosis: live black-box probing + reading the deployed bundle + the DB
  `oauth_client` rows nailed the root cause with the owner's actual clients (not inference).
- The durable-fix framing (expand loopback variants + regression test + already-present frozen
  lockfile) survives both current and future better-auth behavior — answered the owner's
  "will it survive updates?" directly.
- Cross-vendor Codex review earned its keep: 3 valid findings + 1 real edge (userinfo) that the
  gate wouldn't have caught.

## Friction
- **Under-used graphify — the designated token-saver (owner's headline concern).** I ran two
  graphify queries at the START (they located the OAuth flow and replaced a blind sweep — good),
  then reverted to grep/Glob/Read for the rest of source discovery, and never used `path` or
  `explain`. graphify-first should have been the default move for every source-location question,
  not just the opening.
- **The big token sink was OUT of graphify's scope.** The `127→localhost` rewrite lives inside
  the better-auth dependency + deployed bundle; the graph is **code-only (COS source, 347 files),
  not node_modules/build output**. So the longest, most expensive stretch (hunting the exact
  rewrite line via SSH bundle-greps) was structurally un-graphify-able — confirmed by a mid-hunt
  query returning only COS auth-adjacent nodes. Lesson: graphify saves tokens on *source*
  discovery; dependency/bundle internals need other tools, and a confirmed dependency-internal
  defect is a STOP signal, not a grep marathon.
- **Rabbit-holed the dependency hunt.** Related to the above: I pinpoint-hunted long after the
  behavior was confirmed and a proven fix existed.

## Proposals (max 3)
| # | Target file | Exact change | If target at cap: what to remove |
|---|---|---|---|
| 1 | models/graphify.md (53/120) | (Proposed, NOT adopted) "When to use": add SCOPE bullet — graph indexes project source only, not node_modules/deps/build output; dependency-internal/compiled behavior is out of scope, use direct tools; a defect confirmed to live in a dependency/bundle is a stop signal (behavioral fix + regression test, or owner-approved instrumentation), not a grep hunt. | n/a (room) |

Owner decision: RECORD ONLY — no harness edits this session. Owner rejected the earlier
session-specific proposals (hunt-budget/staging-write) as too narrow; the graphify-scope proposal
above is recorded as data, not applied. Standing learning for future sessions: default to
graphify (query/path/explain) for source discovery; recognize it is source-only and don't chase
dependency/bundle internals through it.
