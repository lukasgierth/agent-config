---
name: nixpkgs-package
description: Use when adding a new package to Nixpkgs, updating or fixing a Nixpkgs package you maintain, or registering yourself as a Nixpkgs maintainer. Triggers on "add package to nixpkgs", "package this for nixpkgs", "pkgs/by-name", "bump my nixpkgs package", "nix-update", "add me as maintainer", "maintainer-list.nix", "nixpkgs-review", "nixpkgs-vet".
---

# Adding or Maintaining a Nixpkgs Package

Nixpkgs is an upstream repository, not a personal flake: every change goes through a
PR, review by a committer, and a documented set of conventions. Follow the official
documents below rather than this skill where they disagree — this skill only orders the
work and records the traps.

## Sources of truth

Local copies exist in any nixpkgs checkout; GitHub links are given for citing in PRs.

| Topic | Local path | GitHub |
|---|---|---|
| PR process, commit conventions, review, backports | `CONTRIBUTING.md` | [#how-to-create-pull-requests](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md#how-to-create-pull-requests), [#commit-conventions](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md#commit-conventions), [#how-to-merge-pull-requests-yourself](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md#how-to-merge-pull-requests-yourself) |
| AI/automation disclosure | `CONTRIBUTING.md` | [#automationai-policy](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md#automationai-policy), [#transparency](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md#transparency) |
| Adding a package, naming, versioning, meta, sources, tests, updaters | `pkgs/README.md` | [#quick-start-to-adding-a-package](https://github.com/NixOS/nixpkgs/blob/master/pkgs/README.md#quick-start-to-adding-a-package), [#package-naming](https://github.com/NixOS/nixpkgs/blob/master/pkgs/README.md#package-naming), [#versioning](https://github.com/NixOS/nixpkgs/blob/master/pkgs/README.md#versioning), [#meta-attributes](https://github.com/NixOS/nixpkgs/blob/master/pkgs/README.md#meta-attributes), [#automatic-package-updates](https://github.com/NixOS/nixpkgs/blob/master/pkgs/README.md#automatic-package-updates) |
| `pkgs/by-name/` rules | `pkgs/by-name/README.md` | [file](https://github.com/NixOS/nixpkgs/blob/master/pkgs/by-name/README.md) |
| Maintainer list, becoming a maintainer | `maintainers/README.md` | [#how-to-become-a-maintainer](https://github.com/NixOS/nixpkgs/blob/master/maintainers/README.md#how-to-become-a-maintainer), [#individual-maintainer-list](https://github.com/NixOS/nixpkgs/blob/master/maintainers/README.md#individual-maintainer-list) |
| Language frameworks (`buildRustPackage`, `buildGoModule`, `python`, `npm`, …) | `doc/languages-frameworks/<lang>.section.md` | [manual: language support](https://nixos.org/manual/nixpkgs/unstable/#chap-language-support) |
| CI, merge bot | `ci/README.md` | [file](https://github.com/NixOS/nixpkgs/blob/master/ci/README.md) |
| `nixpkgs-vet` (by-name + format rules) | `ci/nixpkgs-vet.sh` | [repo & wiki](https://github.com/NixOS/nixpkgs-vet/wiki) |
| `nixpkgs-review` | — | [usage](https://github.com/Mic92/nixpkgs-review#usage) |
| ofborg CI | — | [NixOS/ofborg](https://github.com/NixOS/ofborg) |

## Part 1 — Adding a new package

### 0. Pre-flight (the part reviewers enforce socially)

From `pkgs/README.md#quick-start-to-adding-a-package`, answer **no** to any of these and
the package should not be added: is it ready for general use; does it have a clear
license; will anyone besides you use it; is it actively maintained upstream; are you
willing to maintain it for at least one release cycle? Libraries need at least one
dependent, ideally packaged in the same PR. Do not add a maintainer entry for someone
else without their consent.

### 1. Branch, path, attribute name

```sh
git switch -c add-<pkg> origin/master     # never work on master
mkdir -p pkgs/by-name/<2-letter-prefix>/<pkg>
```

- Directory name **is** the attribute name: `pkgs.hello` ← `pkgs/by-name/he/hello/package.nix`.
  Prefix is the lowercased first two characters.
- Do **not** also add an entry to `pkgs/top-level/all-packages.nix`; by-name packages are
  auto-included. Only declare there when you must change implicit argument defaults
  (`pkgs/by-name/README.md#changing-implicit-attribute-defaults`).
- By-name is impossible when the attribute name is not a valid directory name (e.g.
  contains dots or needs a category). Then use the category hierarchy + a `callPackage`
  entry in `all-packages.nix`.
- Naming: upstream name, lowercased, hyphens not underscores (`pkgs/README.md#package-naming`).
  If the natural name is ambiguous or clashes, prefer the upstream crate/package name
  (`purple-ssh` rather than `purple`) and say so in the PR body.

### 2. Write `package.nix`

- A function taking dependencies as arguments; return one derivation. `pname` should equal
  the directory name, `version` from the upstream release.
- Source: `fetchFromGitHub { owner; repo; rev = "v${finalAttrs.version}"; hash; }`, or the
  ecosystem fetcher. Never a binary blob when source builds are possible.
- **`__structuredAttrs = true;` is mandatory for new packages** — `nixpkgs-vet` rejects new
  packages without it (NPV-166).
- Language-specific: `rustPlatform.buildRustPackage` (+ `cargoHash`), `buildGoModule`
  (+ `vendorHash`), `python3Packages`, `stdenv.mkDerivation`, `npmHooks.npmConfigHook`
  (+ `fetchNpmDeps`). Read the matching `doc/languages-frameworks/*.section.md` first —
  they are short and current; training data is not.
- `meta`: `description`, `homepage`, `license` (must be a `lib.licenses.*` value),
  `mainProgram` for CLI tools, `platforms`, `maintainers`, and `changelog` when upstream
  has one. `platforms` is a claim about support — do not list platforms you cannot reason
  about.
- `passthru.updateScript = nix-update-script { };` — required in practice so the package is
  bump-able by you, r-ryantm and reviewers. See
  `pkgs/README.md#nix-update-script`.

### 3. Hashes: the fake-hash loop

```sh
# 1. put lib.fakeHash in for src and every derived hash
# 2. build once; the error prints the real hash
nix-build -A <pkg> 2>&1 | grep -A2 'hash mismatch'
# 3. paste the reported hash, repeat until the build survives fetch
```

Same loop per hash kind: `hash` (src), `cargoHash`, `vendorHash`, `npmDepsHash`.
`nix-prefetch-url --unpack <url>` + `nix hash convert --to sri` also works for plain
tarballs. Never hand-edit a hash to make a build pass.

### 4. Format and verify locally

```sh
nix fmt                                    # nixfmt-rfc-style
nix-build -A <pkg>
./result/bin/<prog> --version
./result/bin/<prog> --help                 # and any subcommand that proves it runs
nix-build lib/tests/maintainers.nix        # if you touched the maintainer list
```

Run every installed binary. If the tool needs runtime companions (`ssh`, `ffmpeg`,
`imagemagick`), wrap them with `wrapProgram $out/bin/<prog> --prefix PATH : ${lib.makeBinPath [...]}`
and say in the PR why each one is a hard dependency, not an optional one. Set
`doCheck = false` only with a comment naming what the suite needs (`$HOME`, network, a
writable `/tmp`, a real TTY).

### 5. `nixpkgs-vet` — the by-name gate

```sh
./ci/nixpkgs-vet.sh master
```

It compares a **git worktree of HEAD** against master, so commit first; uncommitted
changes are silently ignored ("Dirty tree" warning). It enforces by-name placement,
`__structuredAttrs`, maintainer validity, and flags discouraged patterns.

### 6. Commits

Nixpkgs commit format (`pkgs/README.md#commit-conventions`):

```
<pkg>: init at <version>      # new package
<pkg>: <old> -> <new>         # update
<pkg>: fix build / refactor / …
maintainers: add <handle>     # always its own commit
```

One package change per commit; do not mix a new package with unrelated cleanups. Add a
short body when the change needs context (changelog link, why a patch is needed).

If AI tooling produced part of the change, every assisted commit needs a trailer:

```
Assisted-by: <tool> <model name and version>
```

`Co-authored-by:` does **not** satisfy the policy. You also must disclose AI use
separately in the PR summary, and you must have reviewed and understood the change
(`CONTRIBUTING.md#automationai-policy`).

### 7. PR

- Fork → push your branch → PR against `NixOS/nixpkgs` `master` (not a release branch).
- Rebase onto current `origin/master` before opening; `git fetch` first, nixpkgs moves fast.
- Fill `.github/PULL_REQUEST_TEMPLATE.md` honestly: the platform boxes you actually built
  on, the binary smoke test, and the `nixpkgs-review` box only after running it.
- Post the smoke-test transcript. It saves the reviewer a round trip.
- Do not self-merge a first package; a committer has to merge it. Afterwards, maintainers of
  by-name packages can merge their own PRs with `@NixOS/nixpkgs-merge-bot merge`
  (`CONTRIBUTING.md#how-to-merge-pull-requests-yourself`).

### 8. `nixpkgs-review` after the PR exists

```sh
nix run nixpkgs#nixpkgs-review -- pr <number> --post-result
```

Purpose: answer "which attributes does this PR change, and do they still build — on *my*
machine?". It diffs the PR against its merge-base, maps touched paths to attributes
(by-name dirs, package files, aliases, modules → NixOS tests), merges the PR into the base
in a throwaway worktree, evaluates just those attributes, builds them sandboxed, and emits
a markdown report (built/failed per attribute, log snippets, closure-size delta for
updates). Optionally posts it as a PR comment and drops you into a `nix-shell` with
`./result` ready.

```sh
... pr <n>                 # review a PR (also accepts ranges and several numbers)
... pr <n> --no-shell      # build + report only, no interactive shell
... pr <n> -c commit       # build the branch as pushed, not the merge result
... rev <commit|branch>    # review a local change without a PR
... wip                    # review the uncommitted working tree
```

Flags worth knowing: `--systems`, `--build-args`, `--post-result`, `--token` (avoids
reading `gh`'s config), `--eval local` (evaluate locally instead of trusting GitHub's
eval job). It needs GitHub auth — `gh auth login` or `GH_TOKEN`.

It is evidence, not a CI gate: not required to merge, and not a substitute for running the
binary. Use it on update PRs as well, not just on new packages.

## Part 2 — Adding yourself as a maintainer

Needed the first time a package references you in `meta.maintainers`; a package PR that
references a non-existent handle fails evaluation.

1. Append to the keep-sorted block in `maintainers/maintainer-list.nix` (markers at the
   top and bottom of the list: `# keep-sorted start case=no numeric=no block=yes`):

   ```nix
   yourhandle = {
     name = "Your Name";
     email = "you@example.org";     # optional but expected
     github = "yourhandle";         # required in practice
     githubId = 1234567;            # required whenever github is set
     matrix = "@you:matrix.org";    # optional
   };
   ```

   `githubId` comes from `https://api.github.com/users/<handle>` (`.id`), never guessed.
   The handle is conventionally equal to the GitHub username.
2. Reference it: `meta.maintainers = [ lib.maintainers.yourhandle ];`.
3. Separate commit: `maintainers: add yourhandle`.
4. Validate: `nix-instantiate --eval -E 'with import ./. {}; lib.maintainers.yourhandle'`
   and `nix-build lib/tests/maintainers.nix` (checks required fields and uniqueness of
   `github`, `githubId`, `email`).
5. After merge you get an email invitation to the `NixOS/nixpkgs-maintainers` GitHub team —
   **valid for seven days only**. Accepting gives review requests, triage rights and access
   to the merge bot. Expired? Request a new invite per
   `maintainers/README.md#how-to-become-a-maintainer`.

Adding a maintainer who is not you requires their consent; do not invent emails or IDs.

## Part 3 — Updating a package you maintain

```sh
nix-update <attr>                      # latest upstream version
nix-update --version=<x.y.z> <attr>    # specific version
nix-update --flake <attr>              # inside a flake repo
```

`nix-update` handles `version` and the fetcher hash (and `vendorHash` for some ecosystems).
It does **not** recompute every derived hash: `cargoHash`, `npmDepsHash` and similar need
the fake-hash loop from step 3. Verify by building, not by trusting the diff.

Then:

- Commit `<pkg>: <old> -> <new>`, with a changelog link in the body for anything notable.
- Major or breaking update → add a Nixpkgs release-note entry
  (`doc/release-notes/`, see `CONTRIBUTING.md#release-notes`).
- Backport when the old version is still in a release branch:
  `git switch -c backport-<pkg>-<release> release-<YY.MM>` +
  `git cherry-pick -x <commit>` + PR targeting that branch, linking the original PR.
- Security updates: mention the CVE/advisory in the PR body; do not wait for a review round
  to disclose it.
- r-ryantm/ofborg may open bump PRs for your package. Review them like any contributor's
  change — the update script does not test the result.

## Traps worth remembering

- `nixpkgs-vet` ignores uncommitted changes; commit before running it.
- Version must match the upstream tag, not a vendor string; keep `version` literal in
  `package.nix` so `nix-update` can bump it (`cargo`-derived versions break the updater).
- Do not add the package to `all-packages.nix` unless you need custom arguments.
- Comments in the derivation are for *why* (a disabled test suite, a runtime wrapper), and
  reviewers will ask for them.
- Keep agent artifacts (`PLAN.md`, `CIR-LOG.md`, notes) out of the PR: they are untracked
  local files, and `git add -A` will leak them.
- Never force-push a branch someone else is reviewing without a note in the PR.

## Checklist

```
[ ] pre-flight questions answered yes; license clear; willing to maintain
[ ] pkgs/by-name/<xx>/<pkg>/package.nix, __structuredAttrs, updateScript, meta
[ ] hashes real (no fakeHash left), nix fmt clean
[ ] nix-build -A <pkg> + every installed binary runs
[ ] maintainers: add <handle> in a separate commit (first time only)
[ ] nix-build lib/tests/maintainers.nix, ./ci/nixpkgs-vet.sh master
[ ] branch rebased on origin/master; pushed to fork
[ ] PR against master, template filled, smoke test pasted, AI use disclosed +
    Assisted-by trailers
[ ] nixpkgs-review pr <n> --post-result
[ ] after merge: accept the maintainers team invite within 7 days
```
