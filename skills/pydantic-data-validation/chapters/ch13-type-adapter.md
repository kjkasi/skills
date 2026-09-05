# Chapter 13: TypeAdapter

## Core Idea
TypeAdapter lets you validate, serialize, and generate schemas for any Python type — not just models — making it the go-to tool for standalone validation.

## Frameworks Introduced
- **Standalone Validation**: Validate arbitrary types without defining a model.
  - When to use: Validating primitives, collections, unions, or custom types outside a model context.
  - How: Create `TypeAdapter(MyType)` and call `.validate_python()` or `.validate_json()`.
- **Schema Generation for Any Type**: Generate JSON Schema from any type.
  - When to use: API documentation, config file validation, tooling.
  - How: Call `.json_schema()` on a TypeAdapter instance.

## Key Concepts
- **`TypeAdapter`**: Wraps any type and exposes `validate_python()`, `validate_json()`, `dump_python()`, `dump_json()`, and `json_schema()`.
- **`validate_python()`**: Validates a Python object against the wrapped type, coercing as needed.
- **`validate_json()`**: Parses and validates a JSON string directly — faster than `json.loads` + `validate_python`.
- **`dump_python()` / `dump_json()`**: Serialize an object according to the type's serialization schema.
- **`json_schema()`**: Returns a JSON Schema dict for the wrapped type.

## Mental Models
1. **Type Wrapper**: TypeAdapter is a lightweight wrapper that applies Pydantic's validation engine to any type.
2. **Model-Free Validation**: When a full model is overkill (e.g., validating a `list[str]`), TypeAdapter is the right tool.
3. **Unified Serialization**: Same `dump_python`/`dump_json` API as models, but for raw types.

## Anti-patterns
- **Using TypeAdapter for complex nested structures**: If you need multiple fields with relationships, a BaseModel is clearer.
- **Ignoring `validate_json` performance**: Parsing JSON separately then validating is slower than `validate_json` directly.

## Code Examples
```python
from pydantic import TypeAdapter

ta = TypeAdapter(list[int])
result = ta.validate_python([1, '2', 3])
print(result)  # [1, 2, 3]

schema = ta.json_schema()
```
- **What it demonstrates**: Validating a collection type and generating its JSON Schema.

## Worked Example
Validate an API query parameter that accepts a comma-separated list of IDs:

```python
from pydantic import TypeAdapter

ta = TypeAdapter(list[int])

# From a query string like "1,2,3"
raw = "1,2,3"
ids = ta.validate_python(raw.split(','))
print(ids)  # [1, 2, 3]

# From JSON
ids = ta.validate_json('[1, "2", 3]')
print(ids)  # [1, 2, 3]
```

For a union type:

```python
from pydantic import TypeAdapter

ta = TypeAdapter(int | str)
print(ta.validate_python("42"))    # 42
print(ta.validate_python(42))      # 42
```

## Key Takeaways
1. TypeAdapter validates any type, not just model classes.
2. `validate_json()` is faster than parsing JSON then validating Python objects.
3. Supports the same config options as BaseModel for strict/coerce behavior.
4. `json_schema()` generates schemas for any type, useful for documentation and tooling.

## Connects To
- **Ch 4**: BaseModel uses TypeAdapter internally for field validation.
- **Ch 11**: `validate_json` is a key performance optimization.
- **Ch 12**: TypeAdapter is how you validate stdlib dataclasses.
- **Ch 15**: TypeAdapter raises the same ValidationError with the same structure.
