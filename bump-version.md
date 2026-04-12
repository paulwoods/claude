---
name: bump-version
description: Bumps the patch version of the current project and creates a matching git tag. Detects version strings in common manifest files (package.json, pom.xml, Cargo.toml, pyproject.toml, etc.), increments the patch component, updates all occurrences, and tags the commit. Use when the user asks to bump, increment, or release a new patch version of a repository.
---

# Bump Version

## Preconditions

- Run `git status --porcelain` first. If the working tree is dirty, ask the user whether to proceed, stash, or abort.
- Confirm you are in a git repository (`git rev-parse --git-dir`).

## Workflow

1. **Detect current version.** Search for version declarations in (in priority order):
   - `package.json` (`"version"` field; also check workspaces)
   - `pom.xml` (`<version>` under `<project>`; check child modules)
   - `Cargo.toml` (`[package] version`)
   - `pyproject.toml` (`[project] version` or `[tool.poetry] version`)
   - `build.gradle` / `build.gradle.kts` (`version = ...`)
   - `VERSION` or `version.txt` files

   If multiple files are found with matching versions, proceed. If they disagree, show the user the conflict and ask which to trust.

2. **Compute new version.** Parse as semver (`MAJOR.MINOR.PATCH[-PRERELEASE]`).
   - Strip any pre-release suffix (e.g. `-SNAPSHOT`, `-rc1`) before incrementing unless the user asks otherwise.
   - Increment PATCH by 1. Confirm the new version with the user before writing.

3. **Update files.** Replace the version string in every file identified in step 1. For `package.json`, prefer `npm version <new> --no-git-tag-version` so the lockfile updates too. For Maven multi-module projects, prefer `mvn versions:set -DnewVersion=<new> -DprocessAllModules`.

4. **Commit and tag.**
   - `git add` the modified files.
   - `git commit -m "chore: bump version to <new>"`
   - `git tag -a v<new> -m "Release <new>"` (confirm the `v` prefix convention matches existing tags with `git tag --list`).

5. **Report.** Tell the user the new version, the tag name, and remind them to `git push && git push --tags` when ready. Do not push automatically.

