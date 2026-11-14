# Global Agent Rules

**Before any file edit, config change, or dependency install: write `PLAN.md`, then stop
and wait for explicit human approval.** Read-only work needs no plan. If a task already
reached an edit without a plan, stop there, say so, and write the plan first.

Applies to every opencode, pi, and goose session, alongside any repo-local `AGENTS.md`
found walking up from the working directory. Layering and conflicts: section 8.

## 1. Empty repository handling

If the working directory is empty or contains no project files (no source,
no build config, no existing `AGENTS.md`):

1. Stop. Do not start scaffolding on assumption.
2. Create `AGENTS.md` in the repo root — via the tool's init flow (e.g.
   opencode's `/init`) or by drafting one myself — populated with: project
   name, purpose, stack, build/lint/test commands, layout, conventions.
3. Then proceed to planning (section 2).

If the repo already has an `AGENTS.md`, update it in place rather than
replacing it.

## 2. Planning workflow (Agile)

For **every** task — a single-file edit, a typo fix, and a one-line config
change included — produce a `PLAN.md` at the repo root (or a tool-specific
path such as `.opencode/PLAN.md`, if that is the repo convention) before
touching any file. It contains:

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

Once `PLAN.md` is final and before any code, config, or other implementation artifact is
written or edited or a dependency is installed, **stop and request explicit human
approval**. Gated: source edits, build/CI/IaC changes, dependency installs, and anything
that mutates state outside the repo (deploy, cluster change, push). Not gated: `PLAN.md`,
`CIR-LOG.md`, `AGENTS.md`, other repo markdown, and read-only operations.

The approval request names the `PLAN.md` section, any open `CIR-LOG.md` items, deviations
from the ask, and the exact first step.

Approval covers follow-up edits within that task. Approval of task N does not approve task
N+1.

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

opencode's `~/.config/opencode/` (`opencode.json`, `tui.json`, `AGENTS.md`)
and pi's `~/.pi/agent/` (`AGENTS.md`, `settings.json`) are symlinks into the
Nix store (home-manager). goose's `~/.config/goose/AGENTS.md` is not a
symlink but is copied there by home-manager activation. Custom skills in
`~/.agents/skills/<name>/SKILL.md` are home-manager symlinks into the Nix
store as well. Do not edit these deployed copies in place — they are
overwritten on the next home-manager switch. Surface a change to the user so
it goes into the home-manager flake.

## 8. Layering and conflict resolution with local AGENTS.md

Both files are read in full: repo-local `AGENTS.md` first (walking up from cwd), then this
one — `~/.config/opencode/AGENTS.md`, `~/.pi/agent/AGENTS.md`, or
`~/.config/goose/AGENTS.md` (goose also reads `.goosehints`). Layered, not mutually
exclusive: this file is the default behavior, the local file the project override.

- **They agree** → behave as instructed.
- **Only one covers the topic** → apply that one.
- **They conflict** → stop and ask. Do not guess. Quote the exact lines from each file,
  let the user pick with the `question` tool, and record the resolution in `CIR-LOG.md`.
  Conflicts include planning format, DoD style, tool/permission boundaries, commit and
  commit-message policy, and style requirements.
