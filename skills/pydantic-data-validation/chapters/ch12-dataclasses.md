# Chapter 12: Dataclasses

## Core Idea
Pydantic dataclasses are drop-in replacements for stdlib dataclasses, giving you the same field syntax with Pydantic's validation and serialization baked in.

## Frameworks Introduced
- **Pydantic Dataclass as BaseModel Alternative**: Use `@pydantic.dataclasses.dataclass` for validation without inheriting from `BaseModel`.
  - When to use: When you prefer dataclass syntax but need validation/serialization.
  - How: Decorate with `@dataclass` from `pydantic.dataclasses` and annotate fields with types.
- **Integrating Stdlib Dataclasses**: Validate existing stdlib dataclasses using `TypeAdapter`.
  - When to use: Working with codebases that already use `@dataclass`.
  - How: Wrap with `TypeAdapter(MyStdlibDataclass).validate_python(obj)`.

## Key Concepts
- **`@pydantic.dataclasses.dataclass`**: A decorator that adds Pydantic validation to a dataclass. Supports `__pydantic_fields__` and `__pydantic_config__`.
- **`__pydantic_config__`**: Use to pass `ConfigDict` options to a Pydantic dataclass.
- **`__pydantic_fields__` vs `__dataclass_fields__`**: Pydantic dataclasses expose both; `__pydantic_fields__` has richer type info.
- **Nested Dataclasses**: Pydantic dataclasses can contain other Pydantic (or stdlib) dataclasses, with automatic validation.

## Mental Models
1. **Drop-in Replacement**: Think of `pydantic.dataclasses.dataclass` as a swap for `dataclasses.dataclass` that adds validation.
2. **Shared Engine Under the Hood**: Pydantic dataclasses use the same validation core as `BaseModel` — performance and behavior are comparable.
3. **Stdlib Bridge**: When you can't change existing dataclasses, `TypeAdapter` lets you validate them without modification.

## Anti-patterns
- **Mixing `__init__` behaviors**: Pydantic dataclasses handle defaults and coercion differently from stdlib dataclasses; assumptions about `__init__` may break.
- **Assuming `slots=True` compatibility**: Not all Pydantic dataclass features work with `slots=True`; test thoroughly.
- **Ignoring `eq` differences**: Pydantic dataclasses may compare by value differently depending on config.

## Code Examples
```python
from pydantic import dataclass

@dataclass
class User:
    name: str
    age: int

user = User(name='John', age='30')  # coerced
```
- **What it demonstrates**: Pydantic dataclass with automatic type coercion.

## Worked Example
You have an existing stdlib dataclass in a library:

```python
from dataclasses import dataclass

@dataclass
class LegacyUser:
    name: str
    age: int
```

To validate instances:

```python
from pydantic import TypeAdapter

ta = TypeAdapter(LegacyUser)
validated = ta.validate_python(LegacyUser(name='John', age='thirty'))
# Raises ValidationError: age must be int
```

For new code, use Pydantic's dataclass directly:

```python
from pydantic import dataclass
from pydantic.config import ConfigDict

@dataclass
class ModernUser:
    model_config = ConfigDict(strict=True)
    name: str
    age: int
```

## Key Takeaways
1. `pydantic.dataclasses.dataclass` is a drop-in replacement with full validation and serialization support.
2. ConfigDict is supported via `__pydantic_config__` or `model_config`.
3. Stdlib dataclasses can be validated using `TypeAdapter` without changing existing code.
4. Performance is comparable to `BaseModel`.

## Connects To
- **Ch 4**: BaseModel and dataclasses share the same validation engine.
- **Ch 13**: TypeAdapter is the bridge for validating stdlib dataclasses.
- **Ch 11**: Performance characteristics are similar to BaseModel.
