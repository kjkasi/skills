# MCP Python SDK Patterns & Design Techniques

## Tool Registration

**When to use**: Exposing a callable function the model can invoke.

**How**: `@mcp.tool()` derives name, description, schema from type hints.

```python
@mcp.tool()
async def search(query: str, limit: int = 10) -> str:
    """Search catalog."""
```

**Trade-offs**: MCPServer auto-generates schemas. Low-level Server lets you hand-write `input_schema` for exact control.

---

## Resource Templating

**When to use**: Data identified by parameterized URI.

**How**: `@mcp.resource("scheme://{param}")` maps URI params to function args.

```python
@mcp.resource("greeting://{name}")
async def greeting(name: str) -> str:
    return f"Hello, {name}!"
```

**Trade-offs**: Templates listed under `list_resource_templates()`. Static resources under `list_resources()`.

---

## Lifespan

**When to use**: Shared state (DB pool, HTTP client) across all requests.

**How**: `lifespan=` with `@asynccontextmanager` yielding one object. Access via `ctx.request_context.lifespan_context`.

```python
@asynccontextmanager
async def app_lifespan(server):
    db = await Database.connect()
    try:
        yield AppContext(db=db)
    finally:
        await db.disconnect()
```

**Trade-offs**: Typed lifespan gives full type-checking. Runs once, not per request.

---

## Dependency Injection

**When to use**: Parameters the model must not supply (prices, identities, confirmations).

**How**: `Annotated[T, Resolve(fn)]` runs fn before tool body. Chains to other resolvers.

```python
@mcp.tool()
async def reserve(title: str, stock: Annotated[Stock, Resolve(check_stock)]) -> str:
    ...
```

**Trade-offs**: Invisible to model/client. Bad graphs fail at registration. Resolvers run once per call.

---

## Elicitation

**When to use**: Tool needs user input mid-call.

**How**: Resolver returns `Elicit(message, Model)` or `ElicitationResult[T]`. Form mode only (flat fields).

```python
async def confirm(path: str) -> ElicitationResult[Confirm]:
    folder = list_folder(path)
    if not folder:
        return Confirm(ok=True)
    return Elicit(f"Delete {len(folder)} files?", Confirm)
```

**Trade-offs**: Resolvers work on all versions. `ctx.elicit()` only on legacy connections. Always handle decline/cancel.

---

## Multi-Round-Trip Requests

**When to use**: Server needs input without opening back-channel (2026-07-28 pattern).

**How**: Return `InputRequiredResult` with `input_requests` and `request_state`. Client retries same call automatically.

**Trade-offs**: Bounded by `input_required_max_rounds` (default 10). State sealed/verified by default. Manual loop possible via `allow_input_required=True`.

---

## Subscriptions

**When to use**: Notifying clients of tool/resource/prompt list changes.

**How**: `ctx.notify_tools_changed()`, `ctx.notify_resource_updated(uri)`. Client opens `client.listen(...)`.

```python
await ctx.notify_resource_updated("board://sprint")
```

**Trade-offs**: No subscribers = no-op. Events are cues to refetch. In-memory bus by default; implement `SubscriptionBus` for multi-process.

---

## Middleware

**When to use**: Timing, logging, tracing, refusing messages, rewriting params.

**How**: `async (ctx, call_next)` appended to `server.middleware`. Outermost-first.

```python
async def timing(ctx, call_next):
    start = time.monotonic()
    try:
        return await call_next(ctx)
    finally:
        logger.info(f"{ctx.method} took {(time.monotonic()-start)*1000:.1f}ms")
```

**Trade-offs**: Provisional API. Good for observe/refuse. Cannot call server-to-client during `initialize`.

---

## Extension

**When to use**: Contributing tools, resources, custom methods, or interceptors behind an identifier.

**How**: Subclass `Extension` with `vendor-prefix/name`. Override `tools()`, `settings()`, `methods()`, `intercept_tool_call()`.

**Trade-offs**: Off by default. Fixed at construction. Cannot replace core behavior. Closed surface is the feature.

---

## Pagination

**When to use**: Thousands of items that can't serialize in one response.

**How**: Low-level Server: return page + `next_cursor=None` for last page. Client loops until `next_cursor is None`.

**Trade-offs**: MCPServer returns everything in one page. Cursors are opaque. Server picks page size.

---

## OAuth Client

**When to use**: Server requires token; client needs discovery, registration, PKCE, refresh.

**How**: `OAuthClientProvider` on `httpx2.AsyncClient(auth=...)` → `streamable_http_client(url, http_client=...)`.

**Trade-offs**: Auto discovery/registration/PKCE. Persist `client_info` + tokens. `ClientCredentialsOAuthProvider` for M2M.

---

## In-Memory Testing

**When to use**: Testing without subprocess/port.

**How**: `Client(mcp, raise_exceptions=True)`. Era-neutral by default.

**Trade-offs**: `raise_exceptions=True` only for in-memory. Pin `mode="legacy"` for legacy-specific semantics.

---

## Session Group

**When to use**: Multiple servers with unified tool/resource/prompt view.

**How**: `ClientSessionGroup()` + `connect_to_server(params)` per server. `component_name_hook` for name uniqueness.

**Trade-offs**: Names must be unique. Uses classic handshake. `connect_with_session` adds existing session.

---

## Request State Protection

**When to use**: Multi-worker/load-balanced where retry can hit different instance.

**How**: `RequestStateSecurity(keys=[shared_key])`. Rotate: add new → promote → retire old.

**Trade-offs**: Default process-local key for single process. Keys ≥32 bytes. Never promote minter before verification rollout.

---

## Structured Output

**When to use**: Tools return typed data for client applications.

**How**: Declare `output_schema`, return matching `structured_content`. MCPServer validates automatically.

**Trade-offs**: Client raises on mismatch. Low-level requires manual schema/result. `content` + `structured_content` travel together.

---

## Caching

**When to use**: Stable lists that don't change per-request.

**How**: `cache_hints={method: CacheHint(ttl_ms=N)}` at server construction. Client honors automatically.

**Trade-offs**: `cacheScope="public"` shares across users. Notifications evict immediately. No stale-if-error. TTL capped 24h.
