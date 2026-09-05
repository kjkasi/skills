---
name: mcp-python-sdk
description: "Knowledge base from \"MCP Python SDK\" by Model Context Protocol. Use when building MCP servers/clients, applying MCP patterns, or referencing SDK APIs."
---

<!-- argument-hint: [topic, feature name, or chapter number] -->

# MCP Python SDK
**Author**: Model Context Protocol | **Chapters**: 9 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `tools`, `resources`, `prompts`, `transports`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch01`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### MCP Architecture (Three Primitives)
- **Tools**: Functions the model can call to perform actions or compute results
  - When to use: Any action that changes state or requires computation
  - How: Decorate with `@mcp.tool()`, provide type hints for auto-schema
- **Resources**: Data sources the model can read (files, API responses, templates)
  - When to use: Exposing read-only data or dynamic content
  - How: Decorate with `@mcp.resource("uri://template")` or `@mcp.resource("uri://static")`
- **Prompts**: Reusable prompt templates with arguments
  - When to use: Standardized interactions that need user input
  - How: Decorate with `@mcp.prompt()`, return list of messages

### Server Classes
- **`MCPServer`** (Recommended): High-level API with decorators, automatic schema generation
  - When to use: Most servers, rapid development, clean code
  - How: `mcp = MCPServer("name")`, then `@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()`
- **`Server`**: Low-level API for full control over protocol messages
  - When to use: Custom middleware, complex message handling, extensions
  - How: Subclass and override handlers, register with `server.list_tools()`, `server.call_tool()`

### Client Architecture
- **`Client`**: Manages connection to MCP server, handles session lifecycle
  - When to use: Any client application connecting to MCP servers
  - How: `async with Client(server_or_url) as client: ...`
- **Transports**: Communication layers (stdio, Streamable HTTP, SSE)
  - When to use: Different deployment scenarios
  - How: `StdioServerTransport`, `StreamableHttpServerTransport`, `SseServerTransport`

### Handler Patterns
- **Context**: Request-scoped state and utilities
  - When to use: Accessing request metadata, sending notifications, logging
  - How: `ctx: Context` parameter in handlers, provides `ctx.report_progress()`, `ctx.info()`
- **Dependencies**: Injectable services via `Resolve`
  - When to use: Database connections, API clients, shared state
  - How: `db: Database = Resolve(get_database)`, automatic lifecycle management
- **Lifespan**: Server startup/shutdown hooks
  - When to use: Resource initialization, cleanup, connection pools
  - How: `@mcp lifespan`, yield context, cleanup on exit

### Extension System
- **Extensions**: Opt-in bundles of MCP behavior behind one identifier
  - When to use: Adding capabilities without breaking existing clients
  - How: `MCPServer(extensions=[MyExtension()])`, advertise in `capabilities.extensions`
- **Apps**: UI-bound tools with HTML resources
  - When to use: Interactive tools that render in host UI
  - How: `apps = Apps()`, `@apps.tool(resource_uri="ui://x")`, serve HTML resource

### Security & Auth
- **OAuth 2.1**: Authorization framework for protected resources
  - When to use: Server requires authentication, multi-tenant deployments
  - How: Implement `OAuthAuthorizationServerProvider`, configure in `AuthorizationConfig`
- **Identity Assertion**: Client identity verification
  - When to use: Server needs to know which client is calling
  - How: Implement `IdentityAssertionProvider`, validate in handler

### Deployment Patterns
- **ASGI Integration**: Mount in FastAPI/Starlette apps
  - When to use: Existing ASGI infrastructure, mixed HTTP/MCP endpoints
  - How: `MCPApp(mcp_server).build_asgi_app()`, mount at path
- **Standalone Server**: Direct ASGI/WSGI deployment
  - When to use: Dedicated MCP server, simple deployment
  - How: `uvicorn mcp_server:app`, configure transport

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-getting-started.md) | Getting Started | Installation, Three Primitives, Client Testing |
| [ch02](chapters/ch02-building-servers.md) | Building Servers | Tool Registration, Resource Templating, Prompts |
| [ch03](chapters/ch03-handler-development.md) | Handler Development | Context, Dependencies, Elicitation, Lifespan |
| [ch04](chapters/ch04-server-features.md) | Server Features | Completions, Media, Error Handling, URI Templates |
| [ch05](chapters/ch05-client-development.md) | Client Development | Transports, OAuth, Callbacks, Session Groups |
| [ch06](chapters/ch06-running-servers.md) | Running Servers | ASGI, Authorization, Deployment, Host Config |
| [ch07](chapters/ch07-advanced-features.md) | Advanced Features | Extensions, Apps, Middleware, Pagination |
| [ch08](chapters/ch08-protocol-migration.md) | Protocol & Migration | v1→v2 Migration, 2026-07-28 Protocol |
| [ch09](chapters/ch09-troubleshooting.md) | Troubleshooting | Error Messages, What's New, Diagnostics |

---

## Topic Index

- **Tools** → ch02, ch04
- **Resources** → ch02, ch04
- **Prompts** → ch02
- **Completions** → ch04
- **Transports** → ch05
- **OAuth** → ch05, ch06
- **Session Groups** → ch05
- **ASGI** → ch06
- **Deployment** → ch06
- **Extensions** → ch07
- **Apps** → ch07
- **Middleware** → ch07
- **Pagination** → ch07
- **Context** → ch03
- **Dependencies** → ch03
- **Elicitation** → ch03
- **Lifespan** → ch03
- **Logging** → ch03
- **Progress** → ch03
- **Sampling** → ch03
- **Subscriptions** → ch03, ch05
- **Media** → ch04
- **Error Handling** → ch04
- **URI Templates** → ch04
- **Structured Output** → ch04
- **Migration** → ch08
- **Protocol Versions** → ch08
- **Troubleshooting** → ch09
- **What's New** → ch09

---

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the MCP Python SDK documentation only. For hands-on implementation in your codebase,
combine with project-specific tools. For topics beyond this SDK, check related skills
or ask the agent directly.
