# Product Contract: <name>
<!-- HARD CAP: 1 page. The owner approves this BEFORE planning or code.
     This page — not the plan — is what every worker and reviewer anchors to. -->

**Purpose** (1–2 sentences): what this is and who it serves.

**In scope (V1):**
- behavior 1
- behavior 2

**Explicitly OUT of scope:**
- exclusion 1  <!-- e.g. "this is NOT a file-sync system" — boundary lines live here -->

**Safety invariants** (what it must NEVER do):
- e.g. never overwrite local work; never auto-delete; never push

**Acceptance checks** (each one verifiable):
- [ ] check 1
- [ ] check 2

**Deployment boundary:** where it runs, as which user, what it may touch.
("None — local tool" is a valid answer.)

**Risk profile:** R0 | R1 | R2 — triggers: <list, or "none">
