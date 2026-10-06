# Global Agent Rules

These rules apply to every opencode session. They are loaded alongside any
repo-local `AGENTS.md` found by walking up from the working
directory — nothing here is meant to override project-specific rules
silently. See section 8 for how conflicts are handled.

## 1. Empty repository handling

If the working directory is empty or contains no project files (no source,
no build config, no existing `AGENTS.md`):

1. Stop. Do not start scaffolding on assumption.
2. Create `AGENTS.md` in the repo root — via `/init` or by drafting one
   myself — populated with: project name, purpose, stack, build/lint/test
   commands, layout, conventions.
3. Then proceed to planning (section 2).

If the repo already has an `AGENTS.md`, update it in place rather than
replacing it.

## 2. Planning workflow (Agile)

For any non-trivial task — anything more than a single-file edit, a typo
fix, or a one-line config change — produce a `PLAN.md` at the repo root
(or `.opencode/PLAN.md` if a repo-local convention exists) containing:

- **Goal** — one-paragraph summary of what success looks like.
- **Scope** — what's in, what's out.
- **User Stories / Tasks** — Agile form:
  `As a <role>, I want <capability>, so that <value>.`
- **Acceptance Criteria / Definition of Done (DoD)** — every task MUST
  carry an explicit DoD with checkable conditions, e.g.:
  - Code merged / file created at `path:line`
  - Lint passes
  - Tests added and passing
  - Docs updated
  - Verified against the Acceptance Criteria
- **Task ordering / dependencies** — numbered, dependencies called out.
- **Risks & assumptions** — explicit, not implicit.
- **Out-of-band items** — discovered during planning, not dropped on the floor.

Keep `PLAN.md` scannable — headings + bullet lists, no prose walls.
Update it as scope changes; never let it drift from reality.

## 3. Clarification loop (`CIR-LOG.md`)

When planning reveals ambiguity — unclear requirements, missing context,
tradeoffs that need a human call — I will:

1. Append each open question to `CIR-LOG.md` at the repo root, in the form:
   ```
   ## Q-NN — <short title>
   - Context: <why this matters>
   - Options considered: <A, B, C>
   - Recommendation: <which and why>
   - Status: open | resolved — <answer>
   ```
2. Surface the highest-priority open questions to the user **in chat**
   using the `question` tool — one batch per turn, top items first,
   recommendation marked.
3. Do not invent answers. Do not pick silently among listed options.
4. Once the user answers, update the entry's `Status` line and remove it
   from the open queue on the next turn.
5. `CIR-LOG.md` is append-only during a session; it gets pruned/archived
   at the start of the next planning cycle.

## 4. Human gate before implementation

Hard rule: **after `PLAN.md` is finalized but before any code, config, or
other implementation artifact is written or edited, stop and request
explicit human approval.**

What the human gate DOES cover:
- Source code edits and creation
- Build/CI/IaC config changes
- Dependency installs/updates
- Any tool invocation that mutates state outside the repo (deploys, cluster
  changes, push, etc.)

What the human gate does NOT cover (freely allowed during planning):
- Writing or updating `PLAN.md`
- Writing or updating `CIR-LOG.md`
- Writing or updating `AGENTS.md`
- Writing or updating repo-local documentation/markdown other than code
- Read-only operations: file reads, searches, git status/diff/log

The approval request must include:
- Pointer to `PLAN.md` (path + section)
- Summary of open items in `CIR-LOG.md` (if any)
- Any deviations from the original ask
- The exact first concrete step I am about to take

After approval of a task, follow-up edits within that same task may proceed
without re-asking. Approval of task N does not pre-approve task N+1.

## 5. House style (carry-over from base config)

- Be concise. Prefer fewer lines over more.
- No comments in code unless the user asks.
- Reference code as `file_path:line_number`.
- Prefer Glob/Grep/Read over `find`/`grep`/`cat`.
- Never commit unless explicitly asked. Never push. Never force.
- Never expose or commit secrets.
- No emojis unless the user requests them.

## 6. Tool / MCP conventions

- `jcodemunch-mcp`, `jdatamunch`, `jdocmunch-mcp` are disabled by config.
  Enable per-session only when needed; ask before turning on.
- `ponytail` plugin is active for over-engineering review. Apply its
  "simplest thing that works" guidance unless the user opts out.

## 7. Nix-managed config awareness

`~/.config/opencode/opencode.json` and `tui.json` are symlinks into the
Nix store (home-manager). Do not edit them in place — changes will be
overwritten on the next home-manager switch. If a config change is
needed, surface it to the user so it goes into the home-manager flake.

## 8. Layering and conflict resolution with local AGENTS.md

When working inside a repo, opencode loads instruction files in this order:
1. Repo-local `AGENTS.md` (walking up from cwd)
2. This global `~/.config/opencode/AGENTS.md`

Both are read in full. They are layered, not mutually exclusive — the global
file is the default behavior, the local file is the project-specific override.

### When local and global agree
No action needed; behave as instructed.

### When local and global conflict
**Stop and ask the user.** Do not guess which to follow. Use the `question`
tool to surface the conflict, quoting the exact lines from each file and
letting the user pick. Record the resolution in `CIR-LOG.md` so it survives
future sessions.

Conflict cases include (but are not limited to):
- Different rules for the same workflow (e.g. planning format, DoD style)
- Different tool/permission boundaries
- Different commit or commit-message policies
- Different style requirements (comments, naming, formatting)

### When only one of the two covers a topic
Apply the one that covers it. No conflict, no question.

### When local rules are absent for a workflow
Default to the global rules in this file.
