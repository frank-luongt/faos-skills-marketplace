# MCP Best Practices

Core guidelines for building high-quality MCP servers.

## Server and Tool Naming

- **Server name**: Use descriptive, lowercase names with hyphens (e.g., `github-mcp-server`, `postgres-mcp-server`)
- **Tool naming**: Use consistent prefixes matching the service (e.g., `github_create_issue`, `github_list_repos`)
- **Action-oriented**: Start tool names with verbs (`search_`, `create_`, `update_`, `delete_`, `list_`, `get_`)
- **Discoverability**: Agents scan tool names and descriptions at startup — clear naming reduces selection errors

## Tool Descriptions

- Keep descriptions concise but complete (1-3 sentences)
- Include the "when to use" context: "Use this tool when you need to search for GitHub issues matching specific criteria"
- Document parameter constraints inline: "limit (1-100, default 10)"
- Mention return format: "Returns JSON array of issue objects with id, title, state, assignee"

## Response Format

- **Structured data**: Return JSON for data that agents will process programmatically
- **Human-readable**: Return Markdown for summaries, reports, or content meant for display
- **Hybrid approach**: Use `structuredContent` (TypeScript SDK) to return both structured data and text content
- **Pagination**: Always support pagination for list operations. Return `hasMore` flag and `nextCursor`

## Pagination

```typescript
// Input schema
{
  cursor: z.string().optional().describe("Pagination cursor from previous response"),
  limit: z.number().min(1).max(100).default(20).describe("Results per page")
}

// Response format
{
  content: [{ type: "text", text: "..." }],
  structuredContent: {
    items: [...],
    pagination: { hasMore: true, nextCursor: "abc123", total: 450 }
  }
}
```

## Error Handling

- Return actionable error messages: "Repository not found. Check that the owner/repo format is correct (e.g., 'vercel/next.js')"
- Include suggested next steps: "Authentication failed. Verify your API token has 'repo' scope"
- Use `isError: true` in tool responses for errors
- Never expose raw stack traces or internal errors to agents

## Transport Selection

| Transport | When to Use | Characteristics |
|-----------|-------------|-----------------|
| **stdio** | Local tools, CLI integrations, desktop apps | Process-based, simple, no networking |
| **Streamable HTTP** | Remote servers, web services, multi-user | HTTP-based, scalable, stateless JSON recommended |

**Default choice**: stdio for local, Streamable HTTP for remote.

**Stateless JSON** (recommended for HTTP): Each request is independent. No session management. Simpler to scale and maintain than stateful streaming sessions.

## Authentication

- Use environment variables for API keys and tokens — never hardcode
- Support multiple auth methods: API key, OAuth token, service account
- Validate credentials on server startup, fail fast with clear error
- For multi-tenant: pass tenant context via tool parameters or request headers

## Security

- Validate all input parameters (use Zod/Pydantic schemas with constraints)
- Sanitize outputs to prevent injection
- Rate limit tool calls to prevent abuse
- Log all tool invocations for audit trails
- Use HTTPS for all remote transports
- Never return raw database queries or internal system details

## Tool Annotations

Annotations help clients understand tool behavior:

```typescript
server.registerTool("delete_repository", {
  annotations: {
    readOnlyHint: false,       // Modifies state
    destructiveHint: true,     // Irreversible action
    idempotentHint: false,     // Multiple calls have different effects
    openWorldHint: false       // Operates on specific known resources
  }
});
```

| Annotation | Purpose |
|-----------|---------|
| `readOnlyHint` | Tool only reads data, no side effects |
| `destructiveHint` | Action cannot be undone |
| `idempotentHint` | Safe to retry — same result each time |
| `openWorldHint` | Interacts with external/unknown resources |

## Performance

- Use async/await for all I/O operations
- Batch API calls where possible (e.g., fetch multiple resources in one call)
- Cache frequently requested data (LRU cache for cross-request, React.cache for per-request)
- Set reasonable timeouts on external API calls (30s default)
- Return partial results if a batch operation partially fails

## Testing

- Test each tool independently with valid and invalid inputs
- Test error paths: auth failure, rate limiting, network timeout, malformed input
- Test pagination: first page, middle page, last page, empty results
- Use MCP Inspector for interactive testing: `npx @modelcontextprotocol/inspector`
- Write integration tests that verify end-to-end tool execution
