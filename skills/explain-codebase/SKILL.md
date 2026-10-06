---
name: explain-codebase
description: Use when asked to explain, document, or give an overview of a codebase or project structure. Produces a structured summary of entry points, key modules, and data flow.
---

# Explain Codebase

## Goal

Produce a clear, structured onboarding overview of the current project.

## Workflow

1. Read the root directory listing, then key config files (`package.json`, `pyproject.toml`, `Cargo.toml`, `flake.nix`, `justfile`, `Makefile`, etc.) to understand the project type and tooling.
2. Identify entry points (main files, CLI scripts, binaries).
3. Map out key modules/packages and what each one is responsible for.
4. Describe the main data flow or request lifecycle in 3-5 steps.
5. Note any non-obvious conventions or gotchas found in the code.

## Output Format

```
## Project: <name>

**Type:** <language/framework>
**Entry points:** <files>

### Module Overview
| Module | Responsibility |
|--------|---------------|
| ...    | ...           |

### Data Flow
1. ...

### Conventions & Gotchas
- ...
```

Keep the output concise — aim for something a new contributor can read in under 5 minutes.
