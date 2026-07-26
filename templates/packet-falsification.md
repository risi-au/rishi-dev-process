# PACKET — falsification card (tests only, must fail)

<!-- Dispatched BEFORE the implementer. See core/GREEN.md. Keep under 40 lines when filled. -->

Repo: `<abs path>` · Branch: `<branch>` (already checked out — stay on it)

## Your job: write tests that FAIL. You will not implement anything.

Another worker implements afterwards and **will not be allowed to edit your tests**. So
your tests are the definition of correct behaviour. If you write a weak test, the defect
ships.

## The guarantee(s) to falsify

> `<state each guarantee in plain language — what must be true, and what must never be
> true. One or two sentences each. No implementation detail.>`

## Required deliverable

For each guarantee, tests that:

1. **Fail right now**, against the current code, for the RIGHT reason — because the
   behaviour is absent, not because of a typo, a missing import or a bad fixture.
2. **Come in pairs.** Every "X must not happen" test needs a positive partner proving X
   still happens when it should. An absence-only test is passed by deleting everything.
3. **Attack the guarantee, not the plan.** Ask how a caller, an attacker or a concurrent
   writer breaks the promise — not whether the code matches a spec.
4. Cover the boundary: absent/null inputs, a value that is only separators or whitespace,
   duplicates, a legacy record predating the change, and the "unknown/unclassified" case
   (does an unrecognised input default to permitted, or to refused?).

## Return the verbatim failure output

Run them and paste the actual failure text. **A test that passes right now is a bug in
your work** — say so explicitly if you cannot make one fail, and explain why; do not
delete it and do not weaken it into passing.

## HARD CONSTRAINTS

- **Write ONLY test files.** Touching any source file is a failure of this card.
- **Write in small incremental edits, never more than ~120 lines per edit.**
- Do not modify existing tests. Do not create a migration.
- Do not commit, stage, branch, merge, or push. Read-only `git diff`/`git status` is fine.
- Your command runner may cap at ~30s: run only your new tests, and say what you could
  not run rather than claiming anything about the full suite.
- Secrets: never read, copy, or print `.env` values.

## Allowed files

- `<exact test file paths — nothing else>`

## Return format (≤30 lines, no code blocks)

```
Test files written:
- path — which guarantee it attacks
Tests added: <name — one line on the failure mode it catches>
VERBATIM failure output: <paste it>
Any test that did NOT fail: <none | which, and why — this is a defect in this card>
Boundary cases covered: <list>
Guarantee I could not express as a test: <none | which, and what would be needed>
```
