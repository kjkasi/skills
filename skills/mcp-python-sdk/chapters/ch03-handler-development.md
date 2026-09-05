# Chapter 3: Handler Development (Context, Dependencies, Elicitation, Lifespan)

## Core Idea
Inside every registered handler, the SDK injects a `Context` object (invisible to the model), resolves dependencies via `Resolve(fn)` before your function runs, and manages server-wide state through a typed lifespan—all without polluting the input schema.

## Frameworks Introduced
- **Context**: Injected into any handler that declares a `ctx: Context` parameter. Provides request ID, headers, session, progress reporting, and resource reading.
  - When to use: Every tool that needs to know about the current request, communicate back to the client, or access server state.
  - How: Add `ctx: Context` parameter. For tools, use `ctx: Context[AppContext]` for typed lifespan access. Resources and prompts use bare `ctx: Context`.
- **Dependencies (Resolve)**: Parameters filled by your own functions instead of the model. `Annotated[T, Resolve(fn)]` runs `fn` before your handler.
  - When to use: Values the model must not invent—prices from your database, user confirmations, permissions.
  - How: `Annotated[Stock, Resolve(check_stock)]`. The resolver can itself have dependencies, use `Context`, or return `Elicit(...)` to ask the user.
- **Lifespan**: An `@asynccontextmanager` that runs once at server startup, yields one object, and cleans up at shutdown.
  - When to use: Database pools, HTTP clients, loaded models—anything built once and shared across requests.
  - How: `MCPServer("Name", lifespan=my_lifespan)`. Access via `ctx.request_context.lifespan_context`.
- **Elicitation**: Asking the user a question mid-tool-call. Two modes: form (structured data) and URL (out-of-band flow like OAuth).
  - When to use: Confirmations, choices, credentials that must not pass through the model.
  - How: Via a resolver returning `Elicit(message, Model)` (works on all connections) or `ctx.elicit()` (legacy connections only).

## Key Concepts
- **Context injection**: The `Context` parameter is invisible to the model—never appears in the input schema.
- **Resolver graph**: Dependencies can depend on each other. The SDK runs each resolver at most once per call, no matter how many consumers declare it.
- **Lifespan context**: The object your lifespan yields is available as `ctx.request_context.lifespan_context`. Type it with `Context[AppContext]` for autocomplete.
- **Elicitation answers**: `"accept"` (user submitted), `"decline"` (user said no), `"cancel"` (user dismissed). `result.data` only exists on accept.
- **Multi-round-trip requests**: On 2026-07-28, the server returns `InputRequiredResult` instead of calling back to the client. `Client` runs the retry loop automatically.

## Mental Models
1. **FastAPI dependency injection**: If you know FastAPI's `Depends`, `Context` and `Resolve` are the same move. Declare what you need, the framework supplies it.
2. **Invisible to the model**: Both `Context` and resolved parameters never appear in the schema. The model only sees your real arguments.
3. **Once per call**: Resolvers run at most once per call, even if multiple tools declare the same dependency. One inventory lookup, two consumers.
4. **Lifespan = startup + shutdown**: Code before `yield` is startup. `finally` after `yield` is shutdown. Runs once, shared by all requests.

## Anti-patterns
- **Using `ctx: Context[AppContext]` in resources or prompts**: Only tools support the type parameter. Resources and prompts must use bare `ctx: Context` or the call fails.
- **Building per-request resources in lifespan**: Lifespan runs once. Per-request resources belong in the handler body.
- **Using `ctx.elicit()` on 2026-07-28**: Server-to-client requests don't exist on modern connections. Use a resolver instead.
- **Ignoring elicitation decline/cancel**: A decline is not an error—your tool decides what it means. But if you can't proceed without the answer, check `result.action` first.

## Code Examples
```python
# lifespan + context + dependency
from contextlib import asynccontextmanager
from mcp.server import MCPServer
from mcp.handlers.context import Context
from mcp.handlers.dependencies import Resolve

class AppContext:
    def __init__(self):
        self.db = Database()

@asynccontextmanager
async def app_lifespan(server):
    ctx = AppContext()
    try:
        yield ctx
    finally:
        await ctx.db.disconnect()

mcp = MCPServer("Bookshop", lifespan=app_lifespan)

@mcp.tool()
async def count_books(ctx: Context[AppContext], genre: str) -> str:
    count = await ctx.db.count(genre)
    return f"{count} {genre} books."

async def check_stock(title: str) -> Stock:
    # Resolves before the tool runs
    ...

@mcp.tool()
async def reserve_book(
    title: str,
    stock: Annotated[Stock, Resolve(check_stock)],
) -> str:
    return f"Reserved {title} ({stock.remaining} left)."
```

## Worked Example
Elicitation via a resolver that asks only when necessary:
```python
async def confirm_delete(path: str) -> Confirm:
    files = list_folder(path)
    if not files:
        return Confirm(ok=True)  # No question needed
    return Elicit(
        message=f"Delete {len(files)} files in {path}?",
        schema=Confirm,
    )

@mcp.tool()
async def delete_folder(
    path: str,
    confirm: Annotated[ElicitationResult[Confirm], Resolve(confirm_delete)],
) -> str:
    match confirm.action:
        case "accept" if confirm.data.ok:
            return do_delete(path)
        case "decline":
            return "Cancelled."
```
The resolver avoids asking when the answer is obvious (empty folder), and the tool handles all three answers.

## Key Takeaways
1. `Context` is injected by type annotation, invisible to the model. Use `Context[AppContext]` in tools for typed lifespan access; bare `Context` in resources/prompts.
2. `Resolve(fn)` runs before your handler. Resolvers can depend on each other, use `Context`, or return `Elicit(...)` to ask the user.
3. Lifespan runs once at startup. Use it for expensive shared resources, not per-request work.
4. On 2026-07-28, use resolvers for elicitation—they work on all connection types. `ctx.elicit()` only works on legacy connections.

## Connects To
- **Ch 2**: Building the handlers these features enhance
- **Ch 4**: Server features that consume context (completions, error handling)
- **Ch 5**: Client-side handling of elicitation callbacks
