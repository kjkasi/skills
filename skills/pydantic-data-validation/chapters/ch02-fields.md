# Chapter 2: Fields

## Core Idea
Fields are customized via the `Field()` function or the `Annotated` pattern to control defaults, constraints, aliases, and metadata for individual model attributes.

## Frameworks Introduced
- **Field()**: Function that configures field behavior including defaults, constraints, and aliases
  - When to use: Any time you need more control than a bare type annotation provides
  - How: Pass parameters like `default`, `alias`, `gt`, `min_length` to `Field()`

- **Annotated pattern**: `Annotated[type, Field(...)]` — the preferred type-safe approach for field configuration
  - When to use: When you want reusable type aliases with built-in constraints
  - How: Define `PositiveInt = Annotated[int, Field(gt=0)]` and use it as a type hint

## Key Concepts
- **Field()**: Function to set defaults, constraints, aliases, and JSON schema metadata on model fields
- **Annotated pattern**: `Annotated[type, Field(...)]` — preferred for type safety and reusable constrained types
- **Default values**: Set via regular assignment or `Field(default=...)`
- **default_factory**: Callable for mutable defaults; can accept the validated data dict as input
- **validate_default**: When `True`, validates the default value itself against field constraints
- **Field aliases**: `alias` for both validation and serialization; `validation_alias` for input only; `serialization_alias` for output only
- **AliasPath / AliasChoices**: Support for nested alias lookups when validating against complex input structures
- **Constraints**: `gt`, `ge`, `lt`, `le`, `min_length`, `max_length`, `pattern`, `multiple_of`, etc.
- **Computed fields**: `@computed_field` decorator for derived values computed from other fields
- **Private attributes**: `PrivateAttr` for fields that exist on the instance but are not part of the schema

## Mental Models
- Use `Annotated` when you want a reusable constrained type (like `PositiveInt`) that works anywhere a type hint is accepted.
- Think of `default_factory` as a lazy initializer — it runs once per instance, not once at class definition time, preventing shared mutable state bugs.
- Treat `validation_alias` and `serialization_alias` as separate channels: validation aliases accept external data formats, serialization aliases output in your system's preferred format.

## Anti-patterns
- **Using mutable defaults directly**: Never write `field: list = []` — use `default_factory=list` to avoid sharing a single list across all instances
- **Mixing alias and validation_alias carelessly**: If both are set, `validation_alias` takes precedence for input; ensure you don't accidentally break deserialization

## Code Examples
```python
from typing import Annotated
from pydantic import BaseModel, Field

class Model(BaseModel):
    name: Annotated[str, Field(strict=True, min_length=1)]
    age: int = Field(ge=0, default=0)
```
- **What it demonstrates**: Strict type checking with min_length constraint and a constrained default value

## Worked Example
```python
from typing import Annotated
from pydantic import BaseModel, Field, AliasPath, AliasChoices

class User(BaseModel):
    name: str = Field(validation_alias=AliasChoices('name', AliasPath('data', 'name')))
    age: int = Field(ge=0, default=0)
    tags: list[str] = Field(default_factory=list)

# Validate from nested alias
user = User.model_validate({'data': {'name': 'Alice'}, 'age': 30})
assert user.name == 'Alice'

# Validation catches constraint violations
try:
    User(name='Bob', age=-1)
except Exception as e:
    print(e)
    #> 1 validation error for User
    #> age
    #>   Input should be greater than or equal to 0 [type=greater_than_equal, ...]
```

## Key Takeaways
1. Prefer the `Annotated` pattern over bare `Field()` for type safety and reusability of constrained types
2. Always use `default_factory` for mutable defaults (list, dict, set) to avoid shared state bugs
3. Use `validation_alias` and `serialization_alias` separately when input and output formats differ

## Connects To
- **Ch 1 Models**: Fields are the building blocks of models; this chapter shows how to customize each one
- **Ch 3 Types**: Type constraints interact with field constraints; understanding both is necessary for correct validation
- **Ch 4 Validators**: Custom validators complement field constraints for complex business logic
