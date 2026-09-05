# Chapter 18: Migration

## Core Idea
Pydantic V2 is a ground-up rewrite with breaking changes. The migration path is well-documented but understanding the "why" behind changes prevents backsliding.

## Frameworks Introduced
- **Migration Checklist**: Systematic approach to upgrading from V1 to V2
  - When to use: Upgrading any Pydantic project from V1
  - How: Update imports → change validators → update field constraints → test
- **Validator Transformation**: Convert V1 decorator patterns to V2 classmethod patterns
  - When to use: Migrating existing validation logic
  - How: @validator → @field_validator, add @classmethod, update pre/always semantics

## Key Concepts
- **@validator → @field_validator**: V2 validators must be classmethods and use different argument patterns
- **@root_validator → @model_validator**: Model-level validators now use @model_validator
- **Field constraints**: Use ge/le/gt/lt in Field() instead of separate constraint types (conint, etc.)
- **Optional handling**: Optional[X] no longer implies default=None; explicitly set default=None if needed
- **Config dict**: class Config → model_config = ConfigDict(...)
- **parse_obj → model_validate**: Dictionary validation method renamed
- **parse_raw → model_validate_json**: JSON string validation method renamed
- **dict() → model_dump()**: Serialization method renamed
- **json() → model_dump_json()**: JSON serialization method renamed
- **Import paths**: Some types moved between pydantic and pydantic-core

## Mental Models
- V1 was Python-only; V2 is Rust-powered — this changes everything about performance and internals
- Validators are now classmethods, not classmethods with "cls" — this reflects the new architecture
- Optional means "can be None" not "can be omitted" — this is a semantic clarification, not a regression
- ConfigDict is a dict, not a class — this enables better type checking

## Anti-patterns
- **Ignoring deprecation warnings**: V1 methods still work but will be removed; migrate proactively
- **Forgetting @classmethod**: V2 field_validator requires classmethod decorator
- **Assuming backwards compatibility**: Some behaviors changed subtly (e.g., Optional semantics)
- **Mixing V1 and V2 patterns**: Causes confusing errors; commit to one version

## Code Examples
```python
# V1:
# from pydantic import BaseModel, validator
# class Model(BaseModel):
#     name: str
#     @validator('name')
#     def validate_name(cls, v):
#         return v.strip()

# V2:
from pydantic import BaseModel, field_validator
class Model(BaseModel):
    name: str
    @field_validator('name')
    @classmethod
    def validate_name(cls, v):
        return v.strip()
```
- **What it demonstrates**: Validator migration from V1 to V2 syntax

## Worked Example
Migrating a V1 model with root_validator and multiple field constraints:
1. Replace @root_validator with @model_validator(mode='before')
2. Convert all @validator to @field_validator with @classmethod
3. Replace constr('pattern') with Field(pattern='...')
4. Update Config class to ConfigDict
5. Replace .dict() with .model_dump()

## Key Takeaways
1. V2 is 5-50x faster due to Rust-based pydantic-core
2. Validators must be classmethods in V2
3. Optional[X] no longer implies default=None — set it explicitly
4. Use model_validate/model_validate_json instead of parse_obj/parse_raw

## Connects To
- **Ch 1**: Basic V2 model definition
- **Ch 3**: Field constraints in V2
- **Ch 16**: Common migration issues and fixes
- **Ch 17**: Why the architecture changed
