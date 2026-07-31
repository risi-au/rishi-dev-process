# BUG-INTAKE — Enhanced bug reporting with structural analysis

**Purpose:** Before an implementer is dispatched on a bug fix, prepare a "pre-briefed ticket" that has already done the expensive thinking — structural analysis, impact scope, root cause investigation, and risk assessment. This saves the implementer from burning tokens rediscovering what we already know.

**When to use:** R1+ bugs and multi-file changes. R0 trivial bugs skip this entirely.

---

## Quick flow

```
REPORT (screenshot + context) 
  ↓
INITIAL TRIAGE (confirm it's a bug)
  ↓
[FOR R0 ONLY: skip to dispatch]
[FOR R1+: continue below]
  ↓
STRUCTURAL ANALYSIS (code-review-graph queries)
  ↓
PRE-BRIEF TICKET (fill template below)
  ↓
DISPATCH (implementer reads ticket, knows exactly what to do)
```

---

## Structural Analysis Phase (for R1+ bugs)

### Tools to use (in order)

1. **code-review-graph queries** (answers: who calls this? what breaks? are there tests?)
   ```bash
   code-review-graph query_graph_tool --operation callers_of --target "affected_function"
   code-review-graph query_graph_tool --operation tests_for --target "affected_function"
   code-review-graph get_impact_radius_tool --target "affected_function"
   ```

2. **graphify query** (only if R2 or cross-module concern)
   ```bash
   graphify query "What is the relationship between [affected system] and [related system]?"
   ```

### What to extract from the results

- **Affected component(s):** Exact file paths, component/function names
- **Call paths:** Who calls this? What's the full execution chain?
- **Test coverage:** Do tests exist for this path? What's the gap?
- **Impact radius:** What else breaks if we change this?
- **Related code:** Are there adjacent functions with similar smells?

---

## Enhanced Ticket Template

Use this instead of the raw screenshot + description:

```markdown
---
id: NNN
title: [one-line title]
status: new
severity: [blocking | wrong | annoying]
reported: YYYY-MM-DD
github-issue:
risk-profile: [R0 | R1 | R2]
---

## What happened (Rishi's words, lightly cleaned)

[Concise description of symptom, as before]

## Expected

[What should happen, as before]

## Initial context (code-graph results)

**Affected component(s):**
- File: `path/to/file.ts:functionName()`
- [related files]

**Call paths (who calls this?):**
- [results from query_graph_tool callers_of]
- [top-level entry points]

**Tests for this path:**
- [results from query_graph_tool tests_for]
- [coverage gaps: which test categories missing?]

**What breaks if we change this:**
- [results from get_impact_radius_tool]
- [dependent code, cascade risk]

**Related code (same area):**
- [other functions with similar patterns or smells]

## Root cause hypothesis (pre-briefed by this intake)

**Likely culprit:** [one sentence]

**Why:** [evidence from code graph + symptom]

**Hypotheses eliminated:** [what did the graph rule out?]

## What this blocks (impact)

- [User impact]
- [System impact]
- [Data/auth risk, if any]

## Known adjacent issues (code graph found these)

- [Other related bugs or patterns]
- [Recommendation: fix together or separately?]

## Implementer roadmap (what to do)

1. [Step 1: locate/diagnose]
2. [Step 2: implement]
3. [Step 3: verify]
4. [Step 4: test]
5. [Step 5: gate commands]

## Build-side notes

### Risk triggers (from risk profile):
- [R2 only: named guarantees at stake]
- [R0: none]

### Gate evidence (pre-run):
- [results of running gate before fix]

### Estimated scope:
- Files to change: [N]
- Test impact: [description]
- Deployment risk: [low | medium | high]

### Known flakes:
- [None | list with reasons]
```

---

## Decision Tree: When to do structural analysis

