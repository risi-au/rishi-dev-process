# Gate Receipt
<!-- Evidence that a specific release candidate passed the gate.
     Valid only while every fingerprint below is unchanged. -->

Candidate: <branch> @ <commit sha>
Base: <default branch> @ <sha>
Diff fingerprint: <output of `git diff --stat` + files count>
Environment: <runtime + key tool versions; lockfile hash if deps exist>

| Check | Command | Result | Duration |
|---|---|---|---|
| typecheck | | ok/fail | |
| lint | | ok/fail | |
| tests | | N passed / M failed | |

Run by: <agent/model> at <timestamp>

Invalidation: void if the candidate sha, dependencies, lockfile, or relevant
config change. Deployment smoke tests are NEVER covered by this receipt.
