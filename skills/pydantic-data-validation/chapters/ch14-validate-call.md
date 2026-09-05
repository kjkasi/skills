# Chapter 14: Validate Call

## Core Idea
`@validate_call` is a decorator that validates function arguments and return values against type hints, enforcing contracts without manual checks.

## Frameworks Introduced
- **Argument Validation Decorator**: Validate all function arguments automatically.
  - When to use: Public APIs, library functions, any function where input correctness matters.
  - How: Decorate with `@validate_call` and add type hints to parameters.
- **Return Type Validation**: Validate what a function returns.
  - When to use: Ensuring downstream consumers get correctly shaped data.
  - How: Add a return type annotation; `@validate_call` validates it.

## Key Concepts
- **`@validate_call`**: Decorator that validates positional and keyword arguments against type hints at call time.
- **Coercion**: Like other Pydantic validators, `@validate_call` coerces compatible types (e.g., `"123"` → `123` for `int`).
- **Return Type Validation**: If the function has a return type annotation, the return value is validated too.
- **Strict Mode**: Pass `strict=True` to `@validate_call(strict=True)` to disable coercion.
- **Async Support**: Works with `async def` functions — the return coroutine is validated when awaited.
- **Signature Preservation**: The decorated function preserves its original signature for IDE autocomplete and type checkers.

## Mental Models
1. **Contract Enforcement**: Think of `@validate_call` as a contract — type hints become runtime guarantees, not just documentation.
2. **Boundary Defense**: Apply at system boundaries (API endpoints, CLI handlers) to catch bad data early.
3. **Lightweight Alternative**: When a full model is overkill, `@validate_call` gives you validation with zero boilerplate.

## Anti-patterns
- **Applying to every internal function**: Adds overhead; reserve for public APIs and boundaries.
- **Relying on coercion instead of explicit parsing**: Coercion is convenient but can hide bugs — use strict mode when clarity matters.
- **Forgetting return type validation**: Without a return type hint, only arguments are validated.

## Code Examples
```python
from pydantic import validate_call

@validate_call
def get_user(user_id: int, name: str = 'default') -> dict:
    return {'id': user_id, 'name': name}

get_user('123')  # coerced to 123
get_user(user_id=456)
```
- **What it demonstrates**: Automatic coercion of arguments and return type validation.

## Worked Example
Validate a function that processes payment amounts:

```python
from pydantic import validate_call

@validate_call(strict=True)
def charge(amount: float, currency: str = 'USD') -> dict:
    return {'amount': amount, 'currency': currency}

# These work:
charge(19.99)
charge(amount=29.99, currency='EUR')

# This fails (strict mode, no coercion):
charge('19.99')  # ValidationError
```

For async functions:

```python
from pydantic import validate_call

@validate_call
async def fetch_user(user_id: int) -> dict:
    return {'id': user_id, 'name': 'John'}

result = await fetch_user('123')  # coerced to 123
```

## Key Takeaways
1. `@validate_call` validates function arguments and return values against type hints.
2. Supports coercion by default; use `strict=True` to disable it.
3. Works with both sync and async functions without modification.
4. Preserves function signatures for IDE support and type checking.

## Connects To
- **Ch 4**: Same type coercion and validation rules as BaseModel fields.
- **Ch 6**: Strict mode behaves identically here as in models.
- **Ch 11**: Adds validation overhead — profile if used in hot paths.
- **Ch 15**: Raises the same ValidationError with the same error structure.
