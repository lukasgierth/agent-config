# agent-config

Personal configuration for AI coding agents (opencode). Contains the global
`AGENTS.md` rule set and the custom skills written for it.

## Layout

| Path | What it is |
|------|-----------|
| `AGENTS.md` | Global agent rules: planning workflow, CIR-LOG clarification loop, human gate, house style, MCP/Nix conventions, conflict resolution. Source for the home-manager-managed `~/.config/opencode/AGENTS.md`. |
| `skills/` | Custom skills, one directory per skill in opencode's `<name>/SKILL.md` format. |

## Skills

| Skill | Purpose |
|-------|---------|
| `add-tests` | Write tests for a function/module following the repo's existing test style and framework. |
| `dep-audit` | Detect package manager, check for outdated/vulnerable dependencies, propose update commits. |
| `explain-codebase` | Produce a concise structured overview of a project: entry points, modules, data flow, gotchas. |
| `git-commit-planner` | Split uncommitted changes into small focused conventional commits; always asks before committing. |
| `release-notes` | Draft human-readable release notes from a git tag range, grouped by conventional commit type. |
| `update-nix-packages` | Bump custom derivations under a flake's `packages/` to latest upstream via `nix-update`. |

## Install

- `AGENTS.md` is deployed to `~/.config/opencode/AGENTS.md` via home-manager
  (do not edit the symlinked copy in the Nix store — change it here).
- Skills are installed by symlinking or copying each `skills/<name>` directory
  into `~/.agents/skills/`.

## Editing rules

Follow the rules in `AGENTS.md` when changing this repo: non-trivial work gets
a `PLAN.md`, ambiguity goes to `CIR-LOG.md`, and no implementation without
explicit approval.