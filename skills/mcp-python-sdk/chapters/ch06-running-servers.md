# Chapter 6: Running Servers (ASGI, Authorization, Deployment)

## Core Idea
`mcp.run()` starts a server with one transport choice (stdio, streamable-http, or SSE). For integration with existing ASGI apps, `streamable_http_app()` returns a Starlette application. Authorization is OAuth 2.1 bearer tokens verified by a `TokenVerifier` you implement. Deployment requires a Host allowlist and shared `requestState` keys for multi-worker setups.

## Frameworks Introduced
- **mcp.run()**: The one-liner to start a server. Synchronous, blocks for the server's lifetime.
  - When to use: Standalone servers. Default transport is stdio.
  - How: `mcp.run()` for stdio, `mcp.run("streamable-http")` for HTTP. Place under `if __name__ == "__main__":`.
- **streamable_http_app()**: Returns a Starlette/ASGI application with `/mcp` route.
  - When to use: When the MCP server is part of a larger web app, or you already have an ASGI deployment.
  - How: `app = mcp.streamable_http_app()`. Mount with `Mount("/", app=app)`. Enter `mcp.session_manager.run()` in the host lifespan.
- **TokenVerifier**: Protocol with one async method `verify_token(token) -> AccessToken | None`.
  - When to use: When the server requires OAuth bearer tokens over HTTP.
  - How: Implement `verify_token`, pass `token_verifier=` and `auth=AuthSettings(...)` to `MCPServer`.
- **AuthSettings**: Configuration for the resource server's OAuth metadata.
  - When to use: Together with `token_verifier` to enable authorization.
  - How: `AuthSettings(issuer_url=..., resource_server_url=..., required_scopes=[...])`.
- **TransportSecuritySettings**: Host allowlist for DNS-rebinding protection.
  - When to use: Any deployment behind a real hostname.
  - How: `TransportSecuritySettings(allowed_hosts=["mcp.example.com"], allowed_origins=["https://app.example.com"])`.

## Key Concepts
- **Transport**: How bytes move between server and client. Three options: stdio (default), streamable-http, SSE (deprecated).
- **DNS-rebinding protection**: Out of the box, `streamable_http_app()` only accepts localhost requests. Deploy behind a real hostname requires `transport_security=`.
- **OAuth resource server**: Your server verifies tokens; it never signs anyone in or issues tokens. The authorization server is separate.
- **Protected Resource Metadata**: RFC 9728 document served at `/.well-known/oauth-protected-resource/mcp`. Clients use it to discover the authorization server.
- **requestState**: Opaque token for multi-round-trip calls. `MCPServer` seals it by default. Multi-worker deployments need shared keys and the same server name.
- **Stateless HTTP**: On 2026-07-28, modern requests are stateless—no session, no stickiness. `stateless_http=True` only affects legacy clients.

## Mental Models
1. **stdio = subprocess, HTTP = web service**: stdio servers are launched as child processes. HTTP servers listen on a port like any web service.
2. **Mounting kills the lifespan**: A mounted sub-app's lifespan never runs. The host app must enter `mcp.session_manager.run()`.
3. **Authorization is HTTP-layer**: `stdio` and in-memory clients never see it. Only HTTP transports have `Authorization` headers.
4. **421 Misdirected Request = wrong Host**: Deployed without `transport_security=`, every request is rejected. The reason is only in the server's log.

## Anti-patterns
- **Forgetting `transport_security=` when deploying**: The default localhost-only protection rejects everything behind a real hostname with 421.
- **Not entering `mcp.session_manager.run()` in host lifespan**: Mounted apps don't run their own lifespan. First request fails with "Task group is not initialized".
- **Using `host=` to allowlist**: Passing `host="mcp.example.com"` doesn't add it to the allowlist. Use `transport_security=` explicitly.
- **Building an authorization server in your MCP server**: `auth_server_provider=` predates the spec's AS/RS separation. New servers should not use it.

## Code Examples
```python
# Standalone HTTP server
from mcp.server import MCPServer
mcp = MCPServer("Bookshop")

@mcp.tool()
def search(query: str) -> str:
    return f"Results for {query}"

if __name__ == "__main__":
    mcp.run("streamable-http")

# ASGI integration
from starlette.applications import Starlette
from starlette.routing import Mount

app = mcp.streamable_http_app()

# Custom lifespan with session manager
@asynccontextmanager
async def lifespan(app):
    async with mcp.session_manager.run():
        yield

starlette_app = Starlette(routes=[Mount("/", app=app)], lifespan=lifespan)

# Authorization
from mcp.run.authorization import TokenVerifier, AuthSettings

class MyVerifier(TokenVerifier):
    async def verify_token(self, token):
        # Verify JWT or call introspection endpoint
        ...

mcp = MCPServer(
    "Secure",
    token_verifier=MyVerifier(),
    auth=AuthSettings(
        issuer_url="https://auth.example.com",
        resource_server_url="https://mcp.example.com/mcp",
        required_scopes=["read", "write"],
    ),
)
```

## Worked Example
Deploying with Host allowlist:
```python
from mcp.server import MCPServer
from mcp.run.deploy import TransportSecuritySettings

security = TransportSecuritySettings(
    allowed_hosts=["mcp.example.com", "mcp.example.com:*"],
    allowed_origins=["https://app.example.com"],
)

mcp = MCPServer("Bookshop", transport_security=security)
# Now the server accepts requests from mcp.example.com
```
Without this, `curl https://mcp.example.com/mcp` returns `421 Misdirected Request` and the client sees `Server returned an error response`.

## Key Takeaways
1. `mcp.run()` for standalone servers, `streamable_http_app()` for ASGI integration. The default transport is stdio.
2. Mounting disables the built-in lifespan. The host app must enter `mcp.session_manager.run()`.
3. Deployed servers need `transport_security=` with allowed hosts and origins. Without it, every request is a 421.
4. Authorization is `token_verifier` + `auth=AuthSettings(...)`. The SDK publishes discovery metadata and handles 401 responses automatically.

## Connects To
- **Ch 1**: Basic server structure and `mcp.run()`
- **Ch 5**: Client-side OAuth handling
- **Ch 8**: ASGI app details and advanced deployment
