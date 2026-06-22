# Claude Code harness notes

## Environment

- **Platform:** Windows 11, VS Code extension
- **Shells:** PowerShell (primary), Git Bash (POSIX, for `grep`/`sed`/`find`)
- **Node.js:** v24.17.0 at `C:\Program Files\nodejs\`

## Workspace

Two repos, per `D:\Documents\code-workspaces\KSP.code-workspace`:

| Root | Path | Purpose |
|------|------|---------|
| GameData | `d:\Games\steamapps\common\Kerbal Space Program\GameData` | Live KSP install, CKAN-managed mods, local patches |
| Kerbalism source | `C:\Users\pherl\src\Kerbalism` | Upstream Kerbalism repo for source investigation and code fixes |

## MCP servers

### textEditor

Provides `str_replace`, `view`, `insert`, `create`, `undo_edit` tools through stdio transport. Configured globally in `~/.claude.json`.

**Use `str_replace` for tab-indented files.** The native `Edit` tool can fail on tab-indented source because the API boundary may normalize tabs to spaces, causing byte-level `old_string` mismatch. The MCP tool receives strings through JSON-RPC over stdio, where tab characters survive as `\t` in the JSON payload.

When `Edit` fails on a tab-indented file with a whitespace error, fall back to `textEditor.str_replace`.

## KSP.log

`../KSP.log` is typically 20-40 MB. Never Read it directly — filter externally first with `grep`/`sed`/`Select-String`, then read only the bounded context around matches. Key search anchors: `[EXC `, `[ERR `, `Exception`, `NullReferenceException`, the implicated assembly or module name.

## Build

```powershell
& "C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe" "C:\Users\pherl\src\Kerbalism\Kerbalism.sln" -v:m
```

- MSBuild 17.14 from VS 2022 Community (not in PATH, use full path above)
- `KerbalismBuild` is the start project — it orchestrates KerbalismBootstrap → Kerbalism → deploy
- Debug configuration only (release requires archive passwords)
- Output goes to `BuildSystem\BinariesDebug\` then auto-copies `*.dll` + `*.pdb` to `GameData\Kerbalism\`
- `UserConfigDevEnv.xml` at `BuildSystem\` defines KSP path and version constants

## Git

- **Upstream:** `git@github.com:Kerbalism/Kerbalism.git` (origin)
- **Fork:** `git@github.com:liujisi/Kerbalism.git` (liujisi) — push PRs here
- **Push from WSL** — SSH keys with passphrase live in WSL; the agent there has them unlocked. Windows Git Bash SSH can use the key too but needs `ssh-add` + passcode.

```bash
# In WSL
cd /mnt/c/Users/pherl/src/Kerbalism
git push liujisi master
```

### Git commit messages

**Use the syntax that matches the shell.** `@'...'@` is PowerShell here-string syntax — it only works in PowerShell. In Bash (Git Bash or WSL), use `-m` per paragraph:

```bash
# Bash — correct
git commit -m "subject" -m "body paragraph 1" -m "body paragraph 2"

# Also fine in Bash
git commit -m "subject

body paragraph 1

body paragraph 2"
```

```powershell
# PowerShell — correct
git commit -m @'
subject

body paragraph 1

body paragraph 2
'@
```

**Never use `@'...'@` in Bash** — Bash treats the `@` marks as literal text, and they end up in the commit message.
