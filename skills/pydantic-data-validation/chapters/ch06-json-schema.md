# Chapter 6: JSON Schema

## Core Idea
Pydantic automatically generates JSON Schema from model definitions, providing a machine-readable contract for your data that integrates with documentation tools, code generators, and validation libraries.

## Frameworks Introduced
- **JSON Schema Generation**: Converts Pydantic models into standard JSON Schema format
  - When to use: API documentation, code generation, interop with non-Python systems
  - How: Call `model_json_schema()` on any model class
- **Schema Modes**: Two perspectives for schema generation
  - When to use: `validation` mode for input schemas, `serialization` mode for output schemas
  - How: Pass `mode='validation'` or `mode='serialization'` to `model_json_schema()`

## Key Concepts
- **model_json_schema()**: Returns a dict representing the JSON Schema of a model
- **$defs**: Section containing shared/referenced schemas for nested models
- **WithJsonSchema**: Attach custom JSON schema metadata to individual fields
- **JsonSchemaExamples**: Add example values to schema for documentation
- **Discriminator in Schema**: Represent union type selection in the schema for OpenAPI
- **model_config schema customization**: Override title, description, and schema behavior via config

## Mental Models
1. **Schema as Contract**: JSON Schema defines the shape of valid data — treat it as your API's source of truth
2. **Two Lenses**: Validation schema describes what you accept; serialization schema describes what you produce
3. **Ref Reuse**: Nested models become `$ref` entries in `$defs`, keeping schemas DRY
4. **Schema ≠ Code**: JSON Schema is a data format, not executable — it describes constraints but doesn't enforce them

## Anti-patterns
- **Ignoring mode parameter**: Using validation mode when you need serialization schema leads to incorrect field visibility
- **Manual schema editing**: Always generate from models — manual edits diverge from the source of truth
- **Overriding schema without WithJsonSchema**: Using model_config overrides affects all fields; use WithJsonSchema for per-field control

## Code Examples
```python
from pydantic import BaseModel, WithJsonSchema

class Model(BaseModel):
    name: str
    age: int

schema = Model.model_json_schema()
# Returns: {'properties': {'name': {'title': 'Name', 'type': 'string'}, 'age': {'title': 'Age', 'type': 'integer'}}, 'required': ['name', 'age'], 'title': 'Model', 'type': 'object'}
```
- **What it demonstrates**: Basic JSON Schema generation from a Pydantic model

```python
from pydantic import BaseModel, Field, WithJsonSchema

class Model(BaseModel):
    name: str = Field(
        json_schema_extra={'examples': ['Alice'], 'description': 'User name'}
    )
    age: int = Field(ge=0)

schema = Model.model_json_schema()
# 'name' includes examples and description in schema
```
- **What it demonstrates**: Adding documentation metadata to schema fields

```python
from typing import Literal, Union
from pydantic import BaseModel, Field

class Cat(BaseModel):
    pet_type: Literal['cat']

class Dog(BaseModel):
    pet_type: Literal['dog']

class Model(BaseModel):
    pet: Cat | Dog = Field(discriminator='pet_type')

schema = Model.model_json_schema()
# Schema includes discriminator mapping for the union
```
- **What it demonstrates**: Discriminator representation in generated JSON Schema

## Worked Example
Generate a schema for an API endpoint accepting user registration data:

```python
from pydantic import BaseModel, Field, ConfigDict

class Address(BaseModel):
    street: str
    city: str
    zip_code: str = Field(pattern=r'^\d{5}$')

class UserRegistration(BaseModel):
    model_config = ConfigDict(title='User Registration')
    username: str = Field(min_length=3, max_length=20, json_schema_extra={'examples': ['alice123']})
    email: str
    address: Address
    role: Literal['admin', 'user'] = 'user'

schema = UserRegistration.model_json_schema()
# Address appears in $defs as a referenced schema
# title is 'User Registration' from model_config
```

## Key Takeaways
1. `model_json_schema()` is the single entry point — no manual schema authoring needed
2. Use `mode='serialization'` when documenting output shapes; default `mode='validation'` is for input
3. `$defs` keeps schemas DRY by referencing nested models instead of inlining
4. Attach examples and descriptions via `Field(json_schema_extra=...)` for rich documentation

## Connects To
- **Ch 7**: ConfigDict options like `title` and `json_schema_extra` affect schema output
- **Ch 8**: Discriminated unions produce cleaner schema representations than plain unions
- **Ch 9**: Alias configuration changes field names in schema output
- **Ch 10**: Strict mode affects type coercion but not schema shape
