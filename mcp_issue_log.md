# MCP Issue Log

## Purpose

This file records MCP-related incidents, errors, causes, fixes, and historical troubleshooting notes. It is intentionally separated from `mcp_usage_rules.md` so the rules file remains concise and cheap to read frequently.

Future assistants should read this file only when troubleshooting, investigating repeated errors, or appending a new incident record.

## Issue Record Template

```markdown
## Issue: Short Title

### Date

YYYY-MM-DD

### MCP Service

- Service:
- Tool:

### Symptoms

- 

### Cause Analysis

- 

### Fix

- 

### Follow-Up Rules

- 
```

## Issue: Repeated `filesystem.ls` Calls Without Progress

### Date

2026-06-23

### MCP Service

- Service: CherryFilesystem
- Tool: ls

### Symptoms

- The assistant repeatedly called `filesystem.ls` to inspect a directory.
- After confirming that the directory existed or was empty, the assistant continued probing.
- The assistant did not promptly perform the actual deliverable action, such as `write`, `edit`, running a command, or generating the requested file.

### Cause Analysis

- The final deliverable was not clearly identified before tool use.
- Environment probing was treated as if it were task progress.
- There was no self-check after each tool call to confirm whether it advanced the final goal.
- No limit was set for repeated checks of the same directory.

### Fix

- Define the final deliverable before calling MCP tools.
- Limit `filesystem.ls` on the same directory to one call unless the path changes, permissions fail, or the user explicitly asks for a recheck.
- Once path or file state is known, continue to the actual deliverable action.
- Report tool failures clearly instead of repeating probes.

### Follow-Up Rules

- Repeated checks that do not advance the goal are forbidden.
- `ls`, `read`, `grep`, and `glob` are context-gathering tools only; they must not replace execution.

## Issue: PowerShell Hashtable Array Commas Caused Parse Error

### Date

2026-06-23

### MCP Service

- Service: CherryFilesystem
- Tool: read, edit

### Symptoms

- Running `mcp-oneclick.ps1` produced `表达式或语句中包含意外的标记“)”`.
- The error pointed near the closing parenthesis of `$McpServers = @(... )`.

### Cause Analysis

- Multiple hashtables inside a PowerShell array were written in a comma-separated style such as `@{ ... }, @{ ... }, @{ ... }`, which caused parsing trouble in the generated script.
- The script was delivered without a PowerShell syntax parse verification.

### Fix

- Changed the array to use multi-line hashtable elements without comma separators.
- Split `Start-Process` standard output and standard error redirection into separate `.out.log` and `.err.log` files.

### Follow-Up Rules

- For generated PowerShell scripts, prefer comma-free multi-line hashtable elements in complex arrays.
- Do not point `RedirectStandardOutput` and `RedirectStandardError` to the same file.
- After script creation or edits, verify key sections and run syntax parsing when feasible.

## Issue: Split Rules And Logs To Reduce Token Waste

### Date

2026-06-23

### MCP Service

- Service: CherryFilesystem
- Tool: read, write, delete

### Symptoms

- A single MCP guidance file mixed stable usage rules with historical problem logs.
- Future assistants would need to read the entire file frequently, including verbose incident history, wasting context tokens.

### Cause Analysis

- Rules and incident records had different reading frequencies but were stored together.
- The rules file lacked an explicit policy requiring logs to be stored separately.

### Fix

- Created `mcp_usage_rules.md` for stable rules only.
- Created `mcp_issue_log.md` for encountered problems, causes, fixes, and historical records.
- Added a rule requiring future assistants to keep rules and logs separated.

### Follow-Up Rules

- Read `mcp_usage_rules.md` before normal MCP operations.
- Read `mcp_issue_log.md` only for troubleshooting or when appending a new issue.
- Keep detailed incidents out of the rules file.

## Issue: Refactor Rules Into High-Frequency And Conditional Files

### Date

2026-06-23

### MCP Service

- Service: CherryFilesystem
- Tool: read, write

### Symptoms

- `mcp_usage_rules.md` still contained PowerShell-specific rules and tutorial-style filesystem flows.
- The high-frequency MCP rule file was longer than necessary for routine reads.

### Cause Analysis

- Stable MCP rules, conditional PowerShell rules, and explanatory workflow details had different reading frequencies but were still partly grouped together.
- The rules file needed stronger admission criteria for future additions.

