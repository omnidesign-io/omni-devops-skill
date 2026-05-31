# AI Agent Workflow: Skill Updates & Releases

This file instructs AI agents on how to pull updates from the master skill file and release new versions of the `omni-devops` skill. Follow these instructions whenever a skill update is requested.

## Master Skill Location

The master installation of this skill currently resides at:
`/Users/hanny/.codex/skills/omni-devops/SKILL.md`

> **Note:** The location of the master skill file might change in the future. Always prioritize the path explicitly provided by the user in their request. If no path is provided, default to the master path above.

---

## Release Workflow

When the master skill file is updated, follow this step-by-step process to pull changes, compare diffs, and cut a new release:

### 1. Retrieve & Merge Changes
- Read the master skill file from its source path.
- Compare it with the workspace copy: `SKILL.md`.
- Copy the master skill contents into `SKILL.md`.
- **CRITICAL:** The master skill might not have a version indicator in its frontmatter. You must preserve the frontmatter structure of this repository's `SKILL.md` by explicitly maintaining or adding the `metadata.version` field. 
- Determine the next version number. A typical minor update (like search indexing controls) bumps the second digit (e.g., `1.0.0` -> `1.1.0`). If in doubt, ask the user.
- Update `metadata.version` in the `SKILL.md` frontmatter to the new version (e.g., `"1.2.0"`).

### 2. Update Version References
- **VERSION file:** Update `/VERSION` to contain only the new version string followed by a newline (e.g., `1.2.0\n`).
- **README file:** Update `/README.md` to reference the new version:
  - Bump the `Version:` field at the top of the file.
  - Add a summary of the release changes under the `## Release` section.

### 3. Document in Changelog
- **CHANGELOG file:** Append or update `/CHANGELOG.md`. 
- Follow standard Keep a Changelog conventions.
- Document all notable additions, changes, or deprecations introduced by the new skill content, prioritizing the specific features highlighted by the user (e.g., "noindex related changes").

### 4. Verification
- Verify the integrity of the workspace.
- Run `git diff --check` to check for trailing whitespaces or unresolved formatting issues. Do not commit if this command fails.

### 5. Git Commit & Release Tagging
- Add all modified and created files:
  ```bash
  git add README.md SKILL.md VERSION CHANGELOG.md
  ```
- Commit the changes using a structured release message:
  ```bash
  git commit -m "release: <version> - <summary of changes>"
  ```
- Create an annotated git tag for the release to finalize the process:
  ```bash
  git tag -a v<version> -m "release v<version>: <summary of changes>"
  ```
