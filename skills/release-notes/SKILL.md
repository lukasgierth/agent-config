---
name: release-notes
description: Use when asked to write, generate, or draft release notes or a changelog entry. Given a git tag range or commit list, produces human-readable release notes grouped by type.
---

# Release Notes

## Goal

Produce human-readable release notes from git history between two refs.

## Workflow

1. Ask the user for the range if not provided (e.g. `v1.2.0..HEAD` or `v1.1.0..v1.2.0`).
2. Run `git log <range> --oneline --no-merges` to get the commit list.
3. Group commits by conventional commit type:
   - **Features** (`feat`)
   - **Bug Fixes** (`fix`)
   - **Performance** (`perf`)
   - **Other** (everything else worth mentioning; skip `chore`/`style`/`ci` unless significant)
4. Write a short summary paragraph (2-3 sentences) at the top describing the overall theme of the release.
5. Output the notes in this format:

   ```
   ## vX.Y.Z — YYYY-MM-DD

   <summary paragraph>

   ### Features
   - ...

   ### Bug Fixes
   - ...
   ```

6. Ask the user whether to append this to `CHANGELOG.md` or just display it.
