# MCP Issue Log

## Purpose

This file records MCP-related incidents, errors, causes, fixes, and historical troubleshooting notes. It is intentionally separated from `mcp_usage_rules.md` so the rules file remains concise and cheap to read frequently.

Future assistants should read this file only when troubleshooting, investigating repeated errors, or appending a new incident record.

## Issue Record Template

```markdown
## Issue: Short Title

### Type

incident | refactor-record | source-review | maintenance-note

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

### Type

incident

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

> ⚠️ SUPERSEDED by `PowerShell Array Hashtable Separators In mcp-oneclick.ps1` on the same date. Keep this record as historical context only; do not follow its comma-free array guidance.

### Type

incident

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

> ⚠️ SUPERSEDED by `PowerShell Array Hashtable Separators In mcp-oneclick.ps1` on the same date. Do not use the comma-free array rule below; current rule requires commas between adjacent hashtable elements in PowerShell arrays.

- For generated PowerShell scripts, prefer comma-free multi-line hashtable elements in complex arrays. **SUPERSEDED: do not follow.**
- Do not point `RedirectStandardOutput` and `RedirectStandardError` to the same file.
- After script creation or edits, verify key sections and run syntax parsing when feasible.

## Issue: Split Rules And Logs To Reduce Token Waste

### Type

refactor-record

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

### Type

refactor-record

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

### Type

incident

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

### Type

incident

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

## Issue: Collapsed Knowledge Data Iteration Run Path

### Type

incident

### Date

2026-07-02

### MCP Service

- Service: CherryFilesystem
- Tool: read, write, edit

### Symptoms

- The assistant reported or reused the run path as `F:\AIStudio\Cherry\CherryFile\CherryProject\Project_EldenRing\Knowdledge_DataIterationRuns\20260702_0134_data_iter_01`.
- The correct path keeps `_DataIterationRuns` as a child directory under `Knowdledge`: `F:\AIStudio\Cherry\CherryFile\CherryProject\Project_EldenRing\Knowdledge\_DataIterationRuns\20260702_0134_data_iter_01`.
- Collapsing `Knowdledge\_DataIterationRuns` into `Knowdledge_DataIterationRuns` can make later stages point to a non-existent run root.

### Cause Analysis

- The separator between the target data directory `Knowdledge` and the historical run directory `_DataIterationRuns` was dropped when composing the next-stage path.
- The final handoff path was not rechecked against the confirmed run root before being reported.

### Fix

- Record the canonical run-root pattern as `...\Knowdledge\_DataIterationRuns\<run_id>` for this project.
- If a user-supplied path is invalid or uses the collapsed `Knowdledge_DataIterationRuns` pattern, warn that the provided path is invalid and stop any workflow operation that depends on that path.
- Treat any possible corrected path as a candidate only; use it in handoffs or `NEXT_MESSAGE_TO_COPY` only after the user provides or confirms the valid path.
- Regenerate or withhold any affected next-step output that would otherwise contain the invalid `run_config_path` or `handoff_path`.

### Follow-Up Rules

- Do not collapse a parent directory and a child directory when composing Windows paths; preserve the `\` separator.
- For Project_EldenRing data-iteration runs, the run directory is under `Knowdledge\_DataIterationRuns`, not `Knowdledge_DataIterationRuns`.
- If a provided path does not resolve, do not infer-and-continue, do not run the dependent stage, and do not output that invalid path in next-step instructions.
- Before returning `NEXT_MESSAGE_TO_COPY`, verify that both `run_config_path` and `handoff_path` use a user-confirmed valid run root; otherwise report a blocker instead of producing a copyable next message.

## Issue: Collapsed Run Path Recurred via Markdown Escaping in NEXT_MESSAGE_TO_COPY

### Type

incident

### Date

2026-07-02

### MCP Service

- Service: CherryFilesystem
- Tool: ls, read, write

### Symptoms

- S03 emitted `NEXT_MESSAGE_TO_COPY` with the path `...\Knowdledge\_DataIterationRuns\...` as plain text in the chat response.
- Markdown rendering treated `\_` as an escaped underscore, so the copied message contained the collapsed invalid path `Knowdledge_DataIterationRuns`.
- S04 then ran against the invalid collapsed path, could not find `run_config.json` or `handoff_S03.md`, and produced blocker artifacts inside a newly created wrong directory `Knowdledge_DataIterationRuns\20260702_0134_data_iter_01`.
- The user reported that `run_config.json` and `handoff_S03.md` "do not exist" even though both files exist at the correct run root `Knowdledge\_DataIterationRuns\20260702_0134_data_iter_01`.

### Cause Analysis

- The earlier fix only banned known-invalid paths but did not require paths to be protected from markdown escaping; a valid path was corrupted at the rendering/copy layer, not at composition time.
- `NEXT_MESSAGE_TO_COPY` was output as plain prose instead of a fenced code block, so `\_` collapsed into `_` when copied.
- S04 created the wrong directory as a side effect, leaving stray artifacts (`change_set.md`, `file_changes_log.md`, `handoff_S04.md`) under the invalid root.

### Fix

- Added core rules: copyable paths and `NEXT_MESSAGE_TO_COPY` must be emitted inside fenced code blocks or inline code; verify a copyable path resolves on disk in the current session before emitting it.
- Re-verified that `run_config.json`, `handoff_S02.md`, `handoff_S03.md`, and `iteration_plan.md` all exist at the correct run root; S03 outputs did not need regeneration.
- Flagged the stray `Knowdledge_DataIterationRuns\20260702_0134_data_iter_01` directory (S04 blocker artifacts) as invalid-run debris; deletion requires user confirmation.

### Follow-Up Rules

- Emit every `NEXT_MESSAGE_TO_COPY` inside a fenced code block; never as plain prose.
- Emit final status blocks containing copyable paths inside a fenced code block, or wrap every path in inline code, so `\\_` cannot be collapsed by Markdown rendering or copying.
- Treat "file does not exist" reports as possibly a path-corruption issue; check both the collapsed and correct variants before assuming outputs were never written.
- When a wrong-path stage run leaves debris directories, report them and ask the user before deleting.

## Issue: Incomplete Task Completion Due to Execution Drift

### Type

incident

### Date

2026-07-02

### MCP Service

- Service: CherryFilesystem
- Tool: read, edit

### Symptoms

- The assistant repeatedly produced analysis or status explanations before ensuring the requested deliverable was fully completed.
- Environment probing, path discussion, or troubleshooting text was treated as progress even when the user requested a concrete write/edit/action.
- Final responses sometimes reported next steps or diagnostics while the actual workflow record update was incomplete or insufficiently verified.

### Cause Analysis

- The final deliverable was not kept as the controlling checkpoint throughout the task.
- Context-gathering tools such as `ls`, `read`, `grep`, and `glob` were allowed to expand the investigation instead of immediately driving the required write/edit/action.
- The assistant did not consistently self-check after each tool call whether the call advanced the final user-requested outcome.
- Repeated checks, invalid path handling, and explanation generation consumed the task budget before the requested completion criteria were satisfied.

### Fix

- Add a concise high-frequency rule requiring final-deliverable completion confirmation before the final response.
- Keep this full incident record in `mcp_issue_log.md` and keep only the reusable short rule in `mcp_usage_rules.md`.
- For future workflow tasks, first identify the required write/edit/action, perform only the minimum necessary context gathering, execute the deliverable, and verify the key written content once.

### Follow-Up Rules

- Do not answer with a diagnosis when the user requested that the diagnosis be written into a workflow file; write first, then respond minimally.
- Treat context gathering as incomplete until it has been converted into the requested artifact, edit, command result, or final deliverable.
- Before final response, check: requested deliverable identified, required file/action completed, key output verified once, and final answer matches the user-requested brevity.

## Issue: User-Supplied Path Segment Was Merged During Workflow Output

### Type

incident

### Date

2026-07-03

### MCP Service

- Service: CherryFilesystem
- Tool: read, edit

### Symptoms

- A workflow response referred to a run directory as `Knowdledge_DataIterationRuns` instead of preserving the user-supplied two-level path `Knowdledge\_DataIterationRuns`.
- The path segment boundary between `Knowdledge` and `_DataIterationRuns` was lost during response composition or copyable path handling.
- The user asked to archive a general rule so the failure is prevented in future workflows.

### Cause Analysis

- A supplied path was treated as a string to be normalized or reconstructed instead of as an immutable workflow input.
- The assistant did not explicitly preserve directory segment boundaries when restating `run_config_path`, `handoff_path`, output paths, or next-stage copy messages.
- Existing rules covered validation and markdown protection but did not directly state that paths from users, config files, and handoffs must remain verbatim.

### Fix

- Added a concise reusable rule to `mcp_usage_rules.md`: preserve paths from users, config files, and handoff files verbatim; do not manually splice, compress, rename, beautify, or merge path segments.
- Kept the full incident record in `mcp_issue_log.md` so the high-frequency rule card remains concise.

### Follow-Up Rules

- Treat user/config/handoff paths as immutable values unless the user explicitly asks for transformation.
- When emitting workflow paths, copy from the confirmed source path and preserve all separators and segment boundaries.
- Keep only the short reusable path-preservation rule in the high-frequency MCP rule file; keep details of this incident in the issue log.
