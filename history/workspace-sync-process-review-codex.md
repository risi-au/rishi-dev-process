# Workspace Sync development-process retrospective

Status: complete

Date: 2026-07-16

## Executive summary

Workspace Sync V1 was delivered and is operational, but the delivery process was
too large for the product. The visible delivery span was about 6 hours 57 minutes
from the first recorded plan-review attempt to the final documentation commit.
The initial implementation itself took only 40 minutes 8 seconds.

The evidence does not show Orca transport as the latency bottleneck. Orca task
creation, dispatch, status, and terminal operations completed in fractions of a
second. The best-supported explanation for the high cost is the model work around
Orca: repeated fresh sessions loading a 27,865-byte plan, more than 33 KB of
supporting documentation, the source tree, tests, and prior findings; repeated
broad reviews; correction cycles; repeated test gates; deployment hardening; and
live login debugging.

The project also had real risk that justified more care than an ordinary folder
utility. It operates on real Git working copies, must never overwrite local work,
uses private GitHub access, runs as an isolated service user, and writes into a
different user's Syncthing-managed project tree. Those constraints justified a
fresh final reviewer, strong Git invariants, authentication review, and a full
release gate. They did not justify restarting a whole-repository review after
every narrow fix.

Because Rishi wants real Orca tracking and multiple agents, the recommended
default for future small applications remains TRIP runtime tier T2, but with a
lean routine-risk lane: one coordinator, one implementer, one fresh independent
reviewer, a complete test gate on every reviewed release candidate, and focused
re-review only for recorded findings. Elevated-risk work adds named specialists
and risk checks while using the same compact context packets, review-delta rules,
and two-cycle stop limit.

No exact token telemetry was available. All token percentages in this report are
diagnostic estimates, not billing data.

## What the project actually built

The final application stayed close to the corrected product intent:

- a standalone service, independent of CompanyOS;
- an isolated workspace-sync VPS user rather than the hermes user;
- polling of private repositories in Brissie-Digital-PTY-LTD;
- cloning new repositories into the existing /home/hermes/projects tree;
- periodic fetch and safe fast-forward-only pull for clean repositories;
- no automatic reset, delete, merge, commit, or push;
- status and error reporting in a password-protected dashboard;
- manual rescan, clone, and safe-pull actions;
- Syncthing, not Workspace Sync, distributes working files between machines.

The application does not upload local working files to GitHub. Extra, modified,
or untracked files remain local Git working-tree state and block automatic pull
where required. A repository deleted from GitHub is reported as missing or in
error; its local folder is not automatically deleted.

An early brief said the process would run as hermes. Rishi corrected this and
required a completely separate user. That correction was important and valid,
but it introduced a cross-user filesystem and ACL boundary that later required
deployment hardening.

## Evidence and limitations

This report uses:

- the approved V1 plan;
- repository commits and timestamps;
- the Orca orchestration task list and recorded outcomes;
- the current architecture, deployment, operations, security, and development
  documentation;
- three read-only Codex retrospective analyses covering timeline, usage causes,
  and process redesign.

Orca stores task timestamps in UTC. The timeline below is converted to Brisbane
time (AEST, UTC+10) to match the Git commit timestamps.

The evidence does not include exact per-session token accounting. It also does
not prove that all time between recorded tasks was active work. Wall-clock spans
therefore include coordinator activity, tests, user approval waits, deployment,
debugging, and idle gaps. Task-runtime totals are more precise but omit work that
was not itself an Orca worker task.

## Timeline

