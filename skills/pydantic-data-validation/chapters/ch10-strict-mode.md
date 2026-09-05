# Chapter 10: Strict Mode

## Core Idea
Strict mode disables Pydantic's automatic type coercion, requiring exact type matches — giving you precise control over what gets accepted and preventing silent type conversions.

## Frameworks Introduced
- **Strict Mode**: No type coercion; values must match the expected type exactly
  - When to use: When you need precise type control, financial data, IDs, or when coercion could cause bugs
  - How: Enable via `Field(strict=True)`, `ConfigDict(strict=True)`, or `model_validate(data, strict=True)`
- **Lax Mode (Default)**: Pydantic coerces compatible types automatically
  - When to use: When you want flexibility (e.g., accepting `'123'` as `int`)
  - How: Default behavior — no configuration needed
- **Strict Types**: Wrapper types that enforce strictness regardless of config
  - When to use: When most fields should be lax but specific fields need strictness
  - How: Use `StrictInt`, `StrictFloat`, `StrictStr`, `StrictBool` as field types

## Key Concepts
- **Field-level strict**: `Field(strict=True)` — one field is strict, rest follow model config
- **Model-level strict**: `ConfigDict(strict=True)` — all fields are strict by default
- **Per-call strict**: `model_validate(data, strict=True)` — strict for one validation call
- **StrictInt / StrictFloat / StrictStr / StrictBool**: Type wrappers that are always strict
- **Union behavior**: Strict mode changes which union member matches — no implicit widening
- **JSON vs Python**: Strict mode behaves differently for JSON input (string types) vs Python objects

## Mental Models
1. **Coercion as Convenience**: Lax mode is Pydantic being helpful; strict mode is Pydantic being precise
2. **Type Gate**: Strict mode is a gatekeeper — only exact type matches pass through
3. **Layered Control**: Field → Model → Call — strictness can be applied at any level, inner wins
4. **Strict ≠ Safe**: Strict mode prevents coercion but doesn't validate business rules — you still need constraints

## Anti-patterns
- **Enabling strict globally when only one field needs it**: Use `Field(strict=True)` or `StrictInt` instead
- **Assuming strict mode prevents all bad data**: It only prevents type coercion, not invalid values
- **Mixing strict and lax without clear reasoning**: Document why each field has its strictness level
- **Using strict mode with from_attributes without understanding**: ORM attributes are already Python types — strict may reject valid data

## Code Examples
```python
from pydantic import BaseModel, Field, StrictInt

class Model(BaseModel):
    strict_int: StrictInt
    regular_int: int = Field(strict=True)
    lenient_int: int

# Model(strict_int='123') → ValidationError (StrictInt never coerces)
# Model(regular_int='123') → ValidationError (Field strict=True)
# Model(lenient_int='123') → ok, coerced to 123
```
- **What it demonstrates**: Three levels of strictness on the same type

```python
from pydantic import BaseModel, ConfigDict

class StrictModel(BaseModel):
    model_config = ConfigDict(strict=True)
    name: str
    age: int
    active: bool

# StrictModel(name='Alice', age='30') → ValidationError (age must be int, not str)
# StrictModel(name='Alice', age=30, active=True) → ok
```
- **What it demonstrates**: Model-level strict configuration

```python
from pydantic import BaseModel

class Model(BaseModel):
    value: int

# Per-call strict
result = Model.model_validate({'value': '123'}, strict=False)  # ok, coerced
result = Model.model_validate({'value': '123'}, strict=True)   # ValidationError
```
- **What it demonstrates**: Per-call strict mode override

```python
from pydantic import BaseModel, StrictFloat, StrictBool

class FinancialData(BaseModel):
    amount: StrictFloat
    is_verified: StrictBool

# amount=10 → ok (int is coerced to float in strict? No — StrictFloat rejects int)
# amount=10.0 → ok
# is_verified=True → ok
# is_verified=1 → ValidationError (StrictBool rejects int)
```
- **What it demonstrates**: Strict types for financial/boolean precision

## Worked Example
Build a configuration parser that rejects ambiguous types:

```python
from pydantic import BaseModel, ConfigDict, Field, StrictInt, StrictStr, StrictBool

class ServerConfig(BaseModel):
    model_config = ConfigDict(strict=True)
    host: StrictStr = 'localhost'
    port: StrictInt = Field(default=8080, ge=1, le=65535)
    debug: StrictBool = False
    max_retries: StrictInt = 3

# Strict: must pass exact types
config = ServerConfig(host='0.0.0.0', port=3000, debug=True, max_retries=5)

# These all fail:
# ServerConfig(port='3000') → ValidationError (port is StrictInt)
# ServerConfig(debug=1) → ValidationError (debug is StrictBool)
# ServerConfig(host=123) → ValidationError (host is StrictStr)

# Per-call override for lax parsing
config = ServerConfig.model_validate(
    {'host': '0.0.0.0', 'port': '3000', 'debug': 1},
    strict=False
)
# port coerced to 3000, debug coerced to True
```

## Key Takeaways
1. Strict mode is layered: field-level overrides model-level, per-call overrides both
2. `StrictInt`, `StrictFloat`, etc. are type-level strictness — use them when only certain fields need precision
3. Strict mode doesn't validate business rules — combine with constraints (`ge`, `le`, `pattern`)
4. JSON input behaves differently — strings are the default JSON type, so strict mode rejects string-to-type coercion more aggressively

## Connects To
- **Ch 7**: `ConfigDict(strict=True)` enables strict mode globally for a model
- **Ch 8**: Strict mode affects which union member matches when types overlap
- **Ch 9**: Strict mode doesn't affect alias resolution — aliases are resolved before type validation
- **Ch 6**: Strict mode doesn't change generated JSON Schema — schema describes the shape, not the coercion behavior
