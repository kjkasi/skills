# Patterns & Techniques

## Pattern: Reusable Type Aliases with Annotated
**When to use**: You need the same constraint across multiple fields.
**How**: Create a type alias using `Annotated` with `Field` constraints.
```python
from typing import Annotated
from pydantic import Field
PositiveInt = Annotated[int, Field(gt=0)]
```
**Trade-offs**: Reusable and type-checker friendly, but cannot include field-specific metadata like `alias`.

## Pattern: Discriminated Union for Polymorphism
**When to use**: A field can be one of several types, and you have a distinguishing tag.
**How**: Use `Literal` types on a discriminator field and `Field(discriminator='tag')`.
```python
from typing import Literal
from pydantic import BaseModel, Field
class Cat(BaseModel):
    pet_type: Literal['cat']
class Dog(BaseModel):
    pet_type: Literal['dog']
class Model(BaseModel):
    pet: Cat | Dog = Field(discriminator='pet_type')
```
**Trade-offs**: Fastest union validation; clean JSON Schema; requires a uniform tag field or callable.

## Pattern: Separate Validation and Serialization Aliases
**When to use**: API uses different field names for input vs output.
**How**: Use `validation_alias` and `serialization_alias` on `Field()`.
```python
from pydantic import BaseModel, Field
class User(BaseModel):
    name: str = Field(validation_alias='username', serialization_alias='user_name')
```
**Trade-offs**: Maximum flexibility; type checkers may not recognize aliases in `__init__`.

## Pattern: Nested Model Composition
**When to use**: Data has hierarchical structure.
**How**: Use models as field types; Pydantic validates recursively.
```python
from pydantic import BaseModel
class Address(BaseModel):
    street: str
    city: str
class User(BaseModel):
    name: str
    address: Address
```
**Trade-offs**: Clean separation; deep nesting can impact performance.

## Pattern: Configurable Extra Fields
**When to use**: You want to control how unexpected input data is handled.
**How**: Set `extra` in `ConfigDict`: 'ignore' (default), 'forbid', or 'allow'.
```python
from pydantic import BaseModel, ConfigDict
class StrictModel(BaseModel):
    model_config = ConfigDict(extra='forbid')
```
**Trade-offs**: 'forbid' catches typos; 'allow' preserves extra data; 'ignore' is silent.

## Pattern: Model for API Contracts
**When to use**: Define request/response schemas for APIs.
**How**: Create separate Input/Output models with appropriate fields and constraints.
```python
from pydantic import BaseModel, Field
class UserCreate(BaseModel):
    name: str = Field(min_length=1)
    email: str
class UserResponse(BaseModel):
    id: int
    name: str
    email: str
```
**Trade-offs**: Clear contract; separate models prevent leaking internal fields.

## Pattern: Generic Response Models
**When to use**: API responses share a common wrapper structure.
**How**: Use `Generic[T]` with Pydantic models.
```python
from typing import TypeVar, Generic
from pydantic import BaseModel
T = TypeVar('T')
class ApiResponse(BaseModel, Generic[T]):
    data: T
    success: bool = True
```
**Trade-offs**: Reusable wrapper; requires `model_rebuild()` for nested generics.

## Pattern: Dynamic Model Creation
**When to use**: Model structure is determined at runtime.
**How**: Use `create_model()` with field definitions.
```python
from pydantic import create_model, Field
DynamicModel = create_model('DynamicModel', name=(str, ...), age=(int, Field(ge=0)))
```
**Trade-offs**: Flexible; loses static type checking and IDE support.

## Pattern: Before Validator for Input Normalization
**When to use**: Raw input needs transformation before type validation.
**How**: Use `BeforeValidator` or `@field_validator(mode='before')`.
```python
from typing import Annotated, Any
from pydantic import BaseModel, BeforeValidator
def ensure_list(v: Any) -> Any:
    return v if isinstance(v, list) else [v]
class Model(BaseModel):
    items: Annotated[list[int], BeforeValidator(ensure_list)]
```
**Trade-offs**: Flexible; must handle all possible input types; use `Any` type hint.

## Pattern: Strict Types for API Boundaries
**When to use**: You want to reject coercion at API boundaries.
**How**: Use `StrictInt`, `StrictFloat`, `StrictStr`, `StrictBool` or `Field(strict=True)`.
```python
from pydantic import BaseModel, StrictInt
class ApiInput(BaseModel):
    id: StrictInt  # rejects '123', accepts only int
```
**Trade-offs**: Prevents surprising coercion; requires callers to send correct types.

## Pattern: Nested AliasResolution
**When to use**: Input data has nested/irregular structure.
**How**: Use `AliasPath` or `AliasChoices` for complex field locations.
```python
from pydantic import BaseModel, Field, AliasChoices
class Model(BaseModel):
    ids: list[int] = Field(validation_alias=AliasChoices('ids', 'data.ids'))
```
**Trade-offs**: Handles complex APIs; adds complexity; test thoroughly.

## Pattern: Model Config for ORM Integration
**When to use**: Validating data from ORM objects or similar.
**How**: Set `from_attributes=True` in ConfigDict.
```python
from pydantic import BaseModel, ConfigDict
class UserSchema(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    name: str
    age: int
```
**Trade-offs**: Enables `model_validate(orm_object)`; requires attribute access on input.

## Pattern: Computed Fields
**When to use**: A field value depends on other fields.
**How**: Use `@computed_field` decorator with `@property`.
```python
from pydantic import BaseModel, computed_field
class Rectangle(BaseModel):
    width: float
    height: float
    @computed_field
    def area(self) -> float:
        return self.width * self.height
```
**Trade-offs**: Read-only; included in serialization; not stored in model data.

## Pattern: Context-Aware Validation
**When to use**: Validation logic depends on runtime context (e.g., user role, environment).
**How**: Pass `context` dict to `model_validate()` and access via `ValidationInfo`.
```python
from pydantic import BaseModel, ValidationInfo, field_validator
class Model(BaseModel):
    @field_validator('field')
    @classmethod
    def check_context(cls, v, info: ValidationInfo):
        if info.context and info.context.get('strict'):
            # strict validation
            pass
        return v
```
**Trade-offs**: Powerful; couples validation to runtime state.

## Pattern: Selective Serialization
**When to use**: Different output formats for different consumers.
**How**: Use `include`/`exclude`/`exclude_unset`/`exclude_none` in `model_dump()`.
```python
model.model_dump(exclude_none=True)  # skip None values
model.model_dump(include={'name', 'email'})  # only these fields
model.model_dump(exclude={'password'})  # all except password
```
**Trade-offs**: Flexible output control; remember to use consistently across API.
