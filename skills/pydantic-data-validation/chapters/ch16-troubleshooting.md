# Chapter 16: Troubleshooting

## Core Idea
Most Pydantic errors stem from misunderstandings about forward references, optional fields, and validation behavior. Knowing the common failure modes saves hours of debugging.

## Frameworks Introduced
- **Error Diagnosis Flow**: Start with the error message → identify the category → apply the fix
  - When to use: Any Pydantic validation or type error
  - How: Read the full traceback, locate the error type, match against known patterns
- **Forward Reference Resolution**: Call model_rebuild() after defining all referenced models
  - When to use: Models reference types not yet defined at class creation time
  - How: Define all models first, then call Model.model_rebuild() for each with forward refs

## Key Concepts
- **class-not-fully-defined**: Error when model has unresolved forward references; fixed by calling model_rebuild()
- **extra-forbid**: Setting extra='forbid' in Config causes validation errors on unexpected fields
- **Field naming collisions**: Don't name fields the same as their types (e.g., `color: Color`)
- **Optional[X] semantics**: In V2, `Optional[X]` means `X | None` but does NOT imply `default=None`
- **validate_default**: When True, default values pass through validators; when False (default), defaults are used as-is
- **pydantic-core types**: Use `pydantic-core` types (like `StrictStr`) for stricter validation when needed
- **mypy plugin**: Install `pydantic[mypy]` and enable plugin in mypy config for proper type checking
- **model_validate vs model_validate_json**: Use model_validate for dicts, model_validate_json for JSON strings

## Mental Models
- Forward references are promises to resolve later — if you forget to call model_rebuild(), the promise is broken
- Optional means "can be None" not "can be omitted" — the default value is a separate concern
- Extra fields are like uninvited guests — forbid mode rejects them at the door
- Validators run on input, not on defaults — validate_default=True changes this behavior

## Anti-patterns
- **Forgetting model_rebuild()**: Forward references silently fail or raise class-not-fully-defined
- **Naming fields after types**: Causes circular references and confusing error messages
- **Assuming Optional means optional input**: Leads to unexpected ValidationError when field is missing
- **Using pydantic types instead of pydantic-core**: Missing strict validation options

## Code Examples
```python
from pydantic import BaseModel

# Forward reference issue:
class Foo(BaseModel):
    x: 'Bar'  # Bar not defined yet

class Bar(BaseModel):
    pass

Foo.model_rebuild()  # Must call this
```
- **What it demonstrates**: Resolving forward references after all models are defined

## Worked Example
A user defines models in separate files and imports them. The import order causes forward reference errors. Solution: call model_rebuild() on models with cross-references after all imports complete. Alternatively, restructure imports so referenced models are defined first.

## Key Takeaways
1. Always call model_rebuild() after defining models with forward references
2. Optional[X] means X | None, not that the field can be omitted from input
3. Use validate_default=True when you want default values to go through validation
4. Install pydantic's mypy plugin to catch type errors early

## Connects To
- **Ch 1**: Core concepts and basic model definition
- **Ch 3**: Field constraints and validation
- **Ch 17**: How model_rebuild() works internally
