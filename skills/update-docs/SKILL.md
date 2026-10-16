---
name: update-docs
description: Use when asked to update, refresh, or fix documentation — "update the docs", "refresh the README", "document this repo", "keep the docs in sync". Updates the root AGENTS.md and README.md, updates every existing Markdown file in subfolders that describes code, and creates README.md files for subfolders that have code but no docs.
---

# Update Docs

## Goal

Bring the repo's Markdown documentation back in line with reality: root `AGENTS.md` and `README.md`, every existing `README.md`/Markdown doc in subfolders, and newly created `README.md` files for undocumented subfolders that contain code worth documenting.

Ground rule: **docs describe the repo as it is now.** Every command, path, and responsibility you write must be verified against the actual code and config. Never invent features, commands, or file names.

## Workflow

### 1. Discover the doc surface

```bash
git ls-files '*.md'          # tracked markdown, respects .gitignore
```

Also list directories to find code with no docs:

```bash
git ls-files | cut -d/ -f1 | sort -u   # top-level dirs (and root files)
```

Build a list of:

- Root `AGENTS.md` and `README.md`.
- Every Markdown file in subfolders that describes code (module READMEs, `docs/`, architecture notes).
- Directories containing source but **no** `README.md` (or an empty one).

**Exclude** (unless the user explicitly asks): `.git`, hidden dirs, vendored/build deps (`node_modules`, `vendor`, `dist`, `build`, `target`, `result`, `.venv`), and auto-generated docs (changelogs, CI-published API docs).

### 2. Verify before you write

- Read the root config to learn the real toolchain: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `flake.nix`, `justfile`, `Makefile`, CI workflows.
- Confirm every command you reference exists and is correct (`just check`, `npm test`, etc.).
- Read each file you're about to update before rewriting it.

### 3. Update the root `AGENTS.md`

Keep it in sync with how the project actually works, **in place** — preserve the existing rules and tone:

- Build / lint / test commands (verify against step 2).
- Repo layout and conventions that changed.
- Do not rewrite the user's rules wholesale; only fix facts and fill gaps.

### 4. Update the root `README.md`

Ensure it has, and keeps accurate:

- Project name and a one-paragraph purpose.
- A layout table (path → what it is) matching the actual tree.
- Install / build / test / run commands.
- Links to key subfolder docs.
- Any index/tables of components or skills — keep them current.

### 5. Update subfolder docs

For each existing Markdown file from step 1:

- Rewrite module/responsibility tables from the real code.
- Fix entry points, data-flow sections, and examples that no longer match.
- Prefer a single source of truth: link to the root README instead of duplicating.

### 6. Create missing docs

For each subfolder with source code but no `README.md`, create a minimal one:

- One-line purpose, entry points (main files/CLI), how to test it.
- Keep it to ~5-10 lines unless the folder is genuinely complex.

Name it `README.md` unless the folder has an existing docs convention.

### 7. Consistency pass

- Fix links/breadcrumbs broken by renames or moves.
- Drop stale sections that describe deleted code.
- Keep one source of truth per fact; point elsewhere rather than duplicating.

### 8. Skip planning artifacts

Do **not** touch `PLAN.md`, `CIR-LOG.md`, `GRILL-ME.md`, or the planning-workflow rules inside `AGENTS.md` unless the user asks. Leave `CHANGELOG.md` to its dedicated workflow.

### 9. Report

End with a summary table:

| File | Action |
|------|--------|
| `README.md` | updated |
| `src/foo/README.md` | created |
| `docs/api.md` | unchanged |
| `vendor/` | skipped (ignored) |

## Shortcuts

- **No docs request specificity**: if the user just says "update docs", run the full workflow. If they name a file or folder, scope to that plus anything it links to, and say what you left out.
