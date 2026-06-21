# KSP GameData workspace instructions

## Purpose and priorities

This repository is a working KSP 1 GameData installation, not a conventional source tree. Its goals are to:

1. Diagnose exceptions, incompatibilities, and gameplay-impacting behavior from `../KSP.log` and relevant configuration.
2. Implement narrow, reversible local fixes.
3. Preserve user-authored patches and intentional configuration edits across CKAN updates.

Favor correctness, save compatibility, and easy rollback over broad cleanup.

## Repository boundaries

- Treat CKAN-installed mod directories as vendored dependencies. Inspect them as needed, but do not edit them by default.
- Put new ModuleManager patches in `zzz_LocalPatches/Patches/`, grouped by affected mod or issue.
- Existing `zKerbalism*` directories contain locally maintained integration code and are tracked intentionally.
- `KerbalismModularScience` is tracked because it contains intentional local configuration edits. Keep changes there narrow; prefer an external patch when ModuleManager can express the fix.
- Never modify `Squad`, `SquadExpansion`, paid content, or third-party DLLs unless the user explicitly requests it and a patch/plugin cannot solve the issue.
- Saves live outside this repository. Treat them as user data: inspect only when relevant, never rewrite them without explicit permission, and recommend a backup before save-affecting work.
- Do not add generated ModuleManager files, logs, caches, crash dumps, screenshots, or editor backup files to Git.

## Efficient investigation

- Start with the active problem and inspect only relevant files. Use `rg`/`rg --files` with targeted paths and patterns; do not inventory or read all of GameData.
- For log work, search `../KSP.log` first for `Exception`, `Error`, `ERR`, `ReflectionTypeLoadException`, and the implicated assembly or part name. Read bounded context around matches.
- Distinguish root-cause exceptions from repeated downstream noise. Report the first meaningful occurrence, affected mod/assembly, and evidence.
- Use `ModuleManager.ConfigCache` only to inspect the final patched state. It is generated evidence, never a source file.
- Consult mod source or current authoritative documentation only when local files do not settle behavior or compatibility.
- For external web research, try Codex's built-in web tool once first. If it fails with the known Cloudflare `403` or another backend error, do not retry it in that response; fall back to the Exa MCP search/fetch tools to limit paid Exa usage. Prefer official documentation, repositories, release notes, and maintainer statements, and clearly distinguish sourced facts from inference.

## Change rules

- Make the smallest dedicated change that fixes the observed issue. Avoid unrelated formatting, renaming, modernization, or restructuring.
- Preserve user edits and unrelated dirty-worktree changes.
- Prefer ModuleManager selectors with explicit dependency guards such as `:NEEDS[...]` and an appropriate pass (`:AFTER[...]` when necessary). Avoid `:FINAL` unless no stable targeted pass works.
- Scope patches tightly by part/module/resource and use `:HAS[...]` guards where practical. Make patches idempotent so reapplication does not duplicate nodes or values.
- Add a short comment to non-obvious patches stating the symptom/workaround, not a guess presented as fact.
- Do not rename mod folders casually: paths and assembly-loading order can be compatibility-sensitive.
- Before editing an otherwise ignored third-party file, add only that exact intentional path to `.gitignore`'s allowlist so the edit is visible in Git. Do not unignore the whole dependency without a reason.

## Verification

There is no automated test suite. Verify in proportion to risk:

1. Review `git diff --check` and the focused diff.
2. Check ModuleManager syntax and confirm referenced part/module names against installed configs.
3. When runtime verification is needed, ask the user to launch KSP, reach the affected scene, and exit cleanly.
4. Recheck the new `../KSP.log` and `ModuleManager.ConfigCache` for the intended result and new errors.

Do not claim a runtime fix is verified solely because a config parses visually. Clearly separate static validation from in-game validation.

## Git and handoff

- Keep commits issue-focused. Do not commit CKAN upgrades together with local fixes.
- Do not run destructive Git commands or discard user changes.
- In the handoff, state what changed, what evidence supports it, what remains unverified in game, and any save-compatibility concern.
- Useful commit prefixes: `fix:`, `patch:`, `config:`, `docs:`, and `chore:`.
