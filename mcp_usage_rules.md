# MCP Usage Rules

## Purpose

This file is the high-frequency rule card for MCP usage. It must stay concise, stable, and free of incident logs.

## File Separation Rules

1. Keep stable MCP usage rules in `mcp_usage_rules.md` only.
2. Keep errors, incidents, causes, fixes, and history in `mcp_issue_log.md` only.
3. Keep PowerShell-specific script rules in `powershell_script_rules.md` only; read it only when creating, reviewing, or editing `.ps1` files.
4. Keep update procedures in `mcp_auto_update_protocol.md`; read it only when updating, auditing, or expanding this MCP knowledge base.
5. Keep medium-detail MCP concepts, official-spec notes, and best-practice summaries in `mcp_high_quality_knowledge.md`; read it only when designing MCP workflows or updating rules.
6. Keep source review notes in `mcp_source_review.md`; read it only during source verification or periodic updates.
7. Add new content to this file only when it is reusable across tasks, stable, and short.
8. When an issue produces a reusable rule, add only the concise rule here and put the full incident record in `mcp_issue_log.md`.

## Core MCP Rules

1. Identify the final deliverable before calling MCP tools.
2. Use each tool call to directly advance that deliverable.
3. Do not treat environment probing as task completion.
4. Do not repeat checks of the same fact unless state changed, a tool failed, or the user requested it.
5. If a tool fails, report the exact blocker or switch to a useful alternative instead of repeating the same call.
6. Validate user-supplied paths before dependent operations; if a required path is invalid, warn the user and stop until the path is provided or confirmed.
7. Preserve paths from users, config files, and handoff files verbatim; do not manually splice, compress, rename, beautify, or merge path segments.
8. Preserve confirmed path spelling exactly, including apparent typos in real directory names; do not autocorrect path segments unless the user explicitly asks and a verified replacement path exists.
9. Protect all copyable Windows paths and final status blocks containing paths in inline code or fenced code blocks.
10. Verify required paths resolve on disk before emitting copyable handoff or status content; never emit known-invalid or markdown-collapsed paths.
11. Before final response, confirm the requested deliverable was actually completed; do not substitute explanations, diagnostics, or environment probing for the required write/edit/action.
12. After creating, modifying, or deleting content, perform one necessary verification step.
13. Use Filesystem MCP for file reads, writes, edits, deletes, and directory inspection.
14. Use Shell MCP only for command execution, runtime checks, parser checks, or validation that cannot be done by Filesystem MCP.

## Filesystem MCP Rules

1. Use `ls` once only when the parent directory or visible structure must be confirmed.
2. Use `glob` for unknown file names or path patterns; use `grep` for content search.
3. Use `read` before editing an existing file.
4. Prefer `edit` for precise replacements in existing files.
5. Use `write` for new files or confirmed full-file replacement only.
6. Do not modify unrelated files.
7. Verify key written or edited content with one targeted `read` when needed.