| AEST time | Stage | Observed result |
| --- | --- | --- |
| 13:36-13:37 | Codex plan review | Blocked by the Windows sandbox before producing a review. |
| 13:38-13:47 | Grok plan review | Requested plan changes. |
| 13:49-13:57 | Grok plan follow-up | Approved the revised plan. |
| 14:22-15:03 | Grok implementation | Initial V1 implementation completed in 40m 08s. |
| 15:07-15:27 | Coordinator correction | Git-safety corrections and verification. |
| 15:28-15:39 | Grok final review | Requested a P1 correction. |
| 15:42-15:44 | Codex review | Blocked by the Windows sandbox. |
| 15:44-16:08 | Codex retry | Broad review returned more requested changes. |
| 16:09-16:20 | Codex follow-up | Requested further changes. |
| 16:34-16:59 | Fresh Codex review | Broad review found additional adjacent issues. |
| 17:17-17:19 | Fresh Codex attempt | Blocked by an authentication-token problem. |
| 17:24-17:38 | Claude attempt | Cancelled after a silent run and repository changes. Rishi later explicitly selected Codex for future work. |
| 17:41-17:50 | Codex audit | Requested five corrections. |
| 18:02-18:05 | Focused Codex review | Found one shutdown-race issue. |
| 18:07-18:08 | Focused Codex re-review | Approved. |
| 18:10 | Feature commit | V1 feature commit recorded. |
| 18:57-19:03 | Deployment-fix review | First attempt blocked by the Windows sandbox. |
| 19:03-19:10 | Deployment review retry | Linux path and deployment review completed. |
| 19:13-19:19 | ACL follow-up review | Cross-user ACL correction reviewed. |
| 19:23 | Deployment-hardening commit | Linux deployment paths and ACLs recorded. |
| 19:55-20:02 | Login/docs review | Returned two P2 documentation findings. |
| 20:03-20:09 | Login/docs re-review | Approved. |
| 20:34 | Final commit | Privacy-browser login fix and operational documentation recorded. |

The observable delivery span was 6h 57m 08s, from 13:36:57 to the 20:34:05
final commit. This is elapsed delivery time, not seven hours of continuous model
execution.

## Recorded task-runtime analysis

| Work category | Task count | Recorded runtime |
| --- | ---: | ---: |
| Plan review attempts | 3 | 17m 54s |
| Initial implementation | 1 | 40m 08s |
| Coordinator safety-correction task | 1 | 20m 10s |
| Core implementation review cycle | 10 | 1h 43m 28s |
| Deployment-fix review cycle | 3 | 18m 49s |
| Login/documentation review cycle | 2 | 12m 17s |
| Total recorded Workspace Sync task runtime | 20 | 3h 32m 46s |

Eighteen of the twenty tasks were plan or review attempts. Only one was the main
implementation task and one was the coordinator correction task. The core review
cycle alone took about 2.6 times as long as the initial implementation.

Across all phases, there were seven explicit REQUEST_CHANGES outcomes, three
Windows sandbox blocks, one Codex authentication block, one cancelled Claude
attempt, and three explicit approvals. Two of the sandbox blocks occurred during
plan/core review and one during deployment-fix review. Deployment and
documentation records also include narrow follow-ups and the final documentation
approval. Some deployment outcomes are preserved only as summarized task
results, so this report does not invent a more granular classification for them.

## Why it took so long

### 1. The plan was much larger than the product brief

The approved plan was 27,865 bytes and contained twelve major sections, a full
state machine, security and reliability design, deployment design, orchestration
ownership, risk decisions, and seven delivery phases. That is a strong artifact
for a high-risk platform, but it was disproportionate to a single lightweight
service and dashboard.

Some expansion was justified by the destructive potential of unsafe Git
automation. However, the plan did not clearly separate:

- the small product contract;
- reusable Git-safety invariants;
- deployment-specific service-user and ACL work;
- optional hardening that could follow V1.

As a result, each fresh reviewer was invited to reconsider the entire safety
surface rather than verify a bounded release candidate.

### 2. Whole-repository reviews were repeatedly restarted

Fresh review is valuable once the testing gate passes. Here, multiple sessions
were repeatedly asked to read the full plan, source, tests, previous findings,
and current diff. Each broad review could discover a new adjacent edge case even
after earlier findings were fixed. The process therefore behaved like an open-
ended adversarial audit instead of a release gate.

