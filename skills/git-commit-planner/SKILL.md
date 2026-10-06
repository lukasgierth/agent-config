---
name: git-commit-planner
description: Use when asked to commit, stage, or create git commits. Reviews uncommitted changes in a git repo, plans how to split them into small focused commits, and creates each commit following conventional commits naming. Do not use for general git operations unrelated to committing.
---

# Git Commit Planner

## Goal

Review all uncommitted changes, group them into small, focused commits, and create each commit individually using conventional commits naming.

## Workflow

### 1. Inspect Changes

Run these commands to understand what has changed:

```bash
git status --short
git diff          # unstaged changes
git diff --cached # staged changes
```

For each modified file, read the full diff to understand the intent.

### 2. Plan Commits

Group changes into the **smallest logical units** that still make sense on their own. Each commit should:

- Contain one cohesive change (one feature, one fix, one refactor, etc.)
- Be independently understandable without reading other commits
- Not bundle unrelated changes together

Think: "If someone bisects to this commit, does it represent one clear change?"

Prefer **many small commits** over one large commit. When in doubt, split.

### 3. Conventional Commit Naming

Every commit message must follow this format:

```
<type>[optional scope]: <short description>
```

Types:
- `feat:` — new feature or capability
- `fix:` — bug fix
- `chore:` — maintenance (deps, tooling, config)
- `docs:` — documentation only
- `refactor:` — restructuring without behavior change
- `style:` — formatting, whitespace, code style
- `test:` — adding or updating tests
- `ci:` — CI/CD pipeline changes
- `perf:` — performance improvements
- `revert:` — reverts a previous commit

Scope (optional) is a short noun in parentheses indicating what was affected, e.g. `feat(auth):`, `fix(networking):`.

Rules:
- Description is lowercase, no trailing period
- Keep the subject line under 72 characters
- Use imperative mood: "add support for X", not "added support for X"

### 4. Present the Plan

Before creating any commits, present the full plan to the user:

- List each proposed commit with its message and which files/hunks it covers
- Use the Question tool to ask for confirmation with **Yes** and **No** choices (do not use free-text prompts)
- If the user selects **No**, stop and do not create any commits

### 5. Create Commits

For each commit in the plan (in order):

1. Stage only the files/hunks for that commit (`git add <files>` or `git add -p` for partial staging)
2. Run `git commit -m "<message>"`
3. Verify the commit was created successfully

Do **not** create all commits at once. Stage and commit one at a time.

## Important Rules

- **Never commit without explicit user confirmation** — always use the Question tool with Yes/No choices, never free-text prompts
- Do not amend existing commits
- Do not force-push
- Do not commit secrets or credentials
- If pre-commit hooks reject a commit, fix the issue and retry — do not skip hooks

### 6. AI Co-Author Trailer

When any part of the changes being committed was written, generated, or
significantly edited by an AI assistant, add a `Co-authored-by:` trailer
in the commit message **body** (after a blank line following the
subject). It is not part of the conventional commit header.

Format:

```
<type>[scope]: <short description>

<optional body>

Co-authored-by: <tool>/<model>
```

Examples:

```
feat(inventory): add cmdb distance sorting

Co-authored-by: opencode/qwen3.8-27b
```

```
fix: handle missing os_version in post-run table

Co-authored-by: opencode/qwen3.8-27b
```

```
refactor: extract haversine helper

Co-authored-by: opencode/qwen3.8-27b
```

Always use the form `Co-authored-by: <tool>/<model>` — fill in the
actual tool name (e.g. `opencode`) and the actual model identifier (e.g.
`qwen3.8-27b`, `minimax-m3`). Do not use `Assisted-by:` and do not omit
either segment.

If a human authored every line, omit the trailer.
