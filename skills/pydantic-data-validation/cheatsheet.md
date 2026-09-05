# Cheatsheet — Pydantic Decision Guide

## Validation Entry Points

| Scenario | Use | Method |
|----------|-----|--------|
| Python dict → model | `Model.model_validate(data)` | Python mode |
| JSON string → model | `Model.model_validate_json(json_str)` | JSON mode |
| String dict → model | `Model.model_validate_strings(data)` | JSON coercion |
| Any type (no model) | `TypeAdapter(T).validate_python(v)` | Standalone |
| Function args | `@validate_call` decorator | Per-call |

## Strict vs Lax Mode

| When | Mode | How |
|------|------|-----|
| API boundary, reject `"123"` as int | **Strict** | `Field(strict=True)` or `StrictInt` |
| Internal code, allow `"123"` → `123` | **Lax** (default) | Coercion enabled |
| Per-call override | Either | `model_validate(data, strict=True)` |

## Union Strategy

| Situation | Strategy | Performance |
|-----------|----------|-------------|
| Simple tag field exists | **Discriminated union** | Fastest |
| No tag, smart matching OK | **Smart mode** (default) | Good |
| Need predictable order | **Left-to-right** | Slowest |
| Complex logic | **Callable discriminator** | Fast |

## Field Naming

| Need | Use | Example |
|------|-----|---------|
| Same name in API & code | Default | `name: str` |
| Different input name | `validation_alias` | `Field(validation_alias='user_name')` |
| Different output name | `serialization_alias` | `Field(serialization_alias='user_name')` |
| Both differ | Both params | `validation_alias='x', serialization_alias='y'` |
| Nested location | `AliasPath` | `AliasPath('data', 'user', 'name')` |
| Multiple locations | `AliasChoices` | `AliasChoices('name', 'user.name')` |

## Default Values

| Pattern | Use |
|---------|-----|
| `field: type = value` | Simple default |
| `Field(default_factory=fn)` | Mutable default (list, dict) |
| `default_factory=lambda data: ...` | Default from validated data |
| `validate_default=True` | Validate the default itself |

## Extra Fields

| `extra` setting | Behavior |
|----------------|----------|
| `'ignore'` (default) | Silently drop extra fields |
| `'forbid'` | Raise error on extra fields |
| `'allow'` | Store in `model_extra` dict |

## Serialization Control

| Goal | Method |
|------|--------|
| Python dict | `model_dump()` |
| JSON string | `model_dump_json()` |
| Use aliases | `model_dump(by_alias=True)` |
| Skip Nones | `model_dump(exclude_none=True)` |
| Only specific fields | `model_dump(include={'a', 'b'})` |
| Exclude fields | `model_dump(exclude={'password'})` |
| Skip unset fields | `model_dump(exclude_unset=True)` |
| JSON-compatible types | `model_dump(mode='json')` |

## Validator Selection

| Validator Type | When to Use |
|---------------|-------------|
| `AfterValidator` | Post-validation checks (type-safe) |
| `BeforeValidator` | Pre-validation transformation |
| `PlainValidator` | Take full control, skip Pydantic checks |
| `WrapValidator` | Wrap entire pipeline, modify/skip/errors |
| `@model_validator` | Cross-field validation rules |

## Common Anti-patterns

| Don't | Do Instead |
|-------|------------|
| Name field same as type (`int: int`) | Use different names |
| Use `Sequence` for list fields | Use `list` (faster validation) |
| Use `model_construct()` for untrusted data | Always validate first |
| Catch `ValidationError` silently | Log and handle properly |
| Use V1 `@validator` in V2 | Use `@field_validator` with `@classmethod` |
| Forget `model_rebuild()` for forward refs | Call after defining referenced classes |
| Use `Optional[X] = None` to mean optional | Use `X = None` (Optional doesn't imply default) |

## Error Handling Quick Reference

```python
from pydantic import ValidationError
try:
    Model(**data)
except ValidationError as e:
    # List all errors
    for err in e.errors():
        print(err['type'], err['loc'], err['msg'])
    # Get JSON representation
    print(e.json())
    # Get total count
    print(f"{len(e.errors())} errors")
```

## ConfigDict Quick Reference

| Setting | Values | Default | Purpose |
|---------|--------|---------|---------|
| `extra` | `'ignore'`, `'forbid'`, `'allow'` | `'ignore'` | Handle unknown fields |
| `frozen` | `bool` | `False` | Immutable models |
| `strict` | `bool` | `False` | No type coercion |
| `validate_default` | `bool` | `False` | Validate defaults |
| `from_attributes` | `bool` | `False` | Validate from objects |
| `validate_by_name` | `bool` | `False` | Allow name or alias |
| `str_max_length` | `int` | `None` | Max string length |
| `alias_generator` | `Callable` | `None` | Auto-generate aliases |