The final closure pattern was much better: a focused review found one shutdown
race, the fix was applied, and a focused re-review approved it in 1m 25s.

### 3. Review findings caused real rework

The reviewers found legitimate issues in Git safety, shutdown behavior, Linux
deployment paths, ACLs, browser-origin handling, and documentation. Fixing these
was valuable. The inefficiency was that the findings arrived in many small
batches from broad sessions rather than one consolidated review followed by a
bounded correction pass.

### 4. Tool failures consumed elapsed time without product progress

Three project review attempts were blocked by the Windows Codex sandbox, another
was blocked by Codex authentication, and one Claude attempt was cancelled. The
same Windows CreateProcessAsUserW access failure reappeared during this
retrospective when read-only Codex analysts tried to inspect local files.

These failures were not Orca failures. Orca dispatched and tracked the work; the
worker environment could not execute the requested local commands. Repeating the
same mechanism after the failure added latency and context consumption without
improving the product.

### 5. Deployment revealed requirements that local tests could not prove

The isolated workspace-sync user had to operate safely in the hermes-owned
project tree without gaining access to unrelated homes. Linux path ownership,
directory promotion, ACL inheritance, service configuration, and credential
access required production-specific work. This was legitimate deployment effort,
but it should have been a named deployment workstream with an explicit smoke
test, not mixed into the core code-review loop.

### 6. The browser login problem needed several production iterations

The dashboard worked except in privacy-browser conditions that produced an
opaque or missing Origin. Several restarts and fixes were needed before login
worked. This is normal live-debugging risk, but it extended the project after the
core service had already passed review.

### 7. User intent had to be re-centered during implementation

Rishi correctly challenged whether the project was becoming a file-sync system.
The intended boundary was simple: GitHub repository discovery and safe Git
updates into a folder that Syncthing already distributes. A one-page product
contract at the top of the plan would have kept this boundary visible to every
worker and reviewer.

## Why token usage was high

Exact token telemetry is unavailable, so the following is a reasoned allocation
of model usage rather than an invoice reconstruction.

| Estimated model-token share | Driver |
| ---: | --- |
| About 35% | Repeated loading of the plan, supporting docs, code, tests, Orca history, and prior findings into fresh sessions. |
| About 30% | Implementation, correction reasoning, and generated changes. |
| About 18% | Review reasoning beyond the repeated context itself. |
| About 10% | Deployment, ACL, login, and documentation debugging. |
| About 5% | Test output and tool transcripts, including repeated full-gate evidence. |
| About 1-2% | Blocked, cancelled, or failed worker sessions. |
| Less than 1% | Orca task/dispatch/status JSON and coordination RPCs. |

The estimates are intentionally approximate and total to roughly 100%. They are
meant to identify where to optimize, not claim accounting precision.

The key distinction is:

- Orca provided coordination state and terminal control cheaply.
- The LLM sessions attached to Orca consumed the context and reasoning tokens.
- More tasks meant more fresh sessions, and more fresh sessions meant repeated
  ingestion of the same large materials.

Therefore, removing Orca would not solve the main problem. Using Orca with fewer,
better-scoped tasks and compact context packets would.

## What went well

- The final application remained independent of CompanyOS.
- No automatic destructive Git behavior was accepted.
- Rishi's separate-user correction was honored.
- The plan was reviewed and explicitly approved before production code.
- The implementer was not used as the final reviewer.
- Real defects were found before and after deployment.
- Commit, push, and deployment approvals remained separate.
- The project ended with architecture, deployment, operations, development, and
  security documentation.
- The production service and dashboard were verified operational.
- Orca retained a usable history of attempts, outcomes, and failures for this
  retrospective.

## What should change

### Keep TRIP runtime tiers and add a separate risk profile

TRIP's T0/T1/T2 labels describe runtime and coordination capability, not product
risk. They must retain their governing meanings:

