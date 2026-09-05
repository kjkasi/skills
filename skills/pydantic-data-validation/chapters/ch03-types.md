# Chapter 3: Types

## Core Idea
Pydantic supports Python's built-in types, standard library types, and custom types via the `Annotated` pattern with validators, enabling precise data validation for any domain.

## Frameworks Introduced
- **Standard library types**: Python's built-in types (`int`, `str`, `float`, `bool`, `list`, `dict`, `set`, `tuple`) with automatic coercion
  - When to use: Default choice for most fields; Pydantic handles type conversion automatically
  - How: Use directly as type hints; Pydantic coerces compatible types (e.g., `"42"` → `42`)

- **Annotated type aliases**: Reusable constrained types defined via `Annotated` with `Field()` or validators
  - When to use: When the same constraint applies across many fields
  - How: Define once (`PositiveInt = Annotated[int, Field(gt=0)]`) and use as a type hint anywhere

## Key Concepts
- **Standard library types**: `int`, `str`, `float`, `bool`, `list`, `dict`, `set`, `tuple` — Pydantic coerces compatible types automatically
- **Date/time types**: `datetime`, `date`, `time`, `timedelta`, `tz-aware` and `tz-naive` variants; parsed from strings and timestamps
- **Numeric types**: `int`, `float`, `Decimal`, `Complex`, `Fraction` — each with different precision and coercion rules
- **String types**: `str`, `SecretStr`, `SecretBytes`, `EmailStr`, `NameEmail` — specialized string validation
- **Network types**: `IPvAnyAddress`, `IPvAnyNetwork`, `AnyHttpUrl`, `PostgresDsn`, etc. — validated network addresses and URLs
- **UUID types**: `UUID1` through `UUID8` — specific UUID version validation
- **Strict mode**: `Field(strict=True)` or `ConfigDict(strict=True)` — only exact type matches accepted, no coercion
- **Custom types via Annotated**: Use `BeforeValidator`, `AfterValidator`, `PlainValidator` for custom parsing logic
- **Named type aliases**: PEP 695 `type` statement for defining reusable type aliases

## Mental Models
- Think of Pydantic's type system as a layered defense: built-in types handle common cases, `Annotated` adds constraints, and custom validators handle domain logic.
- Use strict mode when you need to reject ambiguous input (e.g., `"123"` should not become `123`); it's a semantic signal that exact types matter.
- Treat network types like `AnyHttpUrl` as self-documenting — they encode validation rules in the type itself, making schemas readable.

## Anti-patterns
- **Assuming coercion is always desirable**: Automatic type coercion can hide bugs; use `strict=True` when exact types are required
- **Ignoring timezone awareness**: Mixing naive and aware datetime objects causes runtime errors; be explicit about tz requirements

## Code Examples
```python
from typing import Annotated
from pydantic import BaseModel, Field, AfterValidator

PositiveInt = Annotated[int, Field(gt=0)]

class Model(BaseModel):
    id: PositiveInt
    name: str
```
- **What it demonstrates**: Reusable constrained type alias for a positive integer

## Worked Example
```python
from datetime import datetime, timezone
from pydantic import BaseModel, ConfigDict

class Event(BaseModel):
    name: str
    start: datetime
    duration_seconds: float = Field(ge=0)

# Parses ISO format strings automatically
event = Event(
    name='Conference',
    start='2025-01-15T09:00:00+00:00',
    duration_seconds='3600'
)

assert event.start.tzinfo is not None
assert event.duration_seconds == 3600.0

# Strict mode rejects coercion
class StrictModel(BaseModel):
    model_config = ConfigDict(strict=True)
    value: int

try:
    StrictModel(value='42')  # rejected — string is not int
except Exception as e:
    print(e)
```

## Key Takeaways
1. Start with Python's built-in types; Pydantic handles coercion automatically for most common cases
2. Use `Annotated` type aliases to encode domain constraints (like `PositiveInt`) directly in the type system
3. Enable `strict=True` when exact type matching matters — it prevents subtle coercion bugs

## Connects To
- **Ch 1 Models**: Types are the foundation of model field definitions
- **Ch 2 Fields**: Field constraints work alongside type validation; `Field(strict=True)` disables coercion
- **Ch 4 Validators**: Custom validators extend type validation with business logic
