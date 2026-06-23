# Claude Code harness notes

This file contains harness-specific tool and environment guidance for the Claude Code agent only. Consult `AGENTS.md` for the authoritative project-level instructions, repository boundaries, investigation strategy, and change rules.

## Environment

- **Platform:** Windows 11, VS Code extension
- **Shells:** PowerShell for git operations. Git Bash only for `grep`/`sed`/`find` against file content.
- **Node.js:** v24.17.0 at `C:\Program Files\nodejs\`
- The Claude Code harness may set `GIT_SSH_COMMAND` or other per-process environment variables for git and ssh; do not expect manual password prompts.

## MCP servers

### textEditor

Provides `str_replace`, `view`, `insert`, `create`, `undo_edit` tools through stdio transport.

**Use `str_replace` for tab-indented files.** The native `Edit` tool can silently normalize tabs into spaces, causing byte-level `old_string` mismatches on tab-indented source. The MCP tool preserves tab characters when editing these files.

When `Edit` fails on a tab-indented file with a whitespace error, fall back to `textEditor.str_replace`.

## Notes

- This file is not the workspace policy. Use `AGENTS.md` for project-specific rules and investigation guidance.
- Keep `CLAUDE.md` focused on Claude Code harness behavior only.

## Claude Code critical peer rules

To expand from the critical peer rules in AGENTS.md. You must also follow:

## 1. The Pre-Mortem Checklist
Before proposing any changes, explicitly answer these three questions:
1. Am I fixing the root cause, or just suppressing a symptom/warning?
2. Are there any unmarked assumptions in my plan?
3. Is there a simpler architectural alternative to what the user is asking?

## 2. Certainty Tags
When explaining a bug or proposing a fix, label your premises using these tags:
- **[C] Certain:** Verified by reading the file, test output, or explicit documentation.
- **[I] Inferred:** Educated guess based on naming conventions or standard framework behavior.
- **[S] Speculative:** You do not have enough context.

*(If a core part of your logic relies on an `[S]`, ask to read more files or run a command before proposing the change.)*

## 3. The "Way Out" Clause (Permission to Fail)
When tests, build, or runtime are failing and you cannot confidently identify the root cause after reviewing the logs:
1. **STOP.** Do not implement a workaround, do not cast types to `Any`, and do not silence warnings.
2. Output the exact phrase: `ROOT CAUSE UNKNOWN`.
3. List your 3 best hypotheses for where we should investigate next, and tell me what shell commands or debug logs you need me to run to get more data.
