# Chapter 3: Query Parameters

## Core Idea
Query parameters are key-value pairs in the URL after the `?` character. They're automatically interpreted from function parameters not part of the path, supporting optional values, defaults, and type validation.

## Frameworks Introduced
- **Query**: FastAPI class for adding validation and metadata to query parameters.
- **Pydantic Models**: Use for grouping related query parameters with shared validation.

## Key Concepts
- **Query Parameter**: A function parameter not in the path that becomes part of the URL query string.
- **Default Values**: Optional parameters with default values don't need to be provided by the client.
- **Required Parameters**: Parameters without default values must be provided in the request.
- **Type Conversion**: Query parameters are automatically converted to declared types (e.g., `int`, `bool`).

## Mental Models
- **Use defaults to make parameters optional**: Setting `q: str | None = None` makes `q` optional.
- **Think of query parameters as filters**: They're commonly used for filtering, pagination, and sorting.
- **Use Pydantic models when you have many related parameters**: This reduces code duplication.

## Anti-patterns
- **Don't confuse path and query parameters**: Path parameters are in the URL path, query parameters are after the `?`.
- **Don't forget that query parameters are strings by default**: Always declare their type for proper validation.
- **Don't use query parameters for required data**: Path parameters are better for required identifiers.

## Code Examples
```python
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):
    result = {"item_id": item_id}
    if q:
        result.update({"q": q})
    return result

@app.get("/items/{item_id}")
async def read_item(item_id: str, needy: str, skip: int = 0, limit: int | None = None):
    result = {"item_id": item_id, "needy": needy}
    result.update({"skip": skip, "limit": limit})
    return result
```

## Reference Tables

| Query Parameter | Type | Default | Required | Description |
|----------------|------|---------|----------|-------------|
| `skip` | `int` | `0` | No | Number of items to skip |
| `limit` | `int` | `10` | No | Maximum items to return |
| `q` | `str \| None` | `None` | No | Optional search query |
| `needy` | `str` | None | Yes | Required parameter |

## Worked Example
Create an API with mixed required and optional query parameters:
```python
@app.get("/items/{item_id}")
async def read_item(item_id: str, needy: str, skip: int = 0, limit: int | None = None):
    result = {"item_id": item_id, "needy": needy}
    result.update({"skip": skip, "limit": limit})
    return result
```
- `/items/foo-item?needy=sooooneedy` - Works (required param provided)
- `/items/foo-item` - Error (missing required `needy`)
- `/items/foo-item?needy=sooooneedy&skip=20` - Works with optional skip
- `/items/foo-item?needy=sooooneedy&limit=100` - Works with optional limit

## Key Takeaways
1. Query parameters are optional by default when they have default values.
2. Parameters without default values become required query parameters.
3. Use `str | None` (not just `None` default) to indicate optional parameters for better editor support.
4. Boolean query parameters accept `true`, `True`, `1`, `yes`, `on` as truthy values.

## Connects To
- **Ch 2**: Path parameters are required and in the URL path, unlike query parameters.
- **Ch 4**: Request bodies are for complex data, query parameters for simple filters.
- **Ch 5**: Dependencies can also handle query parameter validation and defaults.
