# MCP Knowledge Base Auto Update Protocol

## Purpose

This file defines how a future assistant should periodically update this MCP usage knowledge base. It is a low-frequency operational file. Read it only when the user asks to update, audit, expand, or refresh MCP usage knowledge.

## Update Trigger Phrases

Treat any of the following user requests as an update task:

- “更新 MCP 使用规则知识库”
- “搜集 MCP 最新规则/最佳实践并写入知识库”
- “检查 MCP 规则是否过期”
- “补充 MCP 工具调用规则”
- “review MCP 使用规则并升级数据库”

## Non-Negotiable Update Principles

1. Define the concrete deliverable before calling any tool.
2. Use Filesystem MCP for directory inspection, file reads, file writes, edits, deletes, and targeted verification.
3. Use Shell MCP only for runtime execution, syntax checks, parser checks, or validation that Filesystem MCP cannot perform.
4. Do not repeat `ls`, `read`, `grep`, or `glob` calls against the same fact unless state changed, a tool failed, or the user asked for recheck.
5. Keep `mcp_usage_rules.md` short; only add stable, reusable, task-agnostic rules there.
6. Keep detailed incidents in `mcp_issue_log.md`.
7. Keep conditional domain rules in separate files, such as `powershell_script_rules.md` or future language/tool-specific rule files.
8. Keep auto-update procedures in this file.
9. Keep research summaries and MCP conceptual knowledge in `mcp_high_quality_knowledge.md`.
10. After writing or editing, perform exactly one necessary targeted verification read unless the change is trivial and already confirmed by the tool response.

## Recommended Update Workflow

1. **Identify deliverable**: State whether the task is to update stable rules, incident logs, auto-update protocol, or conceptual MCP knowledge.
2. **Inspect once**: Use `ls` once only if visible structure is unknown.
3. **Read routing files**: Read `mcp_usage_rules.md`; read this file only for update tasks; read `mcp_issue_log.md` only for incidents or troubleshooting.
4. **Search external sources**: Prefer official MCP specification pages, official SDK repositories, official changelogs, and reputable implementation guides.
5. **Classify new knowledge**: Decide whether each item is a stable rule, incident, conditional rule, conceptual note, or source-review record.
6. **Write minimal high-frequency changes**: If a new rule belongs in `mcp_usage_rules.md`, compress it into one short reusable rule.
7. **Write detailed low-frequency knowledge**: Put detailed explanation, workflows, examples, and source notes in low-frequency files.
8. **Verify once**: Read the specific file sections changed or created.
9. **Report completion**: Summarize files changed and mention any sources that could not be verified.

## Source Quality Tiers

### Tier 1: Preferred Sources

- Official Model Context Protocol specification.
- Official MCP documentation pages.
- Official SDK repositories and release notes.
- Official host/client documentation for tools that implement MCP.

### Tier 2: Useful but Needs Cross-Check

- Well-maintained open-source MCP servers.
- Engineering blogs from reputable AI infrastructure teams.
- Security advisories and issue threads with reproducible details.

### Tier 3: Do Not Use as Sole Authority

- SEO tutorials with no source links.
- Aggregator pages listing MCP tools without technical details.
- Social posts without reproducible evidence.
- Articles that conflict with the official specification.

## Admission Rules for New Knowledge

Add new knowledge only when at least one condition is true:

- It changes how future assistants should safely call MCP tools.
- It prevents repeated failures already seen in this environment.
- It helps choose the right MCP service for a task.
- It clarifies protocol concepts needed to reason about MCP tools.
- It reduces token waste during long-term maintenance.
- It adds a stable verification or rollback practice.

Do not add knowledge when:

- It is a one-off observation with no reusable value.
- It is a long tutorial that can be summarized into a short rule.
- It duplicates existing rules without improving clarity.
- It belongs to a specific language or runtime but is being added to the high-frequency rule card.

## Long-Term Token Budget Rules

1. High-frequency files must be short enough to read before most MCP tasks.
2. Low-frequency files may be detailed but must have clear headings and task-specific sections.
3. Logs must be append-only and read only for troubleshooting.
4. Source-review notes must be summarized, not pasted as full web pages.
5. If a file grows too large, split it by reading frequency or domain.
6. Use route lines that say “read X only when Y” rather than requiring future assistants to scan all files.

## Suggested Periodic Update Prompt

```text
请更新 F:\AIStudio\Cherry\CherryFile\AssiatanceKnowledge\MCP使用规则 中的 MCP 使用规则知识库。先读取 mcp_usage_rules.md 和 mcp_auto_update_protocol.md，再检索官方 MCP 规范、官方文档、SDK 更新与高质量实践资料。只把稳定、可复用、能降低错误或 token 浪费的内容写入知识库；保持高频规则短，把详细内容放入低频专题文件，最后做一次定向验证并告诉我完成。
```

## Update Output Checklist

- [ ] Final deliverable stated before tool calls.
- [ ] `ls` not repeated unnecessarily.
- [ ] Existing files read before edit.
- [ ] New files written only when they have a clear reading frequency.
- [ ] Stable rule additions kept concise.
- [ ] Detailed knowledge separated from rules and logs.
- [ ] External sources classified by quality.
- [ ] One targeted verification performed.
- [ ] Final response lists changed files.
