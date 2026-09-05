# Chapter 5: Client Development (Transports, Caching, Callbacks, OAuth)

## Core Idea
A `Client` connects to MCP servers over four transport types (in-memory, HTTP, stdio, SSE), handles OAuth discovery and token management automatically, and receives elicitation/sampling/roots requests through callbacks you register at construction.

## Frameworks Introduced
- **Client**: The high-level client class. `Client(x)` resolves transport by type: server object → in-memory, URL → HTTP, `StdioServerParameters` → subprocess, transport → custom.
  - When to use: Any application that consumes MCP servers.
  - How: `async with Client(mcp) as client:` is the lifecycle. Inside: `list_tools()`, `call_tool()`, `list_resources()`, `read_resource()`, `list_prompts()`, `get_prompt()`, `complete()`.
- **streamable_http_client**: Production HTTP transport. Takes a URL and optional `httpx2.AsyncClient`.
  - When to use: Deployed servers, remote servers, anything over the network.
  - How: `Client("http://...")` wraps it automatically, or build explicitly with `streamable_http_client(url, http_client=...)`.
- **OAuthClientProvider**: OAuth 2.1 token provider for protected servers. It's an `httpx2.Auth` subclass.
  - When to use: When the server requires OAuth tokens.
  - How: Build provider, attach to `httpx2.AsyncClient(auth=provider)`, pass to `streamable_http_client`.
- **ClientSessionGroup**: Holds multiple server connections and merges their tools/resources/prompts into unified dicts.
  - When to use: Applications that talk to multiple MCP servers simultaneously.
  - How: `group = ClientSessionGroup()`, `await group.connect_to_server(params)` per server, `group.call_tool(name, args)`.

## Key Concepts
- **Transport resolution**: `Client(x)` picks the transport by the type of `x`. No separate configuration needed.
- **In-memory transport**: `Client(mcp)` connects to the server object directly. Used for testing and embedding.
- **OAuthClientProvider**: Handles discovery, dynamic registration, PKCE, state/iss validation, and token refresh automatically.
- **TokenStorage**: A Protocol with `get_tokens`/`set_tokens` and `get_client_info`/`set_client_info`. Store `client_info` too, not just tokens.
- **Callbacks**: `elicitation_callback`, `sampling_callback`, `list_roots_callback` handle server-initiated requests. Registering them declares the capability.
- **Session groups**: `ClientSessionGroup` merges multiple servers. Names must be unique across the group; `component_name_hook` rewrites them.

## Mental Models
1. **Client picks transport by type**: A server object → in-memory, a string → HTTP, `StdioServerParameters` → subprocess, anything else → enter as transport.
2. **OAuth is httpx2-level**: The provider goes on the `httpx2.AsyncClient`, not on the `Client`. The MCP layer never sees tokens.
3. **Callbacks declare capabilities**: Registering `elicitation_callback` is how the server learns the client can be asked. There is no separate capability switch.
4. **Raising tool errors**: A tool that raises comes back as `is_error=True` result, not an exception. `call_tool` never raises for tool failures.

## Anti-patterns
- **Calling methods before `async with`**: `Client(mcp)` only builds the object. `async with` opens the connection. Methods before that raise `RuntimeError`.
- **Passing `headers=` to `streamable_http_client`**: It doesn't accept them. Put HTTP config on the `httpx2.AsyncClient` you pass in.
- **Storing only tokens, not client_info**: The provider registers dynamically on first run. Throw away `client_info` and you mint a fresh registration every time.
- **Assuming `is_error=True` raises**: `call_tool` returns a result, never raises for tool failures. Check `result.is_error` explicitly.

## Code Examples
```python
# Basic client
from mcp import Client
from server import mcp

async with Client(mcp) as client:
    tools = await client.list_tools()
    result = await client.call_tool("add", {"a": 1, "b": 2})
    print(result.content)           # [TextContent(text='3')]
    print(result.structured_content) # {'result': 3}
    print(result.is_error)           # False

# Streamable HTTP
async with Client("http://localhost:8000/mcp") as client:
    ...

# Stdio
from mcp import StdioServerParameters
params = StdioServerParameters(command="uv", args=["run", "mcp", "run", "server.py"])
async with Client(params) as client:
    ...

# With OAuth
from mcp.client.auth import OAuthClientProvider
provider = OAuthClientProvider(server_url=..., client_metadata=..., storage=..., ...)
http_client = httpx2.AsyncClient(auth=provider)
transport = streamable_http_client("http://...", http_client=http_client)
async with Client(transport) as client:
    ...
```

## Worked Example
Using a session group to talk to two servers:
```python
from mcp.client.session_group import ClientSessionGroup
from mcp import StdioServerParameters

group = ClientSessionGroup()
await group.connect_to_server(
    StdioServerParameters(command="uv", args=["run", "mcp", "run", "search_server.py"])
)
await group.connect_to_server(
    StdioServerParameters(command="uv", args=["run", "mcp", "run", "db_server.py"])
)

print(sorted(group.tools))  # ['Search.search', 'DB.search']
result = await group.call_tool("Search.search", {"query": "dune"})
```
The `component_name_hook` prefixes tool names with the server name to avoid collisions.

## Key Takeaways
1. `Client(x)` resolves transport by type. In-memory for testing, HTTP for production, stdio for subprocesses.
2. OAuth is httpx2-level: build `OAuthClientProvider`, attach to `httpx2.AsyncClient`, pass to the transport. Discovery and token refresh are automatic.
3. Registering callbacks (`elicitation_callback`, etc.) declares capabilities. The server sees them and knows it can ask.
4. `call_tool` never raises for tool failures. A raising tool returns `is_error=True` with the message in `content`. Check the flag.

## Connects To
- **Ch 1**: Server basics these clients connect to
- **Ch 3**: Elicitation and sampling callbacks these clients handle
- **Ch 6**: Server transports these clients connect over
