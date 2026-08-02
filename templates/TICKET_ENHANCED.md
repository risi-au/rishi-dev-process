---
id: NNN
title: [one-line plain-English title]
status: new
severity: [blocking | wrong | annoying]
reported: YYYY-MM-DD
github-issue:
risk-profile: [R0 | R1 | R2]
---

## What happened (Rishi's words, lightly cleaned)

[Concise description of what Rishi did and what Lexi did wrong. No jargon. Real-world scenario.]

## Expected

[What should have happened instead. One or two sentences.]

## Initial context (code-graph results)

**Affected component(s):**
- File: `path/to/file.ts:functionName()`
- Related: `path/to/related.ts`
- [any additional modules involved]

**Call paths (who calls this?):**
- [Direct callers from codegraph callers]
- [Top-level entry points — how does a user action reach this?]
- Example format:
  ```
  UserAction
    └→ ComponentX.onClick()
      └→ affectedFunction() [THE BUG IS HERE]
  ```

**Tests for this path:**
- Test file(s): `path/to/__tests__/FileName.test.ts`
- ✓ Tests that exist: [list specific test names]
- ✗ **Gap(s) found:** [what behavior is NOT tested?]
  - Example: "error-on-unavailable-scope case has no test"

**What breaks if we fix this wrong:**
- [Results from codegraph impact]
- [What else depends on this function/component?]
- [What guarantees could we accidentally break?]

**Related code (same area):**
- [Other functions/components in the same module with similar patterns]
- [Known code smells or anti-patterns nearby]

## Root cause hypothesis (pre-briefed by this intake)

**Likely culprit:** [One sentence. Name the exact function/line if known.]

**Why:**
- [Evidence from code graph]
- [How symptom matches this cause]

**Hypotheses eliminated:**
- [What did we rule out, and why?]
- [This prevents implementer from chasing wrong leads]

## What this blocks (impact)

- **User impact:** [What can't the user do?]
- **System impact:** [What downstream systems are affected?]
- **Data/auth risk:** [Is there data loss, corruption, or auth bypass? If yes, R2. If no, note it.]

## Known adjacent issues (code graph found these)

- [Other bugs or patterns in the same area]
- [Related components with similar smells]
- **Recommendation:** [Fix together, or separate tickets? Why?]

## Implementer roadmap (what to do)

Implementer reads this and knows exactly where to start:

1. **Locate the bug:**
   - File(s): [exact path(s)]
   - Search for: [grep pattern or function name]
   - Current code pattern: [brief description]

2. **Diagnose:**
   - Check: [specific question, e.g. "Does this function have X guard?"]
   - Likely finding: [what we expect they'll find]

3. **Implement fix (surgical):**
   - Change: [exact function/line]
   - From: [current pattern]
   - To: [fixed pattern]
   - Why: [explanation of the fix]

4. **Verify (manual):**
   - [ ] Run specific repro steps (from ticket)
   - [ ] Confirm: [exact behavior to check]
   - [ ] Ensure: [what should NOT change]

5. **Regression test (required):**
   - Add test to: `path/to/__tests__/FileName.test.ts`
   - Test should fail before fix, pass after
   - Test name: [e.g. "form retains fields when scope is unavailable"]

6. **Gate (copy-paste ready):**
   ```bash
   npm test -- path/to/affected.test.ts
   npm run typecheck -- path/to/affected.ts
   npm run lint -- path/to/affected.ts
   ```

## Build-side notes

### Risk triggers (from risk profile):

**R0 (trivial):**
- None. No guarantees at risk.

**R1 (routine):**
- List what's at stake: [e.g. "form data persistence", "component rendering"]
- One fresh reviewer required

**R2 (elevated):**
- **Guarantee(s) at stake:** [e.g. "scope isolation", "auth boundary", "data integrity"]
- **What we're preventing:** [the specific failure mode]
- **Contract:** [link GitHub issue if it has scope/acceptance criteria]

### Gate evidence (pre-run baseline):

Copy-paste the output of running gate BEFORE the fix was made:

```
✓ lint: 0 warnings in affected.ts
✓ typecheck: 0 errors in affected.ts
✓ tests: [N] passed, [N] failed (before fix)
⚠ known-flake: [flake name] in [test file] — already failing, not due to this bug
```

This gives implementer a baseline to compare against post-fix.

### Estimated scope:

- **Files to change:** ~[N] (based on codegraph impact)
- **Test impact:** ~[N] tests touch this area
- **New tests:** 1 regression test (required)
- **Deployment risk:** [low | medium | high] — brief reason
  - Low: isolated component, no data/auth, reverts easily
  - Medium: affects form/state but localized, tests catch issues
  - High: auth/permissions/data, touches multiple modules, hard to revert

### Known flakes:

- [None | list with file/test name and reason it's unrelated to this bug]

### Secondary questions (if any):

- [Optional: questions for the owner if something is ambiguous]
- [Example: "Should the error message say *why* scope is unavailable?"]
- If no questions: state "None — this is clear."
