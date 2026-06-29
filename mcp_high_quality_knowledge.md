# MCP High-Quality Knowledge Notes

## Purpose

This file stores medium-detail MCP knowledge that is useful for future assistants but too detailed for the high-frequency `mcp_usage_rules.md` rule card. Read this file when designing MCP workflows, reviewing MCP tool usage, updating MCP rules, or diagnosing whether a tool should be used.

## 1. MCP Conceptual Model

MCP standardizes how LLM applications connect to external data sources, tools, and workflows. A host is the LLM application that initiates connections, a client is the connector inside the host, and a server provides context or capabilities.

MCP communication is based on JSON-RPC 2.0 messages, stateful connections, and capability negotiation. Future rules should account for what each server or client actually declares as supported rather than assuming every MCP feature is available.

Server-side features include:

- **Resources**: contextual data such as files, schemas, documentation, or application-specific information.
- **Prompts**: reusable prompt templates and structured workflows exposed to users.
- **Tools**: executable capabilities that the model can invoke to query systems, call APIs, compute results, or perform operations.

Client-side features include:

- **Sampling**: server-initiated requests for LLM completion behavior.
- **Roots**: server inquiries about filesystem or URI boundaries where it may operate.
- **Elicitation**: server requests for additional user-provided information.

Additional protocol utilities include progress tracking, cancellation, error reporting, logging, pagination, and list-change notifications.

## 2. Choosing Between Resources, Prompts, and Tools

Use **Resources** when the assistant needs to read or reference data and the safest behavior is “retrieve context, then reason.” Examples: documentation, code files, database schemas, style guides, static project metadata, local notes.

Use **Prompts** when the server should expose reusable user-triggered workflows. Examples: “review this code,” “summarize this document,” “generate release notes,” or “run a structured diagnostic checklist.”

Use **Tools** when the assistant must perform an action, computation, query, mutation, external API call, or operation with side effects. Examples: file write, database query, browser navigation, command execution, ticket creation, deployment, or API mutation.

A good MCP knowledge base should distinguish these clearly because confusing a resource with a tool can create unnecessary risk: resources provide context, while tools may execute behavior.

## 3. Tool Design and Invocation Rules

MCP tools should have names that are stable, specific, and action-oriented. Avoid vague names such as `do_task`; prefer names that reveal scope and effect, such as `read_file`, `search_web`, `execute_command`, or `create_issue`.

Each tool should have a clear human-readable description explaining:

- what the tool does;
- what inputs are required;
- what output shape to expect;
- whether the tool has side effects;
- whether it can access private or local data;
- whether it can mutate files, run commands, or interact with external services.

Input schemas should be strict enough to prevent ambiguous invocation. Prefer explicit required fields, bounded enums, path constraints, maximum limits, and clearly documented defaults.

When a tool provides `outputSchema`, clients should validate structured results before passing them to the model. Structured outputs reduce parsing ambiguity and make downstream reasoning safer.

Tool results may include unstructured content, structured content, resource links, or embedded resources. If the result is large, prefer returning a summary plus resource links instead of dumping unnecessary content into the conversation.

## 4. Tool Safety Rules

Tools can represent arbitrary code execution or external side effects, so they require stronger caution than passive context retrieval.

Sensitive operations should require user confirmation when they can:

- modify or delete files;
- execute shell commands;
- send network requests with credentials;
- mutate databases;
- publish, email, or message external parties;
- spend money or consume paid quota;
- access private user data;
- install dependencies or change system configuration.

Tool annotations and descriptions should not be blindly trusted unless they come from a trusted server. A malicious or compromised server can describe a dangerous operation as safe, so the assistant should reason from tool name, parameters, server identity, and actual task context.

Before invoking a sensitive tool, the assistant should be able to answer:

1. What final deliverable does this call advance?
2. What data will be accessed or modified?
3. Is the operation reversible?
4. Is user confirmation required?
5. What is the minimal scope needed?
6. How will the result be verified?

## 5. Filesystem MCP Knowledge

Filesystem MCP is the preferred path for normal file operations: directory inspection, file reading, writing, editing, deletion, and targeted verification.

Use `ls` only to confirm visible structure when needed. Once directory state is known, switch to the deliverable action rather than repeatedly probing.

Use `glob` when file names or patterns are unknown. If `glob` fails because of runtime/tooling issues, fall back to `ls`, `read`, or another targeted Filesystem operation and record the failure if it is reusable.

Use `grep` when searching content across files. Do not replace actual editing or writing with repeated search calls.

