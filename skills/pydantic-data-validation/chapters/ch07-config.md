# Chapter 7: Config

## Core Idea
Pydantic models are configured via `ConfigDict`, a typed dictionary that controls validation behavior, field access, immutability, and interoperability — all declared in one place.

## Frameworks Introduced
- **ConfigDict Configuration**: Centralized model behavior control
  - When to use: Any model needing non-default validation, serialization, or access behavior
  - How: Set `model_config = ConfigDict(...)` as a class attribute on any Pydantic model
- **Type Adapter Config**: Apply config to non-model types
  - When to use: Validating TypedDict, dataclasses, or arbitrary types via `TypeAdapter`
  - How: Pass `config=ConfigDict(...)` to `TypeAdapter` constructor

## Key Concepts
- **ConfigDict**: Typed dict with all configuration options; type-safe and IDE-friendly
- **extra**: Controls extra fields — `'ignore'` (drop), `'forbid'` (raise error), `'allow'` (include)
- **frozen**: Makes model instances immutable; assignment raises error
- **validate_default**: Validate default values on instantiation, not just provided values
- **from_attributes**: Validate from object attributes (e.g., ORM instances) instead of dict keys
- **validate_by_name**: Allow validation by both field name and alias
- **alias_generator**: Callable that generates aliases from field names
- **arbitrary_types_allowed**: Permit fields with non-Pydantic types
- **revalidate_instances**: Control whether already-validated model instances are re-validated
- **str_max_length / str_min_length**: Global string length constraints

## Mental Models
1. **Config as Policy**: Think of ConfigDict as your model's policy document — it declares what's allowed
2. **Defaults Are Sensible**: Most config options have safe defaults; only override when you need specific behavior
3. **Freezing = Value Object**: A frozen model is like a value object in DDD — immutable and identity-free
4. **from_attributes = Adapter Pattern**: ORM objects become Pydantic models without manual mapping

## Anti-patterns
- **Overriding config per-field when model-level suffices**: Use `field=Field(...)` only for per-field exceptions
- **Using `extra='allow'` for flexibility**: Leads to silent data loss; prefer explicit fields or `extra='forbid'`
- **Mixing deprecated `populate_by_name` with `validate_by_name`**: Always use `validate_by_name`; `populate_by_name` is deprecated

## Code Examples
```python
from pydantic import BaseModel, ConfigDict

class Model(BaseModel):
    model_config = ConfigDict(
        str_max_length=10,
        validate_default=True,
        frozen=True,
        extra='forbid',
    )
    name: str = 'default'
```
- **What it demonstrates**: Comprehensive model configuration with common options

```python
from pydantic import BaseModel, ConfigDict

class UserModel(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,
        validate_by_name=True,
    )
    name: str
    age: int

# Validate from ORM object
class ORMPerson:
    def __init__(self):
        self.name = "Alice"
        self.age = 30

user = UserModel.model_validate(ORMPerson())
# user.name == 'Alice', user.age == 30
```
- **What it demonstrates**: Validating from ORM attributes with `from_attributes`

```python
from pydantic import BaseModel, ConfigDict, TypeAdapter

config = ConfigDict(strict=True, frozen=True)
adapter = TypeAdapter(dict[str, int], config=config)
# Validates keys/values strictly, returns frozen result
```
- **What it demonstrates**: Applying config to non-model types via TypeAdapter

## Worked Example
Configure a frozen, strict model for a configuration object that shouldn't be modified after creation:

```python
from pydantic import BaseModel, ConfigDict, Field

class AppConfig(BaseModel):
    model_config = ConfigDict(
        frozen=True,
        extra='forbid',
        validate_default=True,
    )
    db_host: str = 'localhost'
    db_port: int = Field(default=5432, ge=1, le=65535)
    debug: bool = False
    max_connections: int = Field(default=10, gt=0)

config = AppConfig()
# config.debug = True  → raises ValidationError (frozen)
# AppConfig(extra_field='x') → raises ValidationError (forbid)
```

## Key Takeaways
1. `ConfigDict` is the single source of truth for model behavior — prefer it over scattered field overrides
2. `frozen=True` makes models value objects — safe for caching, hashing, and thread safety
3. `from_attributes=True` eliminates manual ORM-to-dict mapping
4. Always use `validate_by_name` instead of the deprecated `populate_by_name`

## Connects To
- **Ch 6**: Config options like `title` and `json_schema_extra` affect generated JSON Schema
- **Ch 8**: Config affects how union validation behaves
- **Ch 9**: `alias_generator` and `validate_by_name` control alias resolution
- **Ch 10**: `strict=True` in ConfigDict enables strict mode globally
