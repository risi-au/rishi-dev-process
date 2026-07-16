# New machine setup — bootstrap the dev environment
<!-- CAP: 160 lines. For a fresh Windows PC. Point any agent here:
     "Read project-setup/new-machine.md in the harness and set this PC up."
     Steps marked [OWNER] need the human (logins, UAC, secrets) — the agent
     prepares the command, the owner runs it. Everything else the agent does. -->

The agent works through the phases in order and finishes with the verification
checklist. Don't guess install commands that fail — fetch current instructions
from the official source listed for each tool.

## Phase 1 — Core tooling

1. Git — winget or git-scm.com. Then set identity (a past machine skipped this
   and every commit failed):
   `git config --global user.name "risi-au"` and
   `git config --global user.email "brissiedigital@gmail.com"`
2. GitHub CLI (`gh`) — winget `GitHub.cli`. **[OWNER]** `gh auth login`
   (browser flow; scopes need `repo`, `workflow`).
3. Node.js LTS (for codex + tooling) — winget `OpenJS.NodeJS.LTS`.
4. Python 3.11+ — winget or python.org. Always use `python -m pip ...` (a past
   machine had `pip` pointing at a different Python than `python`).

## Phase 2 — The harness (before any agent CLIs)

```
gh repo clone risi-au/rishi-dev-process <chosen-path>
```

Recommended path mirrors the original: `G:\PROJECTS\Rishi-Setup\rishi-dev-process`
(any path works; use the local clone path in session prompts).

## Phase 3 — Agent CLIs (the worker pool)

Install only what the owner's current `models/REGISTRY.md` lists as active —
check it first, don't install everything by reflex.

| Worker | Install source | Auth step |
|---|---|---|
| Claude Code | claude.com/claude-code (or `irm https://claude.ai/install.ps1 \| iex`) | **[OWNER]** `claude` first run → login |
| Codex | github.com/openai/codex (`npm install -g @openai/codex`) | **[OWNER]** `codex login` |
| Grok CLI | xAI's current install docs (grok.x.ai) | **[OWNER]** first-run auth |
| Orca (optional runtime) | Owner provides the installer | — |

After each install, verify: `<cli> --version`.

## Phase 4 — Codex Windows sandbox (do NOT skip)

Codex's elevated Windows sandbox needs a ONE-TIME admin install. If skipped or
UAC-declined, every codex dispatch relaunches the installer, pops UAC, and hangs
headless agents (this silently ate hours on the original machine).

1. **[OWNER]** Run elevated and click Yes on UAC:
   `Start-Process -FilePath "$env:LOCALAPPDATA\OpenAI\Codex\bin\<hash>\codex-windows-sandbox-setup.exe" -Verb RunAs -Wait`
   (find `<hash>`: the single dir under `$env:LOCALAPPDATA\OpenAI\Codex\bin\`)
2. Verify (free, no tokens): `codex sandbox -- cmd /c "echo SANDBOX-OK"` —
   must print SANDBOX-OK within seconds, no popup.

Full details and history: `models/codex.md`.

## Phase 5 — Supporting tools

1. graphify (code knowledge graphs): `python -m pip install graphifyy`
   (double-y is the real package; verified non-typosquat 2026-07) then
   `graphify install`. Per-project indexing happens per project, not here.
2. Syncthing — if this machine joins the owner's working-file sync mesh.
   **[OWNER]** device pairing.
3. Docker Desktop — only if a project overlay requires it (e.g. companyos
   postgres). Check project docs before installing.

## Phase 6 — Projects

For each project this machine will work on:

1. `gh repo clone <owner>/<repo>` to its conventional path (companyos lives at
   `C:\dev\companyos` on the original machine).
2. Confirm the harness pointer exists in the project's AGENTS.md
   (`project-setup/README.md` has the block).
3. **[OWNER]** Provide `.env` / secrets from the vault — the agent never
   fetches, copies, or reads secret values; it only confirms the file exists.
4. Optional: build the project's code graph — `/graphify <project path>`
   (exclude junk dirs; zero tokens for code-only).

## Phase 7 — Verification checklist (agent runs all, reports a table)

```
git config user.name && git config user.email     -> set
gh auth status                                    -> logged in
claude --version                                  -> prints version
codex --version                                   -> prints version
codex sandbox -- cmd /c "echo SANDBOX-OK"         -> SANDBOX-OK, no UAC
grok --version                                    -> prints version (if active)
graphify --version                                -> prints version
```

Then run each active worker's full preflight from `models/<worker>.md`
(the codex exec line costs a few thousand codex tokens — that's expected).

Finish with a short report: what's installed, what was skipped and why, and any
step that deviated from this guide → propose the fix as a retro
(`core/SELF-IMPROVE.md`) so this guide stays accurate.

## Known machine quirks worth carrying forward

- `pip` vs `python` version mismatch → always `python -m pip`.
- npm-shim CLIs (codex) may need cmd.exe resolution in some spawn contexts.
- PowerShell + long-running child processes: prefer Bash for `--wait`-style
  polling (see `models/orca.md`).
