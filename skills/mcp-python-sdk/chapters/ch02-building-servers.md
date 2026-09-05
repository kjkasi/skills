# Chapter 2: Building Servers (Tools, Resources, Prompts)

## Core Idea
An MCP server exposes three primitives—tools (model-controlled actions), resources (application-controlled data), and prompts (user-controlled templates)—each registered with a single decorator that derives name, description, and schema from the decorated function.

## Frameworks Introduced
- **@mcp.tool()**: Registers a function as a tool the model can call. Type hints become input schema, return type becomes output schema.
  - When to use: Any function the model should be able to invoke to take an action.
  - How: Decorate a function; constraints via `Annotated[..., Field(...)]`; `async def` for I/O; `def` runs in a thread.
- **@mcp.resource(uri)**: Registers a function as a read-only data source addressed by URI.
  - When to use: Data the application loads into context—config files, database records, API responses.
  - How: Decorate a function with a URI; `{placeholder}` in URI makes it a template; `str` returns text, `bytes` returns base64 blob, anything else becomes JSON.
- **@mcp.prompt()**: Registers a function as a user-invokable message template.
  - When to use: Reusable message templates the user picks from a menu or slash command.
  - How: Decorate a function; return a `str` for one user message, or `list[UserMessage | AssistantMessage]` for multi-turn.

## Key Concepts
- **Input schema**: JSON Schema auto-generated from function type hints. Required arguments have no default; optional arguments have a default value.
- **Output schema**: Declared by the return type annotation. Published as `output_schema` in `tools/list`.
- **Resource template**: A URI with `{placeholder}` parameters, listed under `resources/templates/list` instead of `resources/list`.
- **Tool annotations**: Behavioral hints like `read_only_hint`, `destructive_hint`, `idempotent_hint`, `open_world_hint` for client UI decisions.
- **Structured output**: Return types (`int`, `BaseModel`, `TypedDict`, `dataclass`, `dict`) produce typed `structured_content` alongside text `content`.
- **Completion handler**: Optional `@mcp.completion()` for server-side autocomplete of prompt and resource-template arguments.

## Mental Models
1. **Resource = GET, Tool = POST**: If you've built web APIs, resources are read-only data loads, tools are actions with side effects. Prompts have no HTTP analogue.
2. **The SDK reads the function**: Name from function name, description from docstring, schema from type hints. You never declare these separately.
3. **Two channels for tool results**: `content` is text for the model; `structured_content` is typed data for the application. Both are filled from one return value.
4. **Bad arguments fail before your code**: The SDK validates against the schema before calling your function. `Field(ge=1, le=50)` gives you self-correcting agents for free.

## Anti-patterns
- **Returning error strings instead of raising**: A returned string has `is_error=False`—the model thinks it worked. Use `raise ToolError(message)` instead.
- **Overriding type hints with manual schemas**: The SDK already generates the schema. Only use `Annotated[..., Field(...)]` for descriptions and constraints.
- **Using `@mcp.resource()` for actions**: Resources are read-only. Use `@mcp.tool()` for anything with side effects.
- **Forgetting `mime_type` on resource returns**: The SDK defaults to `text/plain`. A `dict` resource without a label is still advertised as plain text.

## Code Examples
```python
# tools.md example
from mcp.server import MCPServer
from mcp.server.mcpserver.prompts.base import UserMessage, AssistantMessage
from typing import Annotated
from pydantic import Field

mcp = MCPServer("Bookshop")

@mcp.tool()
def search_books(
    query: str,
    limit: Annotated[int, Field(ge=1, le=50)] = 10,
) -> str:
    """Search the catalog by title or author."""
    return f"Found 3 books matching {query!r} (showing up to {limit})."

@mcp.resource("catalog://genres")
def get_genres() -> str:
    """The available genres."""
    return "fiction, non-fiction, poetry"

@mcp.prompt()
def review_code(code: str, language: str = "python") -> str:
    """Review a piece of code."""
    return f"Please review this {language} code:\n\n{code}"
```

## Worked Example
A structured output tool returning a Pydantic model:
```python
from pydantic import BaseModel, Field

class WeatherData(BaseModel):
    temperature: float = Field(description="Degrees Celsius.")
    humidity: float = Field(description="Relative humidity, 0 to 1.")
    conditions: str

@mcp.tool()
def get_weather(city: str) -> WeatherData:
    """Get current weather for a city."""
    return WeatherData(temperature=16.2, humidity=0.83, conditions="Overcast")
```
Result: `structured_content` is `{"temperature": 16.2, "humidity": 0.83, "conditions": "Overcast"}`. The model also gets a JSON text version in `content`.

## Key Takeaways
1. Type hints are the contract—both input and output schemas come from them. `Annotated[..., Field(...)]` adds descriptions and constraints.
2. Resources are addressed by URI; `{placeholder}` in the URI creates a template listed separately.
3. `async def` handlers for I/O; plain `def` runs in a thread. The SDK never blocks.
4. Tool annotations (`read_only_hint`, etc.) are hints for client UI, not security controls. Never rely on them.

## Connects To
- **Ch 1**: Installation, first steps, and basic server structure
- **Ch 4**: Completions, structured output, error handling, and URI templates
