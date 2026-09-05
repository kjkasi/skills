# Chapter 11: Performance

## Core Idea
Pydantic validation has real costs; profiling and choosing the right validation strategy (bypass vs. concrete types vs. JSON-direct) can yield significant speedups.

## Frameworks Introduced
- **Bypass Validation with `model_construct()`**: Skip validation entirely when you trust the data.
  - When to use: Trusted internal data, batch loading from known-good sources.
  - How: Call `MyModel.model_construct(field=value)` instead of `MyModel(field=value)`.
- **Concrete Types Over Abstract**: Use concrete container types for faster validation.
  - When to use: Always in hot paths.
  - How: Prefer `list` over `Sequence`, `dict` over `Mapping`.
- **Direct JSON Validation**: Validate JSON strings directly instead of parsing then validating.
  - When to use: API responses, file reads, any JSON string source.
  - How: Use `model_validate_json(json_str)` instead of `model_validate(json.loads(json_str))`.

## Key Concepts
- **`model_construct()`**: Creates an instance without running validation. In V2 the gap is smaller for simple models but still significant for complex ones.
- **Concrete vs Abstract Types**: `list[T]` validates faster than `Sequence[T]` because the validator doesn't need to handle arbitrary iterables.
- **Core Schema Caching**: Pydantic builds a core schema on first use; subsequent validations reuse it. Rebuild once, validate many.

## Mental Models
1. **Profile Before You Optimize**: `model_construct` may not be faster for simple models with few fields. Measure first.
2. **Trust Spectrum**: Data you fully control → bypass. Data from external sources → validate. Data from users → strict validation.
3. **Avoid Over-Generalization**: Abstract types (`Sequence`, `Mapping`) add overhead for flexibility you may not need.
4. **JSON Pipeline**: Every JSON parse-then-validate has two passes; `model_validate_json` collapses them.

## Anti-patterns
- **Using `model_construct()` on untrusted data**: Bypasses all validation, potentially allowing invalid or malicious data into your system.
- **Abstract types in hot loops**: Using `Sequence` or `Mapping` in tight loops adds unnecessary validation overhead.
- **Premature optimization without profiling**: Adding `model_construct` everywhere when the model is simple and validation is negligible.

## Code Examples
```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

# Faster for trusted data (skip validation)
user = User.model_construct(name='John', age=30)

# Faster for JSON: validate directly
user = User.model_validate_json('{"name":"John","age":30}')
```
- **What it demonstrates**: Two key performance optimizations: bypassing validation and direct JSON validation.

## Worked Example
Given a list of 10,000 user records loaded from a trusted internal database:

1. Use `User.model_construct(**record)` to skip validation — avoids redundant checks.
2. If loading from JSON files, use `User.model_validate_json(line)` per line.
3. For fields, use `list[str]` instead of `Sequence[str]` in the model definition.
4. Profile with `timeit` or `cProfile` to confirm gains before applying broadly.

## Key Takeaways
1. `model_construct()` skips validation — use only when data is guaranteed valid.
2. Concrete types (`list`, `dict`) validate faster than abstract types (`Sequence`, `Mapping`).
3. `model_validate_json()` is faster than `json.loads()` + `model_validate()`.
4. Always profile before optimizing; simple models may not benefit from `model_construct`.

## Connects To
- **Ch 6**: Strict mode increases validation cost — relevant when profiling.
- **Ch 8**: Custom serializers can add overhead — profile the full pipeline.
- **Ch 13**: TypeAdapter supports the same JSON-direct optimization.
