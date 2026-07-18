# Onboarding a project to the harness

## 1. Pointer in the project's AGENTS.md

Add this block (or create AGENTS.md with it). AGENTS.md is the cross-vendor
standard — Claude Code, Codex, Cursor, Gemini CLI, and 25+ other tools read it.

```markdown
<!-- HARNESS:START -->
## Dev process
All dev work follows the master harness. Read START.md at
https://github.com/risi-au/rishi-dev-process
(local clone: G:\PROJECTS\Rishi-Setup\rishi-dev-process) BEFORE working.
The rest of this file is the PROJECT overlay (subject-matter rules, commands,
architecture); the harness governs process.
<!-- HARNESS:END -->
```

Only edit between the markers when updating the pointer. If the project has a
`CLAUDE.md`, make it a one-liner: `See AGENTS.md.`

## 2. Project overlay content (stays in the project, never in the harness)

Project-specific facts: build/gate commands, architecture, module maps, hard
rules (untouchable paths), environment quirks, issue-board conventions. The
harness stays generic; the overlay carries everything the harness calls
"see project docs."

## 3. Session start prompt (any model, any platform)

```
Read <harness path or repo URL>/START.md and follow it fully — including the
Session Brief before any code (wait for my reply) and the session close ritual
in core/SELF-IMPROVE.md (finish report -> honest retro -> propose improvements
-> apply what I approve to the harness and push).
Lane: <lanes/*.md, or "triage it">.
Project: <canonical path, e.g. C:\dev\companyos>.
Workspace: <open folder / Orca worktree cwd — product work happens here>.
Task: <description or issue #N>.
Code graphs (if any — honor project AGENTS.md; drop lines that do not apply):
  - Locate / architecture: graphify against MAIN absolute path
    (graphify-out is gitignored; Orca worktrees lack it):
      graphify query "<q>" --graph "<MAIN>\graphify-out\graph.json"
    before bulk-reading files. See models/graphify.md.
  - Diff / PR blast radius / review risk: code-review-graph in THIS workspace
    (per-worktree .code-review-graph/; MCP inherits cwd; build once if empty).
    Do NOT hard-code MAIN as CRG cwd. See models/code-review-graph.md.
  - One graph system per question — do not run both for the same ask.
```

## 4. New machine

Clone the harness once: `git clone https://github.com/risi-au/rishi-dev-process`
and use the local path in prompts. Pull before big sessions.
