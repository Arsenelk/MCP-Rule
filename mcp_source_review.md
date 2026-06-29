# MCP Source Review Notes

## Purpose

This file records external source checks used to update the MCP usage knowledge base. It is low-frequency and should be read only during update, audit, or source verification tasks.

## 2026-06-28 Source Review

### Confirmed Sources

1. `https://modelcontextprotocol.io/specification/2025-06-18`
   - Status: fetched successfully.
   - Use: protocol overview, hosts/clients/servers, JSON-RPC, features, security principles.
   - Quality tier: Tier 1 official specification.

2. `https://modelcontextprotocol.io/specification/2025-06-18/server/tools.md`
   - Status: fetched successfully through docs redirect.
   - Use: tools capability, tool listing/calling, tool result types, structured content, output schema, tool errors, security considerations.
   - Quality tier: Tier 1 official specification.

3. `https://modelcontextprotocol.io/specification/2025-06-18/server/resources.md`
   - Status: fetched successfully through docs redirect.
   - Use: resources capability, resource discovery/read, templates, subscriptions, annotations, URI schemes, resource security.
   - Quality tier: Tier 1 official specification.

4. `https://modelcontextprotocol.io/specification/2025-06-18/server/prompts.md`
   - Status: fetched successfully through docs redirect.
   - Use: prompts capability, user-controlled prompt model, prompt listing/getting, arguments, message content, prompt security.
   - Quality tier: Tier 1 official specification.

### Search Notes

- Generic search queries for “MCP best practices” returned noisy or low-authority results.
- The update therefore prioritized official MCP specification pages over blog posts and aggregator pages.
- Future updates should begin from `https://modelcontextprotocol.io/llms.txt` or the official specification index before using search-engine results.

### Knowledge Base Changes Made

- Created `mcp_auto_update_protocol.md` for future update workflow and admission rules.
- Created `mcp_high_quality_knowledge.md` for medium-detail MCP concepts and best practices.
- Created this source review file to keep external-source notes out of the high-frequency rule card.
- Updated `mcp_usage_rules.md` with only short routing rules.

## Future Source Review Checklist

- [ ] Check the latest official MCP specification version.
- [ ] Check whether `tools`, `resources`, `prompts`, `sampling`, `roots`, or `elicitation` behavior changed.
- [ ] Check official SDK release notes if implementation behavior matters.
- [ ] Check whether any local incident created a reusable rule.
- [ ] Keep high-frequency rules short.
- [ ] Store source notes here, not in `mcp_usage_rules.md`.
