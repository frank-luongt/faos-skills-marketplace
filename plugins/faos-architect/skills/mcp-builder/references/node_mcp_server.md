# TypeScript MCP Server Guide

Complete guide for building MCP servers with the TypeScript SDK.

## Project Setup

```bash
mkdir my-mcp-server && cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "declaration": true
  },
  "include": ["src/**/*"]
}
```

### package.json additions

```json
{
  "type": "module",
  "bin": { "my-mcp-server": "./dist/index.js" },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "inspect": "npx @modelcontextprotocol/inspector node dist/index.js"
  }
}
```

## Project Structure

```
src/
├── index.ts           # Server entry point and transport setup
├── tools/             # One file per tool or tool group
│   ├── search.ts
│   ├── crud.ts
│   └── analytics.ts
├── lib/
│   ├── api-client.ts  # Shared API client with auth
│   ├── errors.ts      # Error handling utilities
│   └── formatting.ts  # Response formatting helpers
└── types.ts           # Shared type definitions
```

## Server Initialization

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
  name: "my-service",
  version: "1.0.0",
});

// Register tools (see below)

// Start with stdio transport
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Streamable HTTP Transport

```typescript
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import express from "express";

const app = express();
app.use(express.json());

app.post("/mcp", async (req, res) => {
  const transport = new StreamableHTTPServerTransport("/mcp", res);
  await server.connect(transport);
  await transport.handleRequest(req, res);
});

app.listen(3000, () => console.log("MCP server on port 3000"));
```

## Tool Registration

### Basic Tool

```typescript
import { z } from "zod";

server.registerTool(
  "search_issues",
  {
    description: "Search for issues by query string. Returns matching issues with title, state, and assignee.",
    inputSchema: {
      query: z.string().describe("Search query (e.g., 'bug label:critical')"),
      state: z.enum(["open", "closed", "all"]).default("open").describe("Issue state filter"),
      limit: z.number().min(1).max(100).default(20).describe("Max results to return"),
    },
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: false,
    },
  },
  async ({ query, state, limit }) => {
    const results = await apiClient.searchIssues(query, state, limit);

    return {
      content: [
        {
          type: "text",
          text: formatIssueList(results),
        },
      ],
    };
  }
);
```

### Tool with Output Schema (Structured Content)

```typescript
server.registerTool(
  "get_user",
  {
    description: "Get user details by username.",
    inputSchema: {
      username: z.string().describe("GitHub username"),
    },
    outputSchema: {
      id: z.number(),
      login: z.string(),
      name: z.string().nullable(),
      email: z.string().nullable(),
      public_repos: z.number(),
    },
  },
  async ({ username }) => {
    const user = await apiClient.getUser(username);

    return {
      structuredContent: {
        id: user.id,
        login: user.login,
        name: user.name,
        email: user.email,
        public_repos: user.public_repos,
      },
    };
  }
);
```

## Zod Schema Patterns

```typescript
// Enum with description
z.enum(["open", "closed", "all"]).default("open").describe("Filter by state")

// Optional with default
z.number().min(1).max(100).default(20).describe("Results per page")

// String with format hint
z.string().describe("Date in ISO format (e.g., '2025-01-15T00:00:00Z')")

// Array input
z.array(z.string()).min(1).max(10).describe("List of label names to filter by")

// Pagination cursor
z.string().optional().describe("Cursor from previous response for pagination")
```

## Error Handling

```typescript
import { McpError, ErrorCode } from "@modelcontextprotocol/sdk/types.js";

// In tool handler
async ({ repo }) => {
  try {
    const data = await apiClient.getRepo(repo);
    return { content: [{ type: "text", text: formatRepo(data) }] };
  } catch (error) {
    if (error.status === 404) {
      return {
        content: [{
          type: "text",
          text: `Repository '${repo}' not found. Check the owner/repo format (e.g., 'vercel/next.js').`,
        }],
        isError: true,
      };
    }
    if (error.status === 401) {
      throw new McpError(
        ErrorCode.InvalidRequest,
        "Authentication failed. Verify your API token."
      );
    }
    throw error;
  }
};
```

## API Client Pattern

```typescript
// src/lib/api-client.ts
import { McpError, ErrorCode } from "@modelcontextprotocol/sdk/types.js";

class ApiClient {
  private baseUrl: string;
  private token: string;

  constructor() {
    this.baseUrl = process.env.API_BASE_URL ?? "";
    this.token = process.env.API_TOKEN ?? "";

    if (!this.token) {
      throw new McpError(
        ErrorCode.InvalidRequest,
        "API_TOKEN environment variable is required"
      );
    }
  }

  async request<T>(path: string, options: RequestInit = {}): Promise<T> {
    const response = await fetch(`${this.baseUrl}${path}`, {
      ...options,
      headers: {
        Authorization: `Bearer ${this.token}`,
        "Content-Type": "application/json",
        ...options.headers,
      },
    });

    if (!response.ok) {
      const body = await response.text();
      throw Object.assign(new Error(`API error: ${response.status}`), {
        status: response.status,
        body,
      });
    }

    return response.json() as T;
  }
}

export const apiClient = new ApiClient();
```

## Build and Test

```bash
# Build
npm run build

# Test with MCP Inspector (interactive)
npx @modelcontextprotocol/inspector node dist/index.js

# Test with environment variables
API_TOKEN=xxx npx @modelcontextprotocol/inspector node dist/index.js
```

## Quality Checklist

- [ ] All tools have clear descriptions with usage context
- [ ] Input schemas use Zod with constraints and descriptions
- [ ] Error responses include actionable guidance
- [ ] List operations support pagination (cursor + limit)
- [ ] Annotations set correctly (readOnly, destructive, idempotent)
- [ ] No hardcoded credentials — all via environment variables
- [ ] `npm run build` succeeds with no type errors
- [ ] Each tool tested with MCP Inspector
- [ ] Edge cases handled: empty results, invalid input, auth failure
