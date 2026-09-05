# Chapter 7: Advanced Features (Extensions, Apps, Middleware, Pagination)

## Core Idea
Extensions add opt-in MCP behavior behind one identifier, MCP Apps pair a tool with an HTML UI in a sandboxed iframe, middleware wraps every inbound message for cross-cutting concerns, and pagination handles large result sets through cursors.

## Frameworks Introduced
- **Extensions**: Opt-in bundles of MCP behavior. On a server: contribute tools, resources, methods, and wrap `tools/call`. On a client: claim extra result shapes and observe vendor notifications.
  - When to use: When adding capabilities that not all clients need (e.g., UI rendering via Apps).
  - How: Pass extension instances to `MCPServer(extensions=[...])` at construction. Extensions are fixed—no `add_extension` later.
- **Apps extension (`io.modelcontextprotocol/ui`)**: Built-in extension that pairs a tool with a `ui://` HTML resource.
  - When to use: When tools should have an interactive UI rendered in a sandboxed iframe.
  - How: `apps = Apps()`, `@apps.tool(resource_uri="ui://x/app.html")`, `apps.add_html_resource(uri, html)`.
- **Middleware**: Wraps every inbound message on the low-level `Server`. Deliberately marked provisional.
  - When to use: Cross-cutting concerns like logging, metrics, or request transformation.
  - How: `server = Server("Name", middleware=[my_middleware])`. The middleware sees every message before handlers.
- **Pagination**: Cursor-based navigation for large result sets. Every `list_*` method takes `cursor=`, every result carries `next_cursor`.
  - When to use: When a server has more tools, resources, or prompts than fit in one response.
  - How: Pass `cursor=` to `list_*()`. Loop until `next_cursor is None`.

## Key Concepts
- **Extension contract**: Extensions are off by default. Nothing changes for anyone who didn't opt in. Advertised under `capabilities.extensions`.
- **Apps**: Two parts: a tool (does the work) and a `ui://` resource (HTML the host renders). The tool carries `_meta.ui.resourceUri`. CSP and permissions live on the resource, not the tool.
- **Graceful degradation**: Tools MUST return meaningful `content` even when UI is available. The model reads `content`; the iframe is for humans.
- **Middleware (provisional)**: Wraps every inbound message. Replaces the old private `_handle_*` override pattern.
- **Cursor pagination**: `next_cursor` is `None` when you have everything. `MCPServer` returns everything in one page, so most code never writes the loop.

## Mental Models
1. **Extensions are opt-in bundles**: Nothing changes for clients that don't ask. The capability map advertises what's available.
2. **Apps = tool + UI**: The tool does the work and returns data. The HTML resource is what the human sees. The host renders it in a sandboxed iframe.
3. **Middleware wraps messages**: Think of it as ASGI middleware for MCP—every inbound message passes through before reaching handlers.
4. **Pagination is rare with MCPServer**: The built-in server returns everything in one page. Custom low-level servers may page.

## Anti-patterns
- **Adding extensions after construction**: Extensions are fixed at construction. There is no `add_extension` method.
- **Returning `"[Rendered UI]"` as tool content**: If the fallback text is useless, the tool is useless to text-only clients. Always return meaningful content.
- **Putting CSP on the tool**: CSP and permissions live on the resource, never on the tool. The SDK makes this unrepresentable.
- **Filtering app-only tools server-side**: Filtering is the host's job. Your server lists everything; the host hides what the model shouldn't see.

## Code Examples
```python
# Extensions
from mcp.server import MCPServer
from mcp.server.mcpserver.extensions import Apps

apps = Apps()

@mcp.tool(resource_uri="ui://clock/app.html")
def get_time() -> str:
    """Get the current time."""
    return datetime.now().isoformat()

apps.add_html_resource("ui://clock/app.html", "<html>...</html>")

mcp = MCPServer("Clock", extensions=[apps])

# Pagination
async def list_all_tools(client):
    tools = []
    cursor = None
    while True:
        result = await client.list_tools(cursor=cursor)
        tools.extend(result.tools)
        if result.next_cursor is None:
            break
        cursor = result.next_cursor
    return tools
```

## Worked Example
MCP App with CSP and graceful degradation:
```python
from mcp.server.mcpserver.extensions import Apps, ResourceCsp

apps = Apps()

CLOCK_HTML = """
<html>
<body>
<div id="time"></div>
<script>
window.addEventListener('message', (e) => {
    document.getElementById('time').textContent = e.data;
});
</script>
</body>
</html>
"""

@mcp.tool(resource_uri="ui://clock/app.html", visibility=["app", "model"])
def get_time() -> str:
    """Get the current time."""
    now = datetime.now().isoformat()
    if client_supports_apps(ctx):
        return now  # Both text and UI
    return now  # Text-only fallback

apps.add_html_resource(
    "ui://clock/app.html",
    CLOCK_HTML,
    csp=ResourceCsp(connect_domains=["https://api.example.com"]),
)
```
The tool returns meaningful text for all clients. The UI is an enhancement, not a replacement.

## Key Takeaways
1. Extensions are opt-in bundles registered at construction. Nothing changes for clients that don't ask for them.
2. Apps pair a tool with a `ui://` HTML resource. CSP and permissions go on the resource, not the tool. Always return meaningful text content.
3. Middleware wraps every inbound message on the low-level Server. It's provisional and replaces private `_handle_*` overrides.
4. Pagination uses cursors. `next_cursor is None` means you have everything. `MCPServer` returns everything in one page.

## Connects To
- **Ch 2**: Building the primitives these features extend
- **Ch 4**: Server features (completions, structured output)
- **Ch 8**: Low-level server for custom middleware and pagination
