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

Kerbalism builds via Visual Studio 2022. Open `C:\Users\pherl\src\Kerbalism\Kerbalism.sln`, set `KerbalismBuild` as start project, Debug configuration. Build output auto-copies to `GameData\Kerbalism\`. No command-line `msbuild` available in PATH yet.