| Runtime tier | Meaning |
| --- | --- |
| T0 | Single-agent, process-only execution with the same plan, test, review, and release phases. |
| T1 | Multi-agent execution without the Orca runtime. |
| T2 | Multi-agent execution with real Orca task, dispatch, terminal, and lifecycle tracking. |

Rishi's requested default is T2 because future work should continue using Orca
and multiple agents. Within that runtime tier, classify the work separately:

| Risk profile | When to use it | Default T2 agent structure |
| --- | --- | --- |
| R0 - low | Very small, established-pattern change with no auth, deployment, data, concurrency, or destructive-risk trigger. | Coordinator/implementer plus one fresh reviewer; minimal task graph. |
| R1 - routine | A small reversible application or feature, usually up to about 10 files or 500 changed lines, with clear architecture. | Coordinator, one implementer, one fresh reviewer; at most three Orca tasks and two concurrent workers. |
| R2 - elevated | New production service with auth, secrets, deployment, cross-user permissions, destructive operations, migrations, concurrency, or novel architecture. | Coordinator, plan reviewer, implementer, optional named specialist, complete gate, fresh final reviewer. |

Workspace Sync was an R2 project because it combined private GitHub auth,
production deployment, real Git working copies, and cross-user filesystem access.
The mistake was not treating it as elevated risk; it was running elevated-risk
work without bounded review-delta and context-reuse rules.

### Adopt a lean common pipeline

1. Classify the tier and list the exact risk triggers.
2. Write a compact product contract: purpose, in-scope behavior, exclusions,
   safety invariants, acceptance checks, and deployment boundary.
3. Obtain Rishi's plan approval before production code.
4. Give one implementer ownership of all edits. Add specialists only for named,
   non-overlapping questions.
5. Run affected tests during implementation.
6. Run the complete configured gate on the release candidate and record a gate
   receipt.
7. Give one fresh reviewer a compact review packet and the full release diff.
8. Batch all blocking findings into one correction pass.
9. After source fixes, rerun the complete configured gate on the corrected
   release candidate, then re-review only the findings, fix diff, dependency
   halo, and updated test receipt.
10. Obtain separate approval for commit, push, and deployment.
11. Run one deployment smoke test and use a narrow hotfix path for live-only
    defects.

### Replace repeated full context with role-specific packets

The implementer packet should contain the approved product contract, relevant
architecture, acceptance criteria, and editable file scope.

The final reviewer packet should contain:

- a short approved-plan summary;
- the complete release diff and diff statistics;
- risk triggers and changed interfaces;
- the test-gate receipt;
- unresolved decisions and known limitations.

Full supporting documents remain available on demand, but they should not be
copied into every prompt by default.

### Make follow-up reviews focused

The first final review remains a complete independent review. Findings must be
batched and assigned stable IDs with BLOCKING, NON_BLOCKING, or QUESTION status.

After fixes, the reviewer checks only:

- the original finding IDs;
- files changed by the fixes;
- direct dependency, security, and regression impact around those files;
- updated test evidence.

A new full review is required only if the fixes change architecture, public
interfaces, authentication, deployment behavior, dependencies, or more than
about 25% of the previously reviewed diff.

After two REQUEST_CHANGES cycles, the coordinator stops and either re-plans or
asks Rishi for a scope/risk decision. Review must not become an unlimited search
for new non-blocking improvements.

### Reuse test evidence safely

Create a machine-readable gate receipt containing:

- base revision and release-diff hash;
- dependency, lockfile, and relevant configuration hashes;
- runtime and tool versions;
- commands, results, durations, and mapped risk areas.

Always rerun cheap lint, type, and affected unit tests while making source
changes. Before resumed review, rerun the complete configured testing gate on the
corrected release candidate. Individual expensive results may be reused inside
that gate only when the configured gate explicitly supports equivalent cached
evidence and the receipt proves that no mapped component, configuration,
dependency, fixture, or environment input changed. Ownership moving to another
agent is not itself a reason to rerun an unchanged gate.

