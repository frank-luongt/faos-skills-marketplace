# Python MCP Server Guide

Complete guide for building MCP servers with Python and FastMCP.

## Project Setup

```bash
mkdir my-mcp-server && cd my-mcp-server
python -m venv .venv && source .venv/bin/activate  # or .venv\Scripts\activate on Windows
pip install "mcp[cli]"
```

### Dependencies (pyproject.toml)

```toml
[project]
name = "my-mcp-server"
version = "1.0.0"
requires-python = ">=3.10"
dependencies = [
    "mcp[cli]>=1.0.0",
    "httpx>=0.27.0",
]

[project.scripts]
my-mcp-server = "my_mcp_server:main"
```

## Project Structure

```
my_mcp_server/
├── __init__.py        # Package init
├── __main__.py        # Entry point: python -m my_mcp_server
├── server.py          # Server definition and tool registration
├── tools/
│   ├── __init__.py
│   ├── search.py      # Search-related tools
│   ├── crud.py        # CRUD operations
│   └── analytics.py   # Analytics tools
└── lib/
    ├── __init__.py
    ├── api_client.py   # Shared HTTP client with auth
    ├── errors.py       # Error helpers
    └── formatting.py   # Response formatting
```

## Server Initialization

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My Service Server")
```

### Entry Point

```python
# __main__.py
from .server import mcp

def main():
    mcp.run(transport="stdio")

if __name__ == "__main__":
    main()
```

### Streamable HTTP Transport

```python
from .server import mcp

def main():
    mcp.run(transport="streamable-http", host="0.0.0.0", port=3000)
```

## Tool Registration

### Basic Tool

```python
from mcp.server.fastmcp import FastMCP
from pydantic import Field

mcp = FastMCP("Issue Tracker")

@mcp.tool()
async def search_issues(
    query: str = Field(description="Search query (e.g., 'bug label:critical')"),
    state: str = Field(default="open", description="Filter: open, closed, or all"),
    limit: int = Field(default=20, ge=1, le=100, description="Max results"),
) -> str:
    """Search for issues by query. Returns matching issues with title, state, and assignee."""
    results = await api_client.search_issues(query, state, limit)
    return format_issue_list(results)
```

### Tool with Structured Return

```python
from pydantic import BaseModel

class UserInfo(BaseModel):
    id: int
    login: str
    name: str | None
    email: str | None
    public_repos: int

@mcp.tool()
async def get_user(
    username: str = Field(description="GitHub username"),
) -> UserInfo:
    """Get user details by username."""
    data = await api_client.get_user(username)
    return UserInfo(**data)
```

### Tool with Context (for logging, progress)

```python
from mcp.server.fastmcp import Context

@mcp.tool()
async def long_analysis(
    repo: str = Field(description="Repository in owner/repo format"),
    ctx: Context = None,
) -> str:
    """Run comprehensive repository analysis."""
    await ctx.info(f"Starting analysis of {repo}...")

    # Report progress
    await ctx.report_progress(1, 5)
    contributors = await api_client.get_contributors(repo)

    await ctx.report_progress(3, 5)
    issues = await api_client.get_issues(repo)

    await ctx.report_progress(5, 5)
    return format_analysis(contributors, issues)
```

## Pydantic Schema Patterns

```python
from pydantic import Field
from typing import Literal

# Enum-like constraint
state: Literal["open", "closed", "all"] = Field(default="open", description="Issue state")

# Numeric range
limit: int = Field(default=20, ge=1, le=100, description="Results per page")

# Optional with description
cursor: str | None = Field(default=None, description="Pagination cursor from previous response")

# List input
labels: list[str] = Field(default_factory=list, max_length=10, description="Label names to filter")

# Date string
since: str | None = Field(default=None, description="ISO date (e.g., '2025-01-15T00:00:00Z')")
```

## Error Handling

```python
from mcp.types import TextContent

@mcp.tool()
async def get_repository(
    repo: str = Field(description="Repository in owner/repo format"),
) -> str:
    """Get repository details."""
    try:
        data = await api_client.get_repo(repo)
        return format_repo(data)
    except httpx.HTTPStatusError as e:
        if e.response.status_code == 404:
            return f"Repository '{repo}' not found. Check the owner/repo format (e.g., 'vercel/next.js')."
        if e.response.status_code == 401:
            return "Authentication failed. Verify your API token has the required scopes."
        raise
```

## API Client Pattern

```python
# lib/api_client.py
import os
import httpx

class ApiClient:
    def __init__(self):
        self.base_url = os.environ.get("API_BASE_URL", "")
        self.token = os.environ.get("API_TOKEN", "")

        if not self.token:
            raise ValueError("API_TOKEN environment variable is required")

        self.client = httpx.AsyncClient(
            base_url=self.base_url,
            headers={
                "Authorization": f"Bearer {self.token}",
                "Content-Type": "application/json",
            },
            timeout=30.0,
        )

    async def request(self, method: str, path: str, **kwargs):
        response = await self.client.request(method, path, **kwargs)
        response.raise_for_status()
        return response.json()

    async def get(self, path: str, **kwargs):
        return await self.request("GET", path, **kwargs)

    async def post(self, path: str, **kwargs):
        return await self.request("POST", path, **kwargs)

api_client = ApiClient()
```

## Resources (Optional)

```python
@mcp.resource("config://settings")
async def get_settings() -> str:
    """Expose server configuration as a resource."""
    return json.dumps({
        "api_version": "2025-01",
        "rate_limit": 100,
        "features": ["search", "crud", "analytics"],
    })
```

## Build and Test

```bash
# Verify syntax
python -m py_compile my_mcp_server/server.py

# Run with stdio (for Claude Desktop / Claude Code)
python -m my_mcp_server

# Run with HTTP (for remote access)
python -m my_mcp_server --transport streamable-http --port 3000

# Test with MCP Inspector
npx @modelcontextprotocol/inspector python -m my_mcp_server
```

## Quality Checklist

- [ ] All tools have docstrings describing purpose and return format
- [ ] Pydantic Field() used with descriptions and constraints on all parameters
- [ ] Error responses return helpful guidance (not raw exceptions)
- [ ] List operations support pagination (cursor + limit)
- [ ] No hardcoded credentials — all via environment variables
- [ ] `python -m py_compile` passes for all modules
- [ ] Each tool tested with MCP Inspector
- [ ] Edge cases handled: empty results, invalid input, auth failure, timeouts
- [ ] Async/await used for all I/O operations
