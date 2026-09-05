# Chapter 5: Serialization

## Core Idea
Pydantic models serialize to Python dicts and JSON strings with fine-grained control over aliases, exclusions, and custom transformations via `model_dump()`, `model_dump_json()`, and serializer decorators.

## Frameworks Introduced
- **model_dump() / model_dump_json()**: Core serialization methods for converting models to dicts or JSON strings
  - When to use: Any time you need to output model data for APIs, storage, or logging
  - How: Call `model_dump()` for dicts, `model_dump_json()` for JSON; pass `by_alias=True`, `exclude_none=True`, etc.

- **Custom serializers**: `PlainSerializer`, `WrapSerializer`, `@field_serializer`, `@model_serializer` for output transformation
  - When to use: When default serialization doesn't match your output format requirements
  - How: Decorate fields or models with serializer functions that transform values during output

## Key Concepts
- **model_dump()**: Serializes to a Python dict; `mode='python'` (default) or `mode='json'` for JSON-compatible types
- **model_dump_json()**: Serializes directly to a JSON string; more efficient than `json.dumps(model_dump())`
- **by_alias**: When `True`, uses field aliases as keys in output instead of field names
- **exclude_none**: When `True`, omits fields with `None` values from output
- **include/exclude**: Selective field serialization — include only specified fields or exclude specified fields
- **PlainSerializer**: Custom serializer that replaces the default serialization for a type
- **WrapSerializer**: Custom serializer that wraps the default serialization, allowing modification or conditional logic
- **@field_serializer**: Decorator to define custom serialization for a specific field
- **@model_serializer**: Decorator to define custom serialization for the entire model
- **Serialization exclusions**: Runtime exclusions via `model_dump(exclude={'field'})` or per-field `exclude=True`
- **Pickling support**: Pydantic models can be pickled and unpickled; useful for caching and multiprocessing

## Mental Models
- Think of `by_alias=True` as the "external output" mode — use it when serializing for APIs or storage where field names may differ from internal names.
- Treat `exclude_none` as a quick cleanup tool for optional fields that should disappear when empty, reducing payload size.
- Use `include`/`exclude` to create focused views of your model — different consumers may need different subsets of data.

## Anti-patterns
- **Using `model_dump()` then `json.dumps()`**: Use `model_dump_json()` instead — it's faster and handles serialization correctly in one pass
- **Forgetting `by_alias=True` in API responses**: If you use aliases for input, your output should match what external systems expect

## Code Examples
```python
from pydantic import BaseModel, Field

class FooBarModel(BaseModel):
    banana: float | None = 1.1
    foo: str = Field(serialization_alias='foo_alias')
    
m = FooBarModel(banana=3.14, foo='hello')
print(m.model_dump())
#> {'banana': 3.14, 'foo': 'hello'}
print(m.model_dump(by_alias=True))
#> {'banana': 3.14, 'foo_alias': 'hello'}
```
- **What it demonstrates**: Basic serialization with and without aliases

## Worked Example
```python
from pydantic import BaseModel, Field, model_serializer, field_serializer

class User(BaseModel):
    name: str
    password: str = Field(exclude=True)
    age: int | None = None

    @field_serializer('name')
    @classmethod
    def serialize_name(cls, v: str) -> str:
        return v.upper()

# Password excluded by field definition
user = User(name='alice', password='s3cret', age=30)
print(user.model_dump())
#> {'name': 'ALICE', 'age': 30}

# JSON output
print(user.model_dump_json())
#> {"name":"ALICE","age":30}

# Exclude None values
print(user.model_dump(exclude_none=True))
#> {'name': 'ALICE', 'age': 30}

# Selective include
print(user.model_dump(include={'name'}))
#> {'name': 'ALICE'}
```

## Key Takeaways
1. Use `model_dump_json()` over `json.dumps(model_dump())` — it's faster and handles edge cases correctly
2. Set `by_alias=True` when serializing for external systems that expect alias keys
3. Use `exclude=True` on sensitive fields (like passwords) and `exclude_none=True` to keep payloads clean

## Connects To
- **Ch 1 Models**: `model_dump()` and `model_dump_json()` are defined on `BaseModel`
- **Ch 4 Validators**: Validators run on input; serializers run on output — they form a complete data pipeline
- **Ch 2 Fields**: Field aliases and `serialization_alias` control how fields appear in serialized output
