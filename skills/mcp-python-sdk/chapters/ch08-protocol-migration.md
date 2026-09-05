# Chapter 8: Protocol Migration

## Core Idea
v2 is a major version: the SDK was rebuilt and the protocol moved to 2026-07-28, which removes the connection handshake, the session, and every server-initiated request. The same server serves both eras simultaneously, and `Resolve` replaces server-initiated elicitation as the cross-era pattern.

## Frameworks Introduced
- **Migration Guide**: The comprehensive before/after reference for every breaking change.
  - When to use: When porting v1 code to v2.
  - How: Follow the import table, rename patterns, and behavior changes section by section.
- **Protocol versions**: The negotiation mechanism between client and server. 2026-07-28 is the new default; 2025-11-25 is legacy.
  - When to use: When you need to serve both old and new clients, or control which version is negotiated.
  - How: `Client` probes with `server/discover` and falls back to `initialize`. `mode="legacy"` forces the old handshake.

## Key Concepts
- **FastMCP → MCPServer**: The high-level server class was renamed. Import path changed from `mcp.server.fastmcp` to `mcp.server.mcpserver`.
- **snake_case fields**: Every Python attribute is now snake_case (`result.is_error`, `tool.input_schema`). Wire JSON is still camelCase.
- **No handshake, no session (2026-07-28)**: Every request carries its protocol version in `_meta`. No `Mcp-Session-Id` on the modern path.
- **Server cannot call the client**: Push elicitation, sampling, and roots/list are gone at 2026-07-28. The server returns `InputRequiredResult` instead.
- **Resolve replaces server-initiated elicitation**: One pattern works on both eras: `Resolve(fn)` where `fn` can return `Elicit(...)`.
- **Client is first-class**: One object (`Client`) replaces the three nested layers (transport, session, initialize). `Client(server)` connects in-memory.
- **httpx replaces httpx2**: The HTTP client dependency changed. TLS verification uses OS trust store via `truststore`.

## Mental Models
1. **Same server, both eras**: `MCPServer` serves 2025-11-25 and 2026-07-28 simultaneously. No flag to flip, no separate deployment.
2. **Resolve is the universal pattern**: Whether the client is legacy or modern, `Resolve(fn)` asks via the mechanism the connection supports.
3. **v2 renames are not optional**: `FastMCP` → `MCPServer` and `mcp.server.fastmcp` → `mcp.server.mcpserver` are hard renames, not deprecations.
4. **Protocol types in mcp-types**: Wire types moved to a standalone package. `import mcp.types` still works as a permanent alias.

## Anti-patterns
- **Using `ctx.elicit()` for new code**: Server-to-client requests don't work on 2026-07-28. Use `Resolve(fn)` instead.
- **Calling `ctx.session.create_message()` on modern connections**: It fails with `NoBackChannelError`. Use `Sample(...)` in a resolver.
- **Ignoring the protocol version**: A pre-2026 session has no back-channel. Server-initiated features need the right mechanism for the era.
- **Forgetting that MCPError is now a protocol error**: In v1, any exception became `is_error=True`. In v2, only `ToolError` does. `MCPError` propagates as a JSON-RPC error.

## Code Examples
```python
# v1 (before)
from mcp.server.fastmcp import FastMCP
mcp = FastMCP("Demo")

@mcp.tool()
async def search(query: str) -> str:
    return f"Results for {query}"

# v2 (after)
from mcp.server import MCPServer
mcp = MCPServer("Demo")

@mcp.tool()
async def search(query: str) -> str:
    return f"Results for {query}"

# v1 elicitation (deprecated on 2026)
@mcp.tool()
async def book_table(date: str, ctx: Context) -> str:
    result = await ctx.elicit("Try another date?", schema=AlternativeDate)
    ...

# v2 elicitation (works everywhere)
async def ask_date(date: str) -> AlternativeDate:
    return Elicit(message="Try another date?", schema=AlternativeDate)

@mcp.tool()
async def book_table(
    date: str,
    alt: Annotated[AlternativeDate, Resolve(ask_date)],
) -> str:
    ...
```

## Worked Example
Serving both eras from one server:
```python
from mcp.server import MCPServer
from mcp.handlers.context import Context
from mcp.handlers.dependencies import Resolve, Elicit

mcp = MCPServer("DualEra")

async def confirm_delete(path: str) -> Confirm:
    # Works on both legacy (elicitation) and modern (multi-round-trip)
    files = list_folder(path)
    if not files:
        return Confirm(ok=True)
    return Elicit(message=f"Delete {len(files)} files?", schema=Confirm)

@mcp.tool()
async def delete_folder(
    path: str,
    confirm: Annotated[ElicitationResult[Confirm], Resolve(confirm_delete)],
) -> str:
    match confirm.action:
        case "accept" if confirm.data.ok:
            return do_delete(path)
        case _:
            return "Cancelled."
```
The same code serves legacy clients (elicitation request) and modern clients (multi-round-trip return).

## Key Takeaways
1. v2 is a major version with hard renames: `FastMCP` → `MCPServer`, snake_case fields, httpx → httpx2.
2. 2026-07-28 removes handshake, sessions, and server-initiated requests. The server returns questions instead of calling back.
3. `Resolve(fn)` is the universal pattern for mid-call user input. It works on both legacy and modern connections.
4. The same `MCPServer` serves both protocol eras. `Client` probes and falls back automatically.

## Connects To
- **Ch 3**: Context, dependencies, and elicitation (the new patterns)
- **Ch 4**: Error handling changes (MCPError is now protocol-level)
- **Ch 9**: Troubleshooting migration issues