### Fix

- Compressed `mcp_usage_rules.md` into a short high-frequency rule card.
- Created `powershell_script_rules.md` for `.ps1`-specific rules.
- Added a rule that `powershell_script_rules.md` should be read only when creating, reviewing, or editing PowerShell scripts.

### Follow-Up Rules

- Keep high-frequency rules short and task-agnostic.
- Move conditional domain-specific rules into separate files and reference them from the core rule card.
- Keep incident detail in `mcp_issue_log.md`, not in rule files.

## Issue: PowerShell Array Hashtable Separators In `mcp-oneclick.ps1`

### Date

2026-06-23

### MCP Service

- Service: CherryFilesystem
- Tool: read, edit

### Symptoms

- Running `mcp-oneclick.ps1` failed with `UnexpectedToken` at a closing `)`.
- PowerShell reported: `表达式或语句中包含意外的标记“)”`.

### Cause Analysis

- `$McpServers = @(...)` contained multiple adjacent hashtable literals without comma separators.
- The PowerShell script rule file also contained an incorrect rule that preferred no comma separators for complex array literals.

### Fix

- Added comma separators between adjacent hashtable elements in `$McpServers`.
- Corrected `powershell_script_rules.md` to require commas between adjacent hashtable elements in PowerShell arrays.

### Follow-Up Rules

- In PowerShell array literals, separate adjacent hashtable elements with commas.
- Treat parser errors reported at a closing delimiter as a possible missing separator earlier in the expression.

## Issue: Filesystem For File Changes And Shell For PowerShell Validation

### Date

2026-06-23

### MCP Service

- Service: CherryFilesystem
- Tool: ls, glob, read, edit
- Service: Shell MCP
- Tool: command execution

### Symptoms

- The user required basic file reads and writes to be completed through Filesystem MCP, not Shell MCP.
- Shell MCP was used only to attempt PowerShell validation for `script\mcp-oneclick.ps1`.
- `filesystem.glob` failed with `Package subpath './package.json' is not defined by "exports"` from Cherry Studio's `@anthropic-ai/claude-agent-sdk` dependency.
- Direct Shell MCP invocation of `powershell` failed because the executable name was not found in PATH.
- Direct Shell MCP invocation of the full PowerShell path failed with `spawn ... ENOENT`, indicating command/argument handling expected a simpler executable entry point.
- Shell MCP commands using complex inline PowerShell snippets were fragile because quoting and escaping were interpreted across multiple layers.
- The script error `表达式或语句中包含意外的标记“)”` was traced to a missing closure after the `fetch` hashtable block, not to the reported closing parenthesis itself.

### Cause Analysis

- Filesystem MCP and Shell MCP have different responsibilities; using Shell MCP for file inspection or edits increases risk and bypasses safer file-specific operations.
- Shell MCP command execution depends on executable resolution and argument splitting; Windows commands with spaces or complex quoting can fail if not routed through a stable entry point such as `cmd /c`.
- Parser errors at a closing delimiter often indicate an earlier unmatched `{`, `(`, array item separator, or hashtable closure issue.
- The `glob` failure was a tool/runtime issue in the Cherry Studio environment, so fallback to `ls` or targeted `read` was necessary.

### Fix

- Used Filesystem MCP for directory listing, reading files, and editing documentation.
- Reserved Shell MCP for PowerShell command validation attempts only.
- Switched from failed `glob` discovery to `filesystem.ls` recursive inspection.
- Avoided complex inline PowerShell validation after Shell MCP command handling proved fragile.
- Fixed the script by adding the missing hashtable closure before continuing validation attempts.

### Follow-Up Rules

- Use Filesystem MCP for all normal file reads, writes, edits, deletes, and directory inspection.
- Use Shell MCP only for execution and validation tasks, such as PowerShell parser checks or running scripts.
- When Shell MCP cannot resolve `powershell`, try a stable Windows entry point such as `cmd /c` before changing the target script.
- Prefer simple command invocations over complex inline PowerShell snippets through Shell MCP.
- If `glob` fails due to MCP runtime errors, fall back to `ls`, `read`, or another targeted Filesystem MCP operation and record the tool failure.
- For PowerShell parse errors reported at `)`, inspect earlier hashtable and array closures before changing the reported line.
