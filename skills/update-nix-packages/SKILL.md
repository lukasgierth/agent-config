---
name: update-nix-packages
description: Use when asked to update, bump, or refresh versions of custom Nix package derivations in a `packages/` directory inside a Nix flake repository. Triggers on phrases like "update packages", "bump nix packages", "refresh local packages", or naming a specific package such as "update surge-downloader" in a flake-based project.
---

# Update Nix Packages

## Goal

Refresh every custom package derivation under `packages/` to its latest upstream version, without touching flake inputs.

## Workflow

1. **Verify flake context.** Confirm the working directory contains `flake.nix` and `flake.lock`. If not, stop and tell the user this skill only applies to flake-based repos.
2. **Enumerate packages.** List every subdirectory of `packages/` that contains a `default.nix`. Each one is an update unit.
3. **Inspect each package.** Read its `default.nix` to confirm a `version` attribute is present and that it uses a real fetcher (`fetchFromGitHub`, `fetchurl`, `fetchPypi`, `buildGoModule`, `buildRustPackage`, etc.). If a package has no `version` or uses a flake-style source, skip it and report why.
4. **Run `nix-update` per package.**
   - Default: `nix-update <package-name>` from the repo root (resolves to `packages/<name>/default.nix`).
   - If the user supplied a target version: `nix-update <package-name> --version=<ver>`.
   - If `nix-update` is not on `PATH`, run via `nix shell nixpkgs#nix-update -c nix-update ...`.
5. **Verify.** After each update, run `nix flake check` (or `just check` if defined) to validate the new hashes before declaring success.
6. **Report.** Print a table at the end:

   | Package | Old version | New version | Status |
   |---------|-------------|-------------|--------|

## Rules

- Only operate on the local `packages/` directory. Never run `nix flake update` — flake inputs are handled by `just update` and are out of scope for this skill.
- Do not introduce new dependencies. The only external tools used are `nix-update` and `nix flake check`.
- Do not commit anything. Stage the changes and report; let the user decide when and how to commit.
- If a package has no `version` attribute, or its update cannot be automated (missing `passthru.updateScript`, pinned commit instead of tag, etc.), stop and ask the user how to proceed — do not guess a version or edit hashes by hand.
- Surface major-version bumps separately so the user can review breaking changes before they ship.
- Preserve the package's existing structure (fetchers, buildInputs, meta block). `nix-update` only changes version + hashes; if it rewrites anything else, revert that diff.
