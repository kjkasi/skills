# Chapter 4: Server Features (Completions, Media, Error Handling, URI Templates, Structured Output)

## Core Idea
Beyond the three primitives, servers can autocomplete arguments (completions), return binary content (media), handle failures at two levels (tool errors vs protocol errors), address resources with RFC 6570 templates, and produce typed structured output—all driven by the same decorator-and-type-hint philosophy.

## Frameworks Introduced
- **@mcp.completion()**: One handler per server for autocomplete of prompt arguments and resource template parameters.
  - When to use: When your server has prompts or resource templates with arguments that benefit from suggestions.
  - How: `async def handler(ref, argument, context) -> Completion | None`. Branch on `isinstance(ref, PromptReference)` and `argument.name`.
- **Image / Audio helpers**: SDK convenience types for returning binary content from tools.
  - When to use: When a tool should return images, audio, or embedded resources instead of text.
  - How: `return Image(path="logo.png")` or `return Audio(data=wav_bytes, format="audio/wav")`.
- **ToolError / MCPError / ResourceNotFoundError**: The error hierarchy for tools and resources.
  - When to use: `ToolError` for recoverable failures (model can retry), `MCPError` for protocol-level rejections.
  - How: `raise ToolError("message")` in tools; `raise ResourceNotFoundError(uri)` in resources.
- **URI templates (RFC 6570)**: Parameterized addressing for resources with `{placeholder}` syntax.
  - When to use: When one function serves multiple URIs with parameters (e.g., `users://{id}/profile`).
  - How: `@mcp.resource("users://{user_id}/profile")` with matching function parameter name.

## Key Concepts
- **Completions**: Suggestions for prompt/resource-template arguments. `None` means no suggestions; the SDK turns it into an empty list.
- **ToolError**: Returns `is_error=True` with message in `content`. The model reads it and can retry. Logged at INFO.
- **MCPError**: Protocol error. The call fails with JSON-RPC error. The model sees nothing. Logged at ERROR.
- **ResourceNotFoundError**: Protocol error `-32602` with URI in `data`. For missing resources.
- **URI template operators**: `{name}` simple expansion, `{+path}` reserved expansion, `{?q}` query expansion, `{#fragment}` fragment expansion.
- **Path safety**: SDK rejects URI-extracted values that would resolve outside the intended directory by default.
- **Structured output**: Return type annotation IS the output schema. Scalars wrap in `{"result": ...}`. Models/dicts stay as objects.

## Mental Models
1. **Could a smarter model have avoided this?** Yes → `ToolError`. No → `MCPError`. This one question decides which exception to raise.
2. **The return type IS the schema**: No separate output schema declaration. `-> int` means `{"result": {"type": "integer"}}`. `-> WeatherData` means the model's fields are the schema.
3. **Two channels, one value**: `content` is text for the model, `structured_content` is typed data for the application. The SDK fills both from your return value.
4. **URI templates are addresses, not names**: Clients ask for `config://app`, never for `get_config`. The URI is the identity.

## Anti-patterns
- **Returning error messages instead of raising**: `return "error: not found"` has `is_error=False`—the model thinks it worked. Always `raise ToolError(...)`.
- **Using MCPError for recoverable failures**: The model never sees MCPError messages. Save it for things the model can't fix by retrying.
- **Using `structured_output=False` without reason**: Content blocks (`TextContent`, `Image`, `Audio`) opt out automatically. Only use it when you explicitly don't want typed output.
- **Forgetting `format=` with `data=` on media**: Without a filename to guess from, the SDK defaults to `image/png` or `audio/wav`. MP3 bytes labeled as WAV will fail to decode.

## Code Examples
```python
# Error handling: ToolError vs MCPError
from mcp.server.mcpserver.exceptions import ToolError
from mcp import MCPError

@mcp.tool()
async def get_author(title: str) -> str:
    """Get the author of a book."""
    book = CATALOG.get(title)
    if not book:
        raise ToolError(f"No book titled {title!r} in the catalog.")
    return book.author

@mcp.tool()
async def admin_action(key: str) -> str:
    """Admin-only action."""
    if key != ADMIN_KEY:
        raise MCPError(code=-32602, message="Invalid admin key.")
    return "done"

# Structured output
from pydantic import BaseModel

class Weather(BaseModel):
    temperature: float
    conditions: str

@mcp.tool()
def get_weather(city: str) -> Weather:
    """Get weather for a city."""
    return Weather(temperature=16.2, conditions="Overcast")

# Completions
@mcp.completion()
async def complete(ref, argument, context):
    if isinstance(ref, PromptReference) and ref.name == "review_code":
        if argument.name == "language":
            langs = ["python", "go", "rust", "typescript"]
            return Completion(values=[l for l in langs if l.startswith(argument.value)])
    return None
```

## Worked Example
Resource with URI template and path safety:
```python
@mcp.resource("books://{title}")
async def get_book(title: str) -> str:
    """Get a book's details by title."""
    book = CATALOG.get(title)
    if not book:
        raise ResourceNotFoundError(f"No book titled {title!r}")
    return f"{book.title} by {book.author} ({book.year})"
```
The SDK matches `books://dune` to the template, extracts `title="dune"`, and validates the URI path won't escape the intended scope.

## Key Takeaways
1. `ToolError` for model-recoverable failures (`is_error=True`); `MCPError` for protocol rejections (JSON-RPC error). The deciding question: could a smarter model have avoided this?
2. Return type annotations produce the output schema. `-> int` wraps in `{"result": ...}`. `-> BaseModel` keeps the model's structure.
3. `@mcp.completion()` is the one handler per server. Branch on `isinstance(ref, ...)` and `argument.name`. `None` means no suggestions.
4. URI templates use RFC 6570 syntax. Path safety rejects traversal by default. Template parameter names must match function parameter names.

## Connects To
- **Ch 2**: Building the primitives these features extend
- **Ch 3**: Context and dependencies that interact with error handling
- **Ch 7**: Low-level server for custom error handling
