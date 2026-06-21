# Local KSP GameData fixes

This Git repository tracks local KSP compatibility patches and intentional configuration edits while treating CKAN-installed mods and paid visual content as external dependencies.

## Layout

- `zzz_LocalPatches/` — preferred home for new ModuleManager-only fixes.
- `zKerbalism*` — existing locally maintained Kerbalism integrations and plugins.
- `KerbalismModularScience/` — tracked third-party mod with intentional local edits.
- `docs/issue-template.md` — compact investigation record for reproducible fixes.

The actual installed mod set remains managed by CKAN. Export the CKAN mod list separately after meaningful install changes; do not add paid content to Git.

## Basic workflow

1. Reproduce the problem and exit KSP cleanly.
2. Record the symptom and search `../KSP.log` for the first relevant exception or error.
3. Add the narrowest viable patch under `zzz_LocalPatches/Patches/`.
4. Review the diff, relaunch KSP, and confirm both behavior and the resulting ModuleManager cache/log.
5. Commit only the focused local fix.

