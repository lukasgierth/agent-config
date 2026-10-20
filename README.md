# agent-config

Personal configuration for AI coding agents — opencode, pi, and goose. Contains the global
`AGENTS.md` rule set and the custom skills written for it.

## Layout

| Path | What it is |
|------|-----------|
| `AGENTS.md` | Global agent rules: planning workflow, CIR-LOG clarification loop, human gate, house style, MCP/Nix conventions, conflict resolution. Source for the global `AGENTS.md` deployed to `~/.config/opencode/AGENTS.md` and `~/.pi/agent/AGENTS.md` (home-manager) and copied to `~/.config/goose/AGENTS.md`. |
| `skills/` | Custom skills, one directory per skill in the shared Agent Skills `<name>/SKILL.md` format, discovered by opencode, pi, and goose from `~/.agents/skills/`. |

## Skills

| Skill | Purpose |
|-------|---------|
| `add-tests` | Write tests for a function/module following the repo's existing test style and framework. |
| `dep-audit` | Detect package manager, check for outdated/vulnerable dependencies, propose update commits. |
| `explain-codebase` | Produce a concise structured overview of a project: entry points, modules, data flow, gotchas. |
| `git-commit-planner` | Split uncommitted changes into small focused conventional commits; always asks before committing. |
| `release-notes` | Draft human-readable release notes from a git tag range, grouped by conventional commit type. |
| `update-nix-packages` | Bump custom derivations under a flake's `packages/` to latest upstream via `nix-update`. |
| `update-docs` | Keep Markdown docs in sync with the repo: root `AGENTS.md`/`README.md`, subfolder docs, and new READMEs for undocumented code. |

## Install

The global `AGENTS.md` is read by all three agents:

- opencode: `~/.config/opencode/AGENTS.md` — home-manager symlink
- pi: `~/.pi/agent/AGENTS.md` — home-manager symlink
- goose: `~/.config/goose/AGENTS.md` — copied by home-manager activation
  (not a symlink)

Do not edit the deployed copies in place — symlinks and activation copies are
overwritten on the next home-manager switch; change the files here.

Skills are installed once and shared by all three: symlink or copy each
`skills/<name>` directory into `~/.agents/skills/`.

## Editing rules

Follow the rules in `AGENTS.md` when changing this repo: non-trivial work gets
a `PLAN.md`, ambiguity goes to `CIR-LOG.md`, and no implementation without
explicit approval.
