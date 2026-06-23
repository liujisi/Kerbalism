# Claude Code harness notes

## Environment

- **Platform:** Windows 11, VS Code extension
- **Shells:** PowerShell for all git operations. Git Bash only for `grep`/`sed`/`find` against file content. Never use WSL for git — slow startup and path translation issues.
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

**Two repos, similar remote names — verify which repo you're in before any git command:**

| Repo | Path | Purpose |
|------|------|---------|
| GameData | `d:\Games\steamapps\common\Kerbal Space Program\GameData` | Live KSP install — **minimal git changes here** |
| Kerbalism source | `C:\Users\pherl\src\Kerbalism` | Development, fixes, PRs |

**Step back:** If a git command produces unexpected output (empty results, missing remotes, wrong branches), stop and investigate. Do not retry with a different tool or shell — verify which repo you're in first with `pwd` + `git remote -v`.

**Kerbalism source remote naming:**

| Remote | URL | Purpose |
|--------|-----|---------|
| `origin` | `git@github.com:liujisi/Kerbalism.git` | Fork — push fix branches for PRs |
| `upstream` | `git@github.com:Kerbalism/Kerbalism.git` | Official — fetch latest, base branches |

**Branch workflow** (Kerbalism source repo):

```
upstream/master  ←── fix/<name>  ←── main (local only, combined DLL)
                    (push to origin for PR)
```

- `master` may contain older in-review fixes — don't disturb it.
- `main` is local-only, never pushed. Rebase fixes onto it for a combined gameplay DLL.
- Each fix gets its own branch off `upstream/master`, pushed to `origin` for PR.

`GIT_SSH_COMMAND` is set automatically via `.claude/settings.local.json` — no manual setup needed for git operations.

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

- **No Co-Authored-By** — do not add `Co-Authored-By` trailers to commit messages.