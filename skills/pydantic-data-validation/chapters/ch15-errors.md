# Chapter 15: Errors

## Core Idea
Pydantic's `ValidationError` gives structured, actionable error details — learn to catch, parse, and customize them for robust error handling.

## Frameworks Introduced
- **Structured Error Handling**: Catch `ValidationError` and iterate over error details.
  - When to use: Any validation boundary where you need to report specific failures.
  - How: `try/except ValidationError as e`, then `e.errors()` for structured details.
- **Custom Error Types**: Use `PydanticCustomError` to define domain-specific errors.
  - When to use: When built-in error types don't convey enough context.
  - How: Raise `PydanticCustomError('my_error', 'message', {'key': value})`.

## Key Concepts
- **`ValidationError`**: Raised when validation fails. Contains a list of individual errors.
- **`error.errors()`**: Returns a list of dicts, each with `type`, `loc`, `msg`, `input`, and optionally `ctx` and `url`.
- **Error Location (`loc`)**: Tuple showing where the error occurred (field name, index, nested path).
- **Error Type (`type`)**: Machine-readable identifier like `missing`, `string_type`, `int_parsing`, `value_error`.
- **Error Context (`ctx`)**: Additional info like expected pattern,最小/最大 values, or custom context.
- **Error URL (`url`)**: Link to Pydantic documentation for that specific error type.
- **`PydanticCustomError`**: Create custom error types with your own type string, message template, and context.
- **`PydanticUserError`**: Raised for configuration/programming mistakes (e.g., missing required fields in config).

## Mental Models
1. **Errors as Data**: Don't treat validation errors as strings — they're structured objects with location, type, and context.
2. **Nested Locs = Nested Paths**: `loc=('users', 0, 'name')` means the error is in the first element of a `users` list, in the `name` field.
3. **Type as Contract**: Error types are stable identifiers you can match on in code — use them, not substring matching on messages.

## Anti-patterns
- **Catching `Exception` instead of `ValidationError`**: Misses Pydantic-specific error structure and can mask unrelated errors.
- **Matching error messages by string**: Messages can change between versions; match on `type` instead.
- **Ignoring `ctx`**: Context contains actionable details (expected pattern, bounds) — use them in user-facing messages.
- **Not checking `url`**: Error URLs point to docs that explain the exact issue and fix.

## Code Examples
```python
from pydantic import BaseModel, ValidationError

class Model(BaseModel):
    name: str
    age: int

try:
    Model(name=123, age='invalid')
except ValidationError as e:
    for err in e.errors():
        print(f"{err['type']}: {err['msg']} at {err['loc']}")
```
- **What it demonstrates**: Catching and iterating over structured validation errors.

## Worked Example
Validate a user registration payload with multiple errors:

```python
from pydantic import BaseModel, ValidationError, PydanticCustomError

class Registration(BaseModel):
    username: str
    email: str
    age: int

try:
    Registration(username='', email='not-an-email', age=-5)
except ValidationError as e:
    for err in e.errors():
        print(f"Type: {err['type']}")
        print(f"Location: {err['loc']}")
        print(f"Message: {err['msg']}")
        print(f"Input: {err['input']}")
        if 'ctx' in err:
            print(f"Context: {err['ctx']}")
        if 'url' in err:
            print(f"Docs: {err['url']}")
        print()
```

Custom error for domain logic:

```python
from pydantic import BaseModel, validator, PydanticCustomError

class Order(BaseModel):
    quantity: int

    @validator('quantity')
    def check_quantity(cls, v):
        if v <= 0:
            raise PydanticCustomError(
                'invalid_quantity',
                'Quantity must be positive, got {quantity}',
                {'quantity': v}
            )
        return v
```

## Key Takeaways
1. `ValidationError.errors()` returns structured dicts with `type`, `loc`, `msg`, `input`, `ctx`, and `url`.
2. Match on error `type`, not message strings — types are stable across versions.
3. `PydanticCustomError` lets you define domain-specific error types with context.
4. Error URLs link to documentation explaining the exact issue and fix.

## Connects To
- **Ch 4**: All BaseModel validation raises this same error structure.
- **Ch 6**: Strict mode changes which error types are raised (e.g., `string_type` vs `int_parsing`).
- **Ch 12**: Pydantic dataclasses raise identical validation errors.
- **Ch 13**: TypeAdapter raises the same ValidationError for any type.
- **Ch 14**: `@validate_call` raises ValidationError for invalid function arguments.
