# Chapter 4: Validators

## Core Idea
Custom validation beyond type constraints is implemented via field validators (`@field_validator`) and model validators (`@model_validator`), with four modes controlling where they sit in the validation pipeline.

## Frameworks Introduced
- **@field_validator / @model_validator decorators**: Class-based validation functions that hook into the Pydantic validation pipeline
  - When to use: When field constraints alone aren't enough to enforce business rules
  - How: Define a `@classmethod` decorated with `@field_validator('field_name')` or `@model_validator(mode='...')`

- **Annotated validators**: `BeforeValidator`, `AfterValidator`, `PlainValidator`, `WrapValidator` via `Annotated` pattern
  - When to use: When you want reusable validators as type aliases or prefer functional composition
  - How: `Annotated[str, AfterValidator(strip_whitespace)]` — validator runs as part of the type

## Key Concepts
- **AfterValidator / mode='after'**: Runs after Pydantic's built-in validation; receives already-validated, type-safe data
- **BeforeValidator / mode='before'**: Runs before validation; receives raw input that may not match the target type yet
- **PlainValidator / mode='plain'**: Terminates validation — no further checks run; you own the entire validation chain
- **WrapValidator / mode='wrap'**: Wraps the entire validation pipeline; can modify input, skip validation, or catch errors
- **@model_validator(mode='before')**: Runs before field validation; receives raw input dict for pre-processing
- **@model_validator(mode='after')**: Runs after all fields are validated; receives the constructed model instance
- **@model_validator(mode='wrap')**: Wraps the entire model validation; can modify input or handle errors
- **Raising ValidationError**: Raise `ValueError` in validators (Pydantic converts it) or use `PydanticCustomError` for structured error info
- **Validation context**: Pass extra data via `model_validate(data, context={...})`; accessible in validators via `info.context`

## Mental Models
- Think of the four validator modes as a pipeline: `before` → type validation → `after`, with `wrap` surrounding the whole thing and `plain` replacing it entirely.
- Use `mode='after'` as your default — it's the safest because you receive validated, typed data; only use `before` when you must handle raw input formats.
- Treat `@model_validator(mode='after')` as the place for cross-field business rules; field validators handle single-field logic.

## Anti-patterns
- **Using mode='plain' without careful thought**: It skips all built-in validation; you must handle type checking yourself or risk invalid data passing through
- **Raising generic ValueError without messages**: Pydantic shows your message to users — always provide clear, actionable error messages

## Code Examples
```python
from pydantic import BaseModel, field_validator, model_validator

class Model(BaseModel):
    name: str

    @field_validator('name')
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('name must not be empty')
        return v.strip()

    @model_validator(mode='after')
    def check_name(self) -> 'Model':
        if self.name.startswith('X'):
            raise ValueError('names starting with X are not allowed')
        return self
```
- **What it demonstrates**: Field-level stripping/cleaning with `@field_validator` and cross-field business rule with `@model_validator`

## Worked Example
```python
from pydantic import BaseModel, field_validator, model_validator, ValidationError

class Registration(BaseModel):
    username: str
    password: str
    password_confirm: str

    @field_validator('username')
    @classmethod
    def username_alphanumeric(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError('username must be alphanumeric')
        return v.lower()

    @model_validator(mode='after')
    def passwords_match(self) -> 'Registration':
        if self.password != self.password_confirm:
            raise ValueError('passwords do not match')
        return self

# Valid input
reg = Registration(
    username='Alice123',
    password='s3cret!',
    password_confirm='s3cret!'
)
assert reg.username == 'alice123'

# Invalid: passwords don't match
try:
    Registration(
        username='Bob',
        password='s3cret!',
        password_confirm='wrong'
    )
except ValidationError as e:
    print(e)
    #> 1 validation error for Registration
    #> ...
    #>   Value error, passwords do not match
```

## Key Takeaways
1. Default to `mode='after'` validators — they receive type-safe data and are easier to reason about
2. Use `@model_validator(mode='after')` for cross-field business rules; use `@field_validator` for single-field logic
3. Always provide clear error messages in validators — Pydantic surfaces them to the end user

## Connects To
- **Ch 2 Fields**: Field constraints handle simple rules; validators handle complex logic that constraints can't express
- **Ch 5 Serializers**: Validators and serializers form a pipeline: validate input → store → serialize output
- **Ch 1 Models**: `model_validate()` triggers the full validation pipeline including all validators
