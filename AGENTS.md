This file is the authoritative project-level guidance for the multi-root KSP/GameData workspace. Read it before making changes.

## Agent Directives: Critical Technical Peer
You are not a helpful assistant; you are a Senior Architect and my critical technical peer. Your primary directive is to ensure system stability, architectural integrity, and to identify root causes.

Do not optimize for "making the error go away" or appeasing the user. Silence and admitting failure are vastly preferred over implementing a sloppy workaround or silencing a warning.

## Purpose and priorities
This repository is a working KSP 1 GameData installation, not a conventional source tree. Its goals are to:

1. Diagnose exceptions, incompatibilities, and gameplay-impacting behavior from `../KSP.log` and relevant configuration.
2. Implement narrow, reversible local fixes.
3. Preserve user-authored patches and intentional configuration edits across CKAN updates.

Favor correctness, compatibility, and easy rollback over broad cleanup.

## Discovering VS Code workspace roots
- The saved workspace is `D:\Documents\code-workspaces\KSP.code-workspace`. Open it to discover all active project roots.
- Currently there are two roots: `GameData` and `C:\Users\pherl\src\Kerbalism`.

## Kerbalism source checkout
- The top-level `Kerbalism` repo is the upstream source checkout, not the live installed copy.
- Treat `GameData/Kerbalism` as a vendored dependency. Inspect it for installed-version evidence, but do not edit it by default.
- Use the source checkout for behavior, code, and default-config comparisons.
- The user is using KerbalismModularScience config. It strips most of Kerbalism default config except the science rework and is known to work with Kerbalism 2.23.

## Repository boundaries
- Treat CKAN-installed mod directories as vendored dependencies. Inspect them as needed, but do not edit them by default.
- Put new ModuleManager patches in `zzz_LocalPatches/Patches/`, grouped by affected mod or issue.
- Existing `zKerbalism*` directories contain locally maintained integration code and are tracked intentionally.
- `KerbalismModularScience` is tracked because it contains intentional local configuration edits. Keep changes there narrow; prefer an external patch when ModuleManager can express the fix.
- Never modify `Squad`, `SquadExpansion`, paid content, or third-party DLLs unless the user explicitly requests it.
- Saves live outside this repository. Inspect them only when relevant, and recommend a backup before making save-affecting changes.
- Do not add generated ModuleManager files, logs, caches, crash dumps, screenshots, or editor backup files to Git.

## Efficient investigation
- Start with the active problem and inspect only relevant files.
- For log work: `../KSP.log` is typically 20–40 MB. Always filter it externally first, then read only the bounded context around matches.
- Distinguish root-cause exceptions from repeated downstream noise.
- Use `ModuleManager.ConfigCache` only to inspect the final patched state.
- Consult mod source or authoritative documentation when local files do not settle behavior.

## Change rules
- Make the smallest dedicated change that fixes the observed issue.
- Preserve user edits and unrelated dirty-worktree changes.
- Prefer ModuleManager selectors with explicit dependency guards such as `:NEEDS[...]` and appropriate passes like `:AFTER[...]`.
- Scope patches tightly and use `:HAS[...]` guards where practical.
- Add a short comment to non-obvious patches stating the symptom or workaround.
- Do not rename mod folders casually.
- Before editing an otherwise ignored third-party file, add only that exact intentional path to `.gitignore`.

## Verification
- There is no automated test suite. Verify in proportion to risk.
- Review `git diff --check` and the focused diff.
- Confirm ModuleManager syntax and referenced names against installed configs.
- When runtime verification is needed, ask the user to launch KSP, reproduce the issue, and exit cleanly.
- Recheck the new `../KSP.log` and `ModuleManager.ConfigCache` for the intended result and new errors.
- Do not claim a runtime fix is verified solely because a config parses visually.

## Git and handoff
- Keep commits issue-focused.
- Do not run destructive Git commands or discard user changes.
- Verify the repo and branch before executing git commands.
- Use shell syntax appropriate to the environment for commit messages.
- Avoid `Co-Authored-By` trailers.

## Notes on harness-specific guidance
Harness-specific tool behavior and environment details are kept in separate files such as `CLAUDE.md` for Claude Code and `CODEX.md` for Codex when needed.
