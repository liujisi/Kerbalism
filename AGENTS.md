# KSP GameData workspace instructions

## Purpose and priorities

This repository is a working KSP 1 GameData installation, not a conventional source tree. Its goals are to:

1. Diagnose exceptions, incompatibilities, and gameplay-impacting behavior from `../KSP.log` and relevant configuration.
2. Implement narrow, reversible local fixes.
3. Preserve user-authored patches and intentional configuration edits across CKAN updates.

Favor correctness, save compatibility, and easy rollback over broad cleanup.

## Discovering VS Code workspace roots

- The saved workspace is `D:\Documents\code-workspaces\KSP.code-workspace`. Open it to discover all active project roots.
- Currently there are two roots: `GameData` (this directory) and `C:\Users\pherl\src\Kerbalism` (the Kerbalism source).

## Kerbalism source checkout

- The VS Code workspace also has a top-level `Kerbalism` directory at `C:\Users\pherl\src\Kerbalism`. It contains the Kerbalism source for both the core mod and its official default configuration. The harness may use or modify this checkout when source-level investigation or development is appropriate.
- Do not confuse that repository with `GameData/Kerbalism`, which is the released, CKAN-managed copy installed in the live game. Treat `GameData/Kerbalism` as a vendored dependency: inspect it for installed-version evidence, but do not edit it by default.
- For Kerbalism behavior, native code, or examples of how the official default config handles an integration, consult the top-level `Kerbalism` repository. Use the live installation and `ModuleManager.ConfigCache` to verify what the game actually loaded.

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
- For log work: `../KSP.log` is typically 20–40 MB and far too large to load directly into context. **Always filter it externally first.** Use `grep`, `sed`, `rg`, or PowerShell `Select-String` to extract only matching lines + bounded context before reading. Common anchors:
  - `[EXC ` / `[ERR ` / `Exception` / `NullReferenceException` / `ArgumentException`
  - The implicated assembly, part name, or module name (e.g. `KerbalismScansat`, `SCANsat`)
  - Once the first meaningful occurrence is located, extract ±10 lines of context with `sed -n` and read only that slice.
- Distinguish root-cause exceptions from repeated downstream noise. Report the first meaningful occurrence, affected mod/assembly, and evidence.
- Use `ModuleManager.ConfigCache` only to inspect the final patched state. It is generated evidence, never a source file.
- Consult mod source or current authoritative documentation only when local files do not settle behavior or compatibility.
- For external web research, try Codex's built-in web tool once first. If it fails with the known Cloudflare `403` or another backend error, do not retry it in that response; fall back to the Exa MCP search/fetch tools to limit paid Exa usage. Prefer official documentation, repositories, release notes, and maintainer statements, and clearly distinguish sourced facts from inference.

## Diagnostic discipline

- **Pause before patching.** When you see a broken reference, a missing symbol, or a malformed config line, resist the reflex to fix it in place. First ask: did this ever work? If upstream is actively maintained and this is their code, a "broken" file is more likely a local issue (copy corruption, wrong version, missing build config) than an upstream bug.
- **Check upstream before editing source files.** For any file that appears wrong, compare against the authoritative upstream (GitHub raw, the repo's own `origin/main`) before concluding it needs a local edit. A `grep` mismatch between local and remote can turn a "fix" into a regression.
- **Verify versions, not just names.** When resolving a missing assembly or dependency, check the version number the project actually binds against (`.csproj` `HintPath`, `packages.config`, assembly references). Copying `0Harmony.dll` 2.2.1 when the project binds 2.0.4 produces a different set of errors — the filename matching is necessary but not sufficient.
- **Root-cause the error chain, not the first symptom.** If a build fails with "type not found," trace backward: is the reference path resolving? is the HintPath file present? is the imported `.targets`/`.xml` file loading the properties the reference depends on? Fix the earliest broken link.
- **Local corruption has telltale signs.** `$(Property)\` turning into `\\` across a file, Unix mode bits (`100755` → `100644`) on every binary after a WSL→Windows copy — these patterns point to filesystem translation artifacts, not code bugs. Revert and re-copy cleanly rather than hand-editing dozens of files.
- **When you have a human in the loop, each round-trip is expensive.** A build takes the user a minute; a bad guess costs them that minute with nothing learned. Do the comparison, version check, or upstream verification *before* proposing a fix. If you can't verify (e.g. you lack `msbuild`), say so and ask the user to run a diagnostic command rather than guessing at a patch.

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
