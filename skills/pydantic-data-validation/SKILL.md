---
name: pydantic-data-validation
description: "Knowledge base from \"Pydantic Documentation\" by Pydantic Team. Use when applying Pydantic's validation, serialization, and schema patterns, studying data modeling best practices, or referencing its concepts for API design."
---

<!-- argument-hint: [topic, framework name, or chapter number] -->

# Pydantic Data Validation
**Author**: Pydantic Team | **Chapters**: 20 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `models`, `validators`, ` unions`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch01`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

### BaseModel — The Schema Anchor
Define data contracts as Python classes with annotated fields. Every field has a type, constraints live in `Field()`, and validation happens at instantiation. **Use BaseModel when you need structured, validated data.**

```python
from pydantic import BaseModel, Field
class User(BaseModel):
    id: int
    name: str = Field(min_length=1)
    email: str
```

### Validation Modes — Three Entry Points
- **Python mode**: `Model.model_validate(dict)` or `Model(**kwargs)` — validates Python objects
- **JSON mode**: `Model.model_validate_json(json_str)` — validates JSON directly (faster for API payloads)
- **Strings mode**: `Model.model_validate_strings(dict)` — validates string dicts with JSON-mode coercion rules

### The Annotated Pattern — Composable Type Metadata
Attach constraints, validators, and serializers to types without polluting field definitions:
```python
from typing import Annotated
from pydantic import Field, AfterValidator
PositiveStr = Annotated[str, Field(min_length=1)]
```
**Use Annotated for reusable types. Use Field() defaults for field-specific metadata.**

### Strict vs Lax — Coercion Control
- **Lax mode** (default): Pydantic coerces types (`"123"` → `123`, `1.0` → `1`)
- **Strict mode**: rejects coercion, requires exact type matches
- Enable per-field (`Field(strict=True)`), per-model (`ConfigDict(strict=True)`), or per-call (`model_validate(data, strict=True)`)

### Union Strategy — Match Fastest
1. **Discriminated union** (fastest): `Field(discriminator='tag')` with `Literal` types
2. **Smart mode** (default): tries all members, picks best by exactness
3. **Left-to-right** (predictable): tries in definition order
**Always prefer discriminated unions when a tag field exists.**

### Four Validator Types — Choose by Stage
| Stage | Decorator | Annotated | Use When |
|-------|-----------|-----------|----------|
| After | `@field_validator(mode='after')` | `AfterValidator(fn)` | Post-validation checks |
| Before | `@field_validator(mode='before')` | `BeforeValidator(fn)` | Input normalization |
| Plain | `@field_validator(mode='plain')` | `PlainValidator(fn)` | Full control, skip Pydantic |
| Wrap | `@field_validator(mode='wrap')` | `WrapValidator(fn)` | Wrap entire pipeline |

### Serialization — Controlled Output
- `model_dump()` → Python dict, `model_dump_json()` → JSON string
- `by_alias=True` → use alias names, `exclude_none=True` → skip Nones
- Custom: `PlainSerializer`, `WrapSerializer`, `@field_serializer`

### ConfigDict — Model Behavior Tuning
Key settings: `extra` (ignore/forbid/allow), `frozen` (immutable), `from_attributes` (ORM), `validate_by_name` (name+alias), `validate_default`, `alias_generator`

### TypeAdapter — Validate Anything
Standalone validation without defining a model:
```python
from pydantic import TypeAdapter
ta = TypeAdapter(list[int])
ta.validate_python([1, '2', 3])  # [1, 2, 3]
```

### Discriminator — Smart Union Dispatch
- **String discriminator**: `Field(discriminator='pet_type')` with `Literal` tags
- **Callable discriminator**: `Discriminator(fn)` for complex dispatch logic
- Account for both dict and model inputs in callable discriminators

### model_construct — Skip Validation (Carefully)
Creates model instances without validation. **Only use with data you've already validated.** Performance gap narrowed in V2 — profile before assuming it's faster.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-models.md) | Models | BaseModel, model_validate, model_dump, model_construct |
| [ch02](chapters/ch02-fields.md) | Fields | Field(), Annotated pattern, defaults, constraints, aliases |
| [ch03](chapters/ch03-types.md) | Types | Built-in types, custom types, type aliases |
| [ch04](chapters/ch04-validators.md) | Validators | After/Before/Plain/Wrap validators, @field_validator |
| [ch05](chapters/ch05-serialization.md) | Serialization | model_dump, model_dump_json, custom serializers |
| [ch06](chapters/ch06-json-schema.md) | JSON Schema | model_json_schema, WithJsonSchema, $defs |
| [ch07](chapters/ch07-config.md) | Config | ConfigDict, model_config, from_attributes |
| [ch08](chapters/ch08-unions.md) | Unions | Smart, left-to-right, discriminated unions, Tag |
| [ch09](chapters/ch09-aliases.md) | Aliases | validation_alias, AliasPath, AliasChoices |
| [ch10](chapters/ch10-strict-mode.md) | Strict Mode | StrictInt/Float/Str/Bool, strict layers |
| [ch11](chapters/ch11-performance.md) | Performance | model_construct, concrete types, JSON validation |
| [ch12](chapters/ch12-dataclasses.md) | Dataclasses | Pydantic dataclasses, TypeAdapter bridge |
| [ch13](chapters/ch13-type-adapter.md) | TypeAdapter | Standalone validation, dump, json_schema |
| [ch14](chapters/ch14-validate-call.md) | Validate Call | @validate_call, function arg validation |
| [ch15](chapters/ch15-errors.md) | Errors | ValidationError, error types, PydanticCustomError |
| [ch16](chapters/ch16-troubleshooting.md) | Troubleshooting | Common issues, forward refs, Optional semantics |
| [ch17](chapters/ch17-architecture.md) | Architecture | Core schema, pydantic-core, validation flow |
| [ch18](chapters/ch18-migration.md) | Migration V1→V2 | Validator syntax, method renames, ConfigDict |
| [ch19](chapters/ch19-integrations.md) | Integrations | FastAPI, mypy, ORMs, Hypothesis, Rich |
| [ch20](chapters/ch20-examples.md) | Examples & Recipes | Dynamic models, generics, API patterns |

## Topic Index

- **BaseModel** → ch01
- **ConfigDict** → ch07
- **Field()** → ch02
- **TypeAdapter** → ch13
- **ValidationError** → ch15
- **aliases** → ch09
- **computed fields** → ch02
- **dataclasses** → ch12
- **discriminated unions** → ch08
- **extra fields** → ch01, ch07
- **forward references** → ch16, ch17
- **generics** → ch03, ch20
- **JSON Schema** → ch06
- **model_construct** → ch01, ch11
- **model_dump** → ch05
- **model_validate** → ch01
- **performance** → ch11
- **RootModel** → ch01
- **SecretStr** → ch03
- **serialization** → ch05
- **strict mode** → ch10
- **unions** → ch08
- **validate_call** → ch14
- **validators** → ch04

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the Pydantic documentation only. For hands-on implementation in your codebase,
combine with project-specific tools. For topics beyond Pydantic, check related skills
or ask the agent directly.