Documentation-only changes do not trigger a full source gate unless they alter
commands, deployment behavior, configuration, or operational instructions that
must be verified. Deployment smoke tests are never cached across deployments.

### Fail fast on infrastructure problems

Classify failures as product, test, environment, worker/session, or access
failures. Retry the same failed mechanism once. Then use an approved fallback or
stop for direction.

Allow one replacement worker for authentication, cancellation, or startup
failure. Repeated replacements should not consume more context. Keep the existing
one-strike SSH rule: after one failed connection, stop rather than repeatedly
guessing credentials or host state.

Record sandbox and authentication failures as infrastructure events, not code-
review failures. Preflight the chosen model login, local command execution, repo
access, and required CLI once before dispatching expensive work.

Claude should not be selected for Rishi's development work unless Rishi
explicitly changes the stated Codex preference.

### Add time and scope checkpoints

- R0 checkpoint: 30 minutes.
- R1 checkpoint: 2 hours.
- R2 checkpoint: the estimate recorded in the approved plan.

Crossing a checkpoint requires a concise status report explaining what expanded,
what remains, and the revised approach. It does not automatically cancel the
project. If work crosses its risk profile because of a new risk or architecture
change, promote the profile and re-approve the plan rather than silently
expanding the process.

## Concrete TRIP-on-Orca changes

1. Preserve runtimeTier for the constitutional T0/T1/T2 meaning and add
   riskProfile, riskTriggers, contextBudget, preferredModel, and timeCheckpoint
   to docs/TRIP-ORCA.json.
2. Add a plan-lite template capped at roughly two pages for R0 and concise R1
   work.
3. Require a one-page product contract at the top of every plan.
4. Add a gate-receipt artifact with revision fingerprints, commands, results,
   durations, and invalidation rules.
5. Add a review-delta artifact containing finding IDs, fix diff, affected files,
   dependency halo, and rerun tests.
6. Define the review state machine as FULL_REVIEW -> FOCUSED_FIX ->
   FOCUSED_REREVIEW, with explicit full-review invalidation triggers.
7. Default to two correction cycles before a user decision.
8. Treat sandbox, authentication, cancellation, quota, and access failures as
   infrastructure metrics.
9. Require phase metrics: elapsed time, task attempts, review cycles, test-gate
   count, test minutes reused, failure categories, and context-packet size.
10. Preserve separate states for test_gate_passed, review_approved, and
    release_candidate_unchanged. Approval applies only to the recorded diff.
11. Store concise Orca task results so future workers can use summaries instead
    of rereading entire terminal transcripts.
12. Keep commit, push, deployment, exceptional access, and destructive actions
    behind explicit Rishi approval.

## Target for the next comparable project

For a genuinely routine R1 tool running under Rishi's preferred T2 Orca runtime,
use this target rather than a guarantee:

- first implementation candidate within 30-60 minutes;
- one implementer session;
- one full release-candidate test gate;
- one fresh final-review session;
- at most one focused correction and re-review by default;
- no more than three main Orca tasks;
- 60-120 minutes from approved plan to a release decision when deployment does
  not reveal new environmental requirements.

For R2 work like Workspace Sync, do not promise the R1 time target. Instead,
preserve the extra safety work while applying compact context, consolidated
findings, focused re-review, complete corrected-candidate gates, test receipts,
environment preflight, and the two-cycle stop rule.

## Final assessment

The orchestration model is worth keeping. It provided traceability, independent
review, task ownership, and clear approval boundaries. The best-supported cost
drivers around the orchestration were an oversized plan, too many fresh broad
reviews, repeated full-context loading, late environmental discovery, and failed
worker mechanisms.

The corrective action is not fewer safeguards. It is fewer full-context passes,
clearer risk tiers, one consolidated final review, focused follow-ups, reusable
test evidence, and early environment preflight. This retains the benefits of
multiple agents while making the process proportionate to the work.