| Scenario | Triage | Structural Analysis? | Why |
|---|---|---|---|
| Button text wrong, CSS fix | R0 | No | Trivial. Implementer finds the file in 30 seconds. |
| Single component re-render bug, no auth/data | R1 | Yes | Small scope but needs to understand callers. 10 min of queries saves implementer 30 min of Grep. |
| Permission/auth/data bug | R2 | Yes | MUST. Impact radius prevents introducing new vulns. |
| Cascading state mutation | R1–R2 | Yes | Cross-file; hand-tracing is expensive. Graph tells the story. |
| One-liner SQL bug | R0 | No | Obvious. But if it's a pattern, upgrade to R1 and do analysis. |

---

## Token Economics

**Triage agent runs graph queries:** +2k–5k tokens per ticket
- `query_graph_tool`: ~1k tokens (fast; results are snippets, not whole files)
- `get_impact_radius_tool`: ~1.5k tokens
- `graphify query`: ~1.5k tokens (only if R2/cross-module)

**Implementer no longer needs to rediscover:** -4k–8k tokens saved per ticket
- No Grep of callers (saved ~2k)
- No reading 5–10 files to understand scope (saved ~3k)
- No guessing at test coverage (saved ~1k)

**Net per ticket:** -1k to +2k, depending on complexity. Positive ROI if the implementer is expensive (high-effort model) or if parallelizing (triage agent is cheap tier; implementer is mid).

---

## Example: How to Apply This

**Scenario:** User reports "Invite form clears Name/Email when I select a scope that's unavailable."

### Step 1: Initial triage (30 sec)
- It's a bug (not idea/miss)
- Severity: wrong (data loss)
- Size: Standard (multi-file form logic)
- Risk: R1 (form state, user data)

### Step 2: Run graph queries (5 min)
```bash
# Find the form component
code-review-graph semantic_search_nodes_tool --keyword "InvitePersonForm"
# Result: packages/os/src/components/ScopeSettings/InvitePersonForm.tsx

# Who calls it
code-review-graph query_graph_tool --operation callers_of --target "InvitePersonForm"
# Result: ScopeSettings page (one location; not reused)

# What tests exist
code-review-graph query_graph_tool --operation tests_for --target "InvitePersonForm"
# Result: InvitePersonForm.test.tsx exists; test for "field retained on error" is MISSING

# What breaks if we change the state handler
code-review-graph get_impact_radius_tool --target "onScopeChange"
# Result: Only affects this form's state; no cross-component impact
```

### Step 3: Pre-brief the ticket
- Fill "Initial context" with graph results: file path, callers (one), tests (gap found), impact (low)
- Root cause hypothesis: state merge bug (setter clears entire form instead of just error)
- Implementer roadmap: diagnose → fix state merge → add regression test → run gate

### Step 4: Dispatch
Implementer reads: "Here's the file. Here's the bug pattern. Here's the test you need to write. Go."
They never Grep. They never wonder "what else calls this." They start coding in 30 seconds.

---

## Triage Checklist (for whoever files the ticket)

- [ ] Confirmed it's a bug (not idea/knowledge-miss)
- [ ] Assigned severity + risk profile
- [ ] For R0: skip to dispatch
- [ ] For R1+: ran code-review-graph queries (3–5 min)
- [ ] Filled "Initial context" section with graph results
- [ ] Wrote one-sentence root cause hypothesis (backed by analysis)
- [ ] Listed what breaks if we're wrong (impact assessment)
- [ ] Pre-ran relevant gate (baseline for implementer)
- [ ] Filled "Implementer roadmap" section (step-by-step)
- [ ] Estimated scope (files, tests, deployment risk)

If you skip structural analysis for R1+ or miss sections → file in project's INBOX instead of a full ticket. The build side will triage it when capacity allows.

---

## Project specifics

Projects may override this in their `AGENTS.md`:
- Which graph tool to use for their codebase (code-review-graph path, graphify index location)
- Which bugs require structural analysis (default: R1+)
- Template variations (e.g. add security checklist for auth bugs)

Check project overlay before starting.
