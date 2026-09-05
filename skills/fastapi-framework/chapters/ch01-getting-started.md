# Chapter 1: Getting Started

## Core Idea
FastAPI is a modern Python web framework for building APIs with automatic documentation, validation, and type hints. It uses Python type annotations to provide editor support, data validation, and automatic API documentation.

## Frameworks Introduced
- **FastAPI**: The core web framework for building APIs. Use when creating REST APIs with automatic documentation.
- **Pydantic**: Data validation library. Use when defining request/response models with type hints.
- **Uvicorn**: ASGI server. Use when running FastAPI applications in development and production.
- **Starlette**: Underlying toolkit. FastAPI is built on Starlette for routing, requests, and responses.

## Key Concepts
- **Path Operation**: A function decorated with `@app.get()`, `@app.post()`, etc. that handles a specific HTTP request.
- **Automatic Documentation**: FastAPI generates interactive API docs at `/docs` (Swagger UI) and `/redoc` (ReDoc).
- **OpenAPI Schema**: Standard specification for describing APIs. FastAPI auto-generates this from your code.
- **Type Annotations**: Python hints like `int`, `str`, `bool` that FastAPI uses for validation and documentation.

## Mental Models
- **Use FastAPI when building APIs that need automatic documentation and validation**: The type annotations do double duty - they provide editor support AND generate API docs.
- **Think of path operations as functions that receive validated data**: FastAPI extracts data from requests, validates it, and passes it to your function.
- **Use Pydantic models to define data shapes**: Both for request bodies and response models, Pydantic handles validation and serialization.

## Anti-patterns
- **Don't use `fastapi dev` in production**: Use `fastapi run` or Uvicorn directly for production deployments.
- **Don't skip type annotations**: Without them, you lose automatic validation and documentation.
- **Don't store plain passwords**: Always hash passwords before storing them.

## Code Examples
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    result = {"item_id": item_id}
    if q:
        result.update({"q": q})
    return result
```

## Reference Tables

| Component | Purpose | When to Use |
|-----------|---------|-------------|
| `FastAPI()` | Create app instance | Once per application |
| `@app.get()` | Define GET endpoint | For reading data |
| `@app.post()` | Define POST endpoint | For creating data |
| `@app.put()` | Define PUT endpoint | For updating data |
| `@app.delete()` | Define DELETE endpoint | For deleting data |

## Worked Example
Create a simple API with two endpoints:
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str | None = None):
    result = {"item_id": item_id}
    if q:
        result.update({"q": q})
    return result
```
Run with `uv run fastapi dev main.py` and visit `/docs` for interactive documentation.

## Key Takeaways
1. FastAPI uses Python type annotations for validation, serialization, and documentation automatically.
2. The `/docs` endpoint provides interactive API documentation without any extra code.
3. Use `async def` for path operations that perform I/O operations to avoid blocking the event loop.
4. Path parameters are declared in the URL path, query parameters as function parameters.

## Connects To
- **Ch 2**: Path parameters are covered in detail with type conversion and validation.
- **Ch 3**: Query parameters extend path parameters with defaults and validation.
- **Ch 10**: OpenAPI schema is automatically generated from your code.
