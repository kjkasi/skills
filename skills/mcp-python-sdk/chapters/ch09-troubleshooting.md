# Chapter 9: Troubleshooting and What's New

## Core Idea
Every SDK error message has a one-move fix. The most common issues are: `ExceptionGroup` hiding the real error (read the last line), `421 Misdirected Request` from missing `transport_security=`, and `requestState` failures from multi-worker key mismatches. v2 brings a rebuilt SDK, protocol 2026-07-28, and several behavior changes that don't produce import errors.

## Frameworks Introduced
- **Troubleshooting reference**: Every error message is the exact text of an SDK error, followed by its meaning and fix.
  - When to use: When you encounter an error not covered by other documentation.
  - How: Use browser find-in-page on the last line of your traceback. Read only that entry.
- **What's New in v2**: The tour of SDK and protocol changes between v1 and v2.
  - When to use: When migrating from v1 or understanding v2 behavior differences.
  - How: Read alongside the Migration Guide for complete before/after code.

## Key Concepts
- **ExceptionGroup wrapping**: Anyio wraps exceptions from `async with Client(...)` in `ExceptionGroup`. Read the **last line** for the real error. Catch inside the block to avoid wrapping.
- **Tool errors are results**: `call_tool` never raises for tool failures. `Error executing tool <name>` is a result with `is_error=True`. Check the flag, not try/except.
- **421 Misdirected Request**: Missing `transport_security=`. The server only accepts localhost by default. Deployed servers need `TransportSecuritySettings(allowed_hosts=[...])`.
- **Task group not initialized**: Mounted ASGI app whose host lifespan didn't enter `mcp.session_manager.run()`.
- **requestState unknown key**: Multi-worker deployment with different sealing keys. Share `RequestStateSecurity(keys=[...])` and the same server name across instances.
- **Cannot send elicitation/create**: `ctx.elicit()` on a 2026-07-28 connection. Use a resolver instead—resolvers work on all connection types.

## Mental Models
1. **Read the last line of ExceptionGroup**: The real error is always at the bottom. Catch inside `async with` to avoid the wrapping.
2. **Tool errors are results, not exceptions**: The model chose the call, so the model gets the message. `is_error=True` is the signal.
3. **421 = missing transport_security**: The most common deployment failure. One setting fixes it.
4. **Server log is the diagnosis source**: Wire errors are deliberately generic ("Invalid or expired requestState"). The server log has the real reason.

## Anti-patterns
- **Catching `call_tool` exceptions**: `call_tool` doesn't raise for tool failures. You're catching nothing.
- **Using `try/except MCPError` around tools**: `MCPError` in a tool becomes a protocol error, not a result. The model never sees it.
- **Ignoring the server log**: Wire errors are generic by design. The server log has the actual reason.
- **Forgetting that `@mcp.tool` without `()` raises at import time**: The error looks like a server crash, not a decorator mistake.

## Code Examples
```python
# Common fixes

# 1. ExceptionGroup - catch inside the block
async with Client(mcp) as client:
    try:
        result = await client.call_tool("search", {"query": "bad"})
    except MCPError as e:
        print(e)  # Clean error, no ExceptionGroup

# 2. Tool errors - check is_error
result = await client.call_tool("search", {"query": "missing"})
if result.is_error:
    print(result.content[0].text)  # Error message the model reads

# 3. 421 fix
mcp = MCPServer(
    "Bookshop",
    transport_security=TransportSecuritySettings(
        allowed_hosts=["mcp.example.com"],
        allowed_origins=["https://app.example.com"],
    ),
)

# 4. requestState across workers
mcp = MCPServer(
    "Bookshop",
    request_state_security=RequestStateSecurity(keys=[shared_key]),
)
```

## Worked Example
Diagnosing a 421 deployment failure:
```python
# Server log shows:
WARNING mcp.server.transport_security: Invalid Host header: mcp.example.com

# Client sees:
mcp.shared.exceptions.MCPError: Server returned an error response

# Fix: add transport_security
from mcp.run.deploy import TransportSecuritySettings

security = TransportSecuritySettings(
    allowed_hosts=["mcp.example.com", "mcp.example.com:*"],
    allowed_origins=["https://app.example.com"],
)
mcp = MCPServer("Bookshop", transport_security=security)
```
The `421` body has no `Content-Type: application/json`, so the python Client can't parse it. The real reason is only in the server's log.

## What's New in v2
- **FastMCP → MCPServer**: Hard rename, not deprecation.
- **First-class Client**: One object replaces three nested layers. `Client(server)` connects in-memory.
- **Resolve for elicitation**: Universal pattern that works on all connection types.
- **Protocol 2026-07-28**: No handshake, no session, no server-initiated requests.
- **httpx → httpx2**: HTTP client dependency changed. TLS uses OS trust store.
- **snake_case fields**: Python attributes are snake_case; wire JSON is still camelCase.
- **Sync handlers run in a thread**: `def` tools no longer block the event loop.
- **Results validated before leaving**: Hand-built schemas are checked against the protocol.
- **URI templates are real RFC 6570**: Stricter matching, path traversal rejected by default.

## Key Takeaways
1. `ExceptionGroup` is anyio noise. Read the last line for the real error. Catch inside `async with` to skip the wrapping.
2. `call_tool` never raises for tool failures. Check `result.is_error`. The model chose the call, so the model gets the message.
3. 421 = missing `transport_security`. One setting: `TransportSecuritySettings(allowed_hosts=[...])`.
4. Multi-worker `requestState` failures = different sealing keys. Share `RequestStateSecurity(keys=[...])` and the same server name.
5. `ctx.elicit()` doesn't work on 2026-07-28. Use `Resolve(fn)` which works on all connections.

## Connects To
- **Ch 1**: Basic setup and common first errors
- **Ch 4**: Error handling hierarchy (ToolError vs MCPError)
- **Ch 6**: Deployment and transport security
- **Ch 8**: Protocol version differences and migration
