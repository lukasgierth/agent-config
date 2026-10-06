---
name: dep-audit
description: Use when asked to audit, check, or update dependencies. Checks for outdated or vulnerable packages and proposes update commits.
---

# Dependency Audit

## Goal

Find outdated or vulnerable dependencies and propose targeted update commits.

## Workflow

1. Detect the package manager(s) in use by looking for lockfiles:
   - `uv.lock` / `pyproject.toml` → Python/uv
   - `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` → Node
   - `Cargo.lock` → Rust
   - `flake.lock` → Nix
2. Run the appropriate outdated/audit command:
   - uv: `uv tree --outdated`
   - npm: `npm outdated` and `npm audit --json`
   - yarn: `yarn outdated`
   - cargo: `cargo outdated` and `cargo audit`
3. Summarise findings in a table:

   | Package | Current | Latest | Severity |
   |---------|---------|--------|----------|

4. For each package worth updating, propose a commit using the git-commit-planner skill.
5. Do **not** blindly update everything — flag major-version bumps separately and ask the user before applying them.