Use `read` before editing existing files. This prevents accidental overwrite and lets the assistant preserve exact formatting and local conventions.

Use `edit` for precise replacements in existing files. Use `write` for new files or confirmed full-file replacement only.

After content changes, verify the key written or edited content with one targeted read when useful. Verification should be proportional: do not re-read entire large files if a small section proves success.

## 6. Shell MCP Knowledge

Shell MCP should be reserved for command execution, runtime checks, parser checks, build/test commands, and validations that cannot be performed through Filesystem MCP.

Do not use Shell MCP for ordinary file reads and edits when Filesystem MCP is available. Shell-based file manipulation increases quoting risk, bypasses safer file-specific operations, and can make changes harder to audit.

On Windows, prefer simple command invocations. Complex inline PowerShell snippets can fail due to multi-layer escaping and executable resolution issues. If direct `powershell` cannot be resolved, try a stable entry point such as `cmd /c` when appropriate.

Shell output should be interpreted cautiously. A failing parser location often points near the symptom, not necessarily the root cause.

## 7. Prompt and Resource Design Rules

MCP prompts are user-controlled templates. They are best used when users explicitly choose a workflow, such as slash commands or menu actions.

Prompt arguments should be validated before processing. Missing or invalid arguments should return clear errors rather than producing vague model behavior.

Prompt messages can include text, images, audio, or embedded resources. Embedded resources should include valid URIs and MIME types and should avoid forcing unnecessary large context into the conversation.

MCP resources are application-driven context. Good resources have stable URIs, useful names, optional human-readable titles, descriptions, MIME types, and when available, size or modification metadata.

Resource annotations such as audience, priority, and lastModified can help clients decide what to include in context. Use these hints to avoid wasting tokens on low-priority resources.

## 8. Error Handling and Logging

Distinguish protocol errors from execution errors.

Protocol errors include invalid methods, invalid parameters, unknown tool names, unavailable resources, or internal server errors. These should be reported through JSON-RPC error responses where applicable.

Execution errors occur when the tool call itself ran but the operation failed, such as an API rate limit, invalid business input, missing permission, or external service failure. These may be represented in a tool result with an error flag.

For knowledge-base maintenance, record reusable incidents in `mcp_issue_log.md` only when they produce future value. Do not log every transient failure.

A useful incident record includes:

- short title;
- date;
- MCP service and tool;
- symptoms;
- cause analysis;
- fix;
- follow-up rule.

## 9. Context and Token Efficiency

The MCP rules knowledge base should be organized by reading frequency:

- high-frequency stable rules in `mcp_usage_rules.md`;
- incidents and historical troubleshooting in `mcp_issue_log.md`;
- conditional domain rules in domain-specific files;
- update process in `mcp_auto_update_protocol.md`;
- conceptual and best-practice notes in this file.

Do not put long tutorials, web excerpts, or incident histories into the high-frequency file. Instead, add a short routing line that tells future assistants when to read a low-frequency file.

When updating from external sources, summarize principles and cite source URLs in a source note or update report rather than pasting entire pages.

## 10. MCP Update Watchlist

When periodically updating this knowledge base, check for changes in:

- official MCP specification versions;
- server feature requirements for resources, prompts, and tools;
- client feature requirements for sampling, roots, and elicitation;
- security guidance for user consent and tool confirmation;
- SDK changes that affect tool schema, structured output, or transport;
- host-specific MCP behavior in tools such as IDEs, desktop clients, and local agent systems;
- known Windows/PowerShell invocation issues;
- known filesystem permission, path, and workspace-root constraints.

## 11. Practical Decision Checklist

Before calling an MCP tool, future assistants should check:

- Is the final deliverable clear?
- Is a tool call necessary, or can the answer be given directly?
- Which MCP service has the correct responsibility?
- Is this a read, write, execute, browse, memory, or validation task?
- Is user confirmation required because the operation is sensitive?
- Will this call duplicate a fact already known?
- What is the smallest useful result needed?
- What one verification step is appropriate after mutation?

## Source Notes

- Official MCP specification, 2025-06-18: `https://modelcontextprotocol.io/specification/2025-06-18`
- Official MCP Tools page: `https://modelcontextprotocol.io/specification/2025-06-18/server/tools.md`
- Official MCP Resources page: `https://modelcontextprotocol.io/specification/2025-06-18/server/resources.md`
- Official MCP Prompts page: `https://modelcontextprotocol.io/specification/2025-06-18/server/prompts.md`
