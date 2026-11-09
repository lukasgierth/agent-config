---
name: dep-audit
description: Use when asked to audit, check, or update dependencies. Checks for outdated or vulnerable packages, updates them one at a time, verifies the build, and proposes update commits.
---

# Dependency Audit

## Goal

Find outdated or vulnerable dependencies, update them safely, and propose one focused
commit per update.

## Workflow

### 1. Detect the package manager(s)

| Lockfile | Ecosystem |
|----------|-----------|
| `uv.lock`, `pyproject.toml` | Python / uv |
| `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` | Node |
| `Cargo.lock` | Rust |
| `go.mod` | Go |
| `flake.lock` | Nix — out of scope, use the `update-nix-packages` skill |

More than one can apply; audit each.

### 2. Audit

Outdated:

| Ecosystem | Command |
|-----------|---------|
| uv | `uv tree --outdated` (older uv: `uv pip list --outdated`) |
| npm | `npm outdated` |
| yarn | `yarn outdated` |
| pnpm | `pnpm outdated` |
| cargo | `cargo outdated` |
| go | `go list -m -u all` |

Vulnerabilities:

| Ecosystem | Command |
|-----------|---------|
| uv | `uvx pip-audit` |
| npm / pnpm | `npm audit --json` / `pnpm audit` |
| yarn | `yarn npm audit` (berry) |
| cargo | `cargo audit` |
| go | `go run golang.org/x/vuln/cmd/govulncheck@latest ./...` |

`cargo outdated` and `cargo audit` are not built in. If either is missing, use
`nix shell nixpkgs#cargo-outdated nixpkgs#cargo-audit -c cargo ...`, or
`cargo install --locked cargo-outdated cargo-audit`. Never skip the audit silently.

### 3. Summarise

| Package | Current | Latest | Severity | Update? |
|---------|---------|--------|----------|---------|

Sort by severity (vulnerable first), then by version gap.

### 4. Update one package at a time

| Ecosystem | Command |
|-----------|---------|
| uv | `uv add <pkg>@latest`, or `uv lock --upgrade-package <pkg>` |
| npm | `npm install <pkg>@latest` |
| yarn | `yarn up <pkg>` |
| pnpm | `pnpm update <pkg>` |
| cargo | `cargo update -p <pkg>` |
| go | `go get <pkg>@latest && go mod tidy` |

### 5. Verify

After each update, run the project's build and test command. If either fails, revert
that package's change (`git checkout -- <lockfile>`) and report it as failed — do not
push a broken dependency tree forward.

### 6. Commit

Propose one commit per updated package using the `git-commit-planner` skill. Group
patches of unrelated packages only when they are trivial and verified together.

## Rules

- Do not blindly update everything: flag major-version bumps (`Current` major != `Latest`
  major) separately and ask before applying them.
- A vulnerability with no released fix goes in the report as "no fix available", with the
  affected package and version — do not pin to an unreleased commit.
- Never edit lockfiles by hand.

## Stop and ask

- No lockfile, or a package manager not in the tables above.
- The audit tool is missing and no fallback works.
- A major-version bump is the only way to fix a vulnerability.
- The build or tests stay red after reverting the update.
