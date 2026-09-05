# MCP Python SDK Cheatsheet

## Transport Selection

| Transport | Use When | Notes |
|---|---|---|
| In-memory | Testing, embedding | `Client(mcp)`. No subprocess/port. |
| Streamable HTTP | Production | `Client("http://.../mcp")`. Stateful/stateless. |
| stdio | Desktop/IDE hosts | `Client(StdioServerParameters(...))`. Subprocess. |
| SSE | Legacy only | Deprecated. Don't build new on it. |
| Custom | WebSocket, Unix socket | `Client(transport)`. Implement protocol. |

## Server Class

| Need | Use |
|---|---|
| Auto schema, decorators | `MCPServer` |
| Hand-written schema, custom methods | Low-level `Server` |
| Pagination, middleware | Low-level `Server` |

## Error Handling

| Scenario | Client Sees |
|---|---|
| Tool raises `ToolError` | `is_error=True`, message in content |
| Tool raises other exception | `is_error=True`, generic message |
| Low-level handler raises | `-32603` MCPError |
| Unknown tool | `is_error=True`, "Unknown tool" |
| `MCPError` in `@mcp.tool()` | JSON-RPC error |
| Missing client capability | `-32021` MCPError |

## OAuth Decision

```
Need token?
├── No → In-memory/stdio
├── Human → OAuthClientProvider (browser flow)
│   └── Enterprise IdP → IdentityAssertionOAuthProvider
└── No human → ClientCredentialsOAuthProvider
```

## Elicitation Decision

```
User input mid-call?
├── Resolver (all versions) → Elicit(message, Model)
└── Direct (legacy only) → ctx.elicit() / ctx.elicit_url()
```

## Caching

| Hint | When |
|---|---|
| `ttl_ms=0, scope="private"` | Default. Per-request/per-user. |
| `ttl_ms=5000, scope="private"` | Stable per-user lists. |
| `ttl_ms=60000, scope="public"` | Identical for all callers. |
| `cache=None` | Disable entirely. |

Modes: `"use"` (default), `"refresh"`, `"bypass"`

## Extension Contributions

| Server | Client |
|---|---|
| `tools()`, `resources()`, `methods()` | `settings()`, `claims()`, `notifications()` |
| `intercept_tool_call()` | — |
| `settings()` | `settings()` |

## Subscriptions

| Server | Client |
|---|---|
| `ctx.notify_tools_changed()` | `async with client.listen(...)` |
| `ctx.notify_resource_updated(uri)` | Iterate events (cues to refetch) |

## v2 Migration

| v1 | v2 |
|---|---|
| `FastMCP` | `MCPServer` |
| `mcp.server.fastmcp` | `mcp.server.mcpserver` |
| `McpError` | `MCPError` |
| `result.isError` | `result.is_error` |
| `tool.inputSchema` | `tool.input_schema` |
| `httpx.AsyncClient` | `httpx2.AsyncClient` |
| `mcp.get_context()` | `ctx: Context` param |
| `model_dump()` | `model_dump(by_alias=True)` |

## Host Config

| Host | Location | Key |
|---|---|---|
| Claude Desktop | `mcp install server.py` | `mcpServers` |
| Claude Code | `claude mcp add name -- <cmd>` | CLI |
| Cursor | `.cursor/mcp.json` | `mcpServers` |
| VS Code | `.vscode/mcp.json` | `servers` + `type` |

## Imports

| Symbol | Path |
|---|---|
| `MCPServer`, `Context` | `from mcp.server.mcpserver` |
| `Client` | `from mcp` |
| `MCPError` | `from mcp` |
| Types | `from mcp.types` |
| OAuth | `from mcp.client.auth` |
| Transport | `from mcp import streamable_http_client` |

## `raise_exceptions=True`

| Connection | Effect |
|---|---|
| In-memory | Surfaces real errors (not sanitized) |
| `ToolError` | Still `is_error=True` (both modes) |
| HTTP/stdio | No effect |

## Protocol Versions

| Version | Key Features |
|---|---|
| 2026-07-28 | Multi-round-trip, subscriptions, cache hints, extensions |
| 2025-11-25 | Back-channel, subscribe_resource, ping |
| Legacy | Classic initialize handshake |
