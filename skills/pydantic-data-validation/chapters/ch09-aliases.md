# Chapter 9: Aliases

## Core Idea
Aliases decouple Python field names from external names, enabling validation from different key names and serialization to different output names — essential for integrating with APIs that use different naming conventions.

## Frameworks Introduced
- **Field Aliases**: Map a single field to one external name
  - When to use: API uses different names than your Python fields (e.g., `userName` vs `user_name`)
  - How: `Field(alias='userName')` or separate `validation_alias` / `serialization_alias`
- **AliasPath**: Traverse nested structures to find values
  - When to use: External data is nested but your model is flat
  - How: `Field(validation_alias=AliasPath('data', 'user', 'name'))`
- **AliasChoices**: Try multiple alias paths
  - When to use: External data might be at different locations depending on source
  - How: `Field(validation_alias=AliasChoices('name', AliasPath('user', 'name')))`
- **Alias Generator**: Auto-generate aliases for all fields
  - When to use: Systematic naming conversion (e.g., snake_case to camelCase)
  - How: `model_config = ConfigDict(alias_generator=camel_case)`

## Key Concepts
- **alias**: Single alias used for both validation and serialization
- **validation_alias**: Different name for input; takes priority over `alias` during validation
- **serialization_alias**: Different name for output; takes priority over `alias` during serialization
- **AliasPath**: List of keys to traverse in nested dicts
- **AliasChoices**: Multiple alias candidates; first match wins
- **Alias priority**: `validation_alias` > `alias` for input; `serialization_alias` > `alias` for output
- **validate_by_name**: When `True`, both field name and alias are accepted during validation
- **alias_generator**: Callable receiving field name, returns alias string

## Mental Models
1. **Two Doors, One Room**: A field has one identity but two entry points — validation and serialization can use different doors
2. **Path Finding**: `AliasPath` is like a GPS for nested dicts — it navigates to the value wherever it lives
3. **Fallback Chain**: `AliasChoices` tries multiple paths in order — like checking multiple addresses for someone
4. **Convention Bridge**: Aliases bridge naming conventions between systems without changing your internal model

## Anti-patterns
- **Using alias when you need separate validation/serialization names**: Use `validation_alias` and `serialization_alias` explicitly
- **Overusing AliasPath for deeply nested data**: Flatten the data first with a pre-validator instead
- **Forgetting validate_by_name**: When you want both field name and alias accepted, enable this config
- **Mixing alias_generator with per-field aliases**: Per-field aliases override the generator — be explicit about which wins

## Code Examples
```python
from pydantic import BaseModel, Field, AliasPath, AliasChoices

class Model(BaseModel):
    name: str = Field(validation_alias='username')
    aliases: list[int] = Field(
        validation_alias=AliasChoices('ids', AliasPath('data', 'ids'))
    )
```
- **What it demonstrates**: Separate validation alias and multiple alias choices with path traversal

```python
from pydantic import BaseModel, ConfigDict

def to_camel(name: str) -> str:
    parts = name.split('_')
    return parts[0] + ''.join(word.capitalize() for word in parts[1:])

class UserModel(BaseModel):
    model_config = ConfigDict(alias_generator=to_camel, validate_by_name=True)
    first_name: str
    last_name: str
    phone_number: str

# Accepts: {'firstName': 'Alice', 'lastName': 'Smith', 'phoneNumber': '555-1234'}
# Also accepts: {'first_name': 'Alice', 'last_name': 'Smith', 'phone_number': '555-1234'} (validate_by_name)
# Serializes to camelCase by default
```
- **What it demonstrates**: Auto-generating camelCase aliases with validate_by_name fallback

```python
from pydantic import BaseModel, Field

class Model(BaseModel):
    name: str = Field(alias='userName', serialization_alias='userName')
    # validation uses 'userName', serialization uses 'userName'
    # Python field access uses 'name'
```
- **What it demonstrates**: Separate serialization alias from validation alias

## Worked Example
Integrate with an API that uses camelCase and nested user data:

```python
from pydantic import BaseModel, Field, AliasPath, ConfigDict

class APIResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    user_name: str = Field(validation_alias=AliasPath('data', 'user', 'name'))
    user_email: str = Field(validation_alias=AliasPath('data', 'user', 'email'))
    created_at: str = Field(validation_alias='createdAt')

# Validate from API response
response_data = {
    'data': {
        'user': {
            'name': 'Alice',
            'email': 'alice@example.com'
        }
    },
    'createdAt': '2024-01-15'
}
result = APIResponse.model_validate(response_data)
# result.user_name == 'Alice'
# result.created_at == '2024-01-15'
```

## Key Takeaways
1. Use `validation_alias` and `serialization_alias` when input and output names differ — `alias` alone isn't enough
2. `AliasPath` and `AliasChoices` handle nested and variable external data without model restructuring
3. `validate_by_name=True` is essential when you want to accept both field names and aliases
4. `alias_generator` is powerful for systematic naming conversion but doesn't override explicit per-field aliases

## Connects To
- **Ch 6**: Aliases affect field names in generated JSON Schema
- **Ch 7**: `validate_by_name` and `alias_generator` are ConfigDict options
- **Ch 8**: Discriminator fields may need aliases to match external API names
- **Ch 10**: Strict mode doesn't affect alias resolution but may affect type validation after alias lookup
