# Chapter 1: Models

## Core Idea
Pydantic models are classes inheriting from `BaseModel` with annotated fields that provide runtime validation and serialization via Python's type hint system.

## Frameworks Introduced
- **BaseModel**: The core building block for defining data structures with automatic validation
  - When to use: Any time you need to validate, serialize, or deserialize structured data
  - How: Define fields as annotated class attributes, instantiate with data, call `model_dump()` or `model_dump_json()` for output

- **RootModel**: Wrapper for arbitrary types at the root level when you don't need named fields
  - When to use: Validating lists, dicts, or union types at the model root
  - How: Inherit from `RootModel[YourType]` and access data via `root` attribute

## Key Concepts
- **BaseModel**: Core building block; define fields as annotated class attributes with automatic validation on instantiation
- **model_validate()**: Class method that validates a dict or model instance against the model schema
- **model_validate_json()**: Class method that parses and validates a JSON string against the model schema
- **model_dump()**: Serializes the model to a Python dict, with optional alias and exclusion support
- **model_dump_json()**: Serializes the model directly to a JSON string
- **model_construct()**: Creates a model instance WITHOUT validation; use only with trusted pre-validated data
- **model_rebuild()**: Rebuilds the model's schema to resolve forward references
- **model_fields**: Dict of the model's field information
- **model_extra**: Dict of extra fields if `extra='allow'` is set
- **model_fields_set**: Set of field names that were explicitly set during initialization
- **Data conversion**: Pydantic coerces types automatically (e.g., `int` to `float`, `str` to `int`)
- **Extra data**: Controlled by `ConfigDict(extra='ignore'|'forbid'|'allow')`
- **Nested models**: Use other models as field types to create hierarchical data structures

## Mental Models
- Use `BaseModel` when you need a structured data container with validation; think of it as a typed dict that enforces its schema at runtime.
- Use `model_construct()` only when you have pre-validated data and want to skip validation overhead; treat it as an escape hatch, not the norm.
- Think of `model_validate()` as a bridge between untrusted external data and your validated model instances.

## Anti-patterns
- **Using `model_construct()` with untrusted data**: Bypasses all validation; schema violations or malicious data will propagate silently into your application
- **Ignoring extra fields**: Without `extra='ignore'` or `extra='forbid'`, extra fields are silently discarded, hiding potential data issues

## Code Examples
```python
from pydantic import BaseModel, ConfigDict

class User(BaseModel):
    id: int
    name: str = 'Jane Doe'
    model_config = ConfigDict(str_max_length=10)

user = User(id='123')  # string coerced to int
assert user.id == 123
assert user.model_dump() == {'id': 123, 'name': 'Jane Doe'}
```
- **What it demonstrates**: Type coercion and automatic default values in BaseModel

## Worked Example
```python
from pydantic import BaseModel, ConfigDict

class Address(BaseModel):
    street: str
    city: str

class User(BaseModel):
    id: int
    name: str = 'Jane Doe'
    address: Address | None = None
    model_config = ConfigDict(str_max_length=10)

# Create from dict with nested model
user = User.model_validate({
    'id': '123',
    'name': 'Alice',
    'address': {'street': '123 Main St', 'city': 'Springfield'}
})

# Serialize to JSON
json_str = user.model_dump_json()
print(json_str)
#> {"id":123,"name":"Alice","address":{"street":"123 Main St","city":"Springfield"}}

# Reconstruct from JSON
user2 = User.model_validate_json(json_str)
assert user.id == user2.id
```

## Key Takeaways
1. Always use annotated fields on `BaseModel` for automatic validation — this is Pydantic's core value
2. Use `model_dump()` and `model_dump_json()` for serialization; use `by_alias=True` when outputting to external systems
3. Avoid `model_construct()` unless you're certain the data is pre-validated; it trades safety for speed

## Connects To
- **Ch 2 Fields**: Fields chapter builds on models by showing how to customize individual field behavior
- **Ch 4 Validators**: Validators chapter adds custom validation logic beyond type constraints
- **JSON Schema**: Pydantic models auto-generate JSON Schema; useful for API documentation and code generation
