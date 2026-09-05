# Chapter 8: Unions

## Core Idea
Pydantic offers three union validation strategies — left-to-right, smart, and discriminated — each with different tradeoffs in correctness, performance, and clarity.

## Frameworks Introduced
- **Left-to-Right Union**: Tries members in declaration order, first match wins
  - When to use: Rarely — only when order determines priority and you accept ambiguity
  - How: Use `Union[A, B]` or `A | B` without discriminator
- **Smart Union (Default)**: Tries all members, picks the most specific match
  - When to use: When types are distinct but don't share a discriminator field
  - How: Default behavior — no special configuration needed
- **Discriminated Union**: Uses a field value to select the correct member
  - When to use: When union members have a shared tag field — most common and efficient
  - How: Add `Field(discriminator='field_name')` to the union field

## Key Concepts
- **left_to_right mode**: First matching member wins; can produce surprising results with overlapping types
- **smart mode (default)**: Evaluates all members, selects best match by exactness and field count
- **discriminated mode**: Uses a discriminator field to pick the exact member — fastest and most predictable
- **Discriminator types**: String field name, callable function, or `Discriminator` object with `NestedModel`
- **Tag**: Assign custom discriminator values to union members
- **OpenAPI discriminator**: Discriminated unions generate proper `discriminator` mapping in JSON Schema

## Mental Models
1. **Priority vs Precision**: Left-to-right is about priority; smart is about precision; discriminated is about intent
2. **Discriminator as Switch**: Think of the discriminator field as a `switch` statement — it routes to the right branch
3. **Ambiguity Tax**: Overlapping types in unions create ambiguity — discriminated unions eliminate it
4. **Performance Hierarchy**: discriminated > smart > left_to_right in both speed and correctness

## Anti-patterns
- **Relying on left-to-right for type selection**: Overlapping types cause silent misroutes; use discriminated instead
- **Using `Union[int, float]` without thinking**: Smart mode handles this, but explicit `float | int` ordering matters
- **Forgetting discriminator in JSON**: Discriminated unions need the discriminator field in serialized JSON to deserialize correctly
- **Using complex callables as discriminators**: Simple string discriminators are faster and more readable

## Code Examples
```python
from typing import Literal
from pydantic import BaseModel, Field

class Cat(BaseModel):
    pet_type: Literal['cat']
    meows: int

class Dog(BaseModel):
    pet_type: Literal['dog']
    barks: float

class Model(BaseModel):
    pet: Cat | Dog = Field(discriminator='pet_type')

# Model(pet={'pet_type': 'cat', 'meows': 5}) → Cat instance
# Model(pet={'pet_type': 'dog', 'barks': 1.5}) → Dog instance
```
- **What it demonstrates**: Discriminated union with Literal discriminator field

```python
from typing import Literal, Union, Annotated
from pydantic import BaseModel, Field, Tag

class Cat(BaseModel):
    pet_type: Literal['cat'] = Field(default=Field(alias='type'))
    meows: int

class Dog(BaseModel):
    pet_type: Literal['dog'] = Field(default=Field(alias='type'))
    barks: float

class Model(BaseModel):
    pet: Annotated[
        Cat | Dog,
        Field(discriminator='pet_type')
    ]
```
- **What it demonstrates**: Using `Annotated` with discriminator for cleaner field declarations

```python
from pydantic import BaseModel, Discriminator, Tag

def get_discriminator_value(v):
    if isinstance(v, dict):
        return v.get('pet_type')
    return getattr(v, 'pet_type', None)

class Model(BaseModel):
    pet: Annotated[
        Cat | Dog,
        Discriminator(
            get_discriminator_value,
            custom_error_type='invalid_pet',
            custom_error_message='Invalid pet type'
        )
    ]
```
- **What it demonstrates**: Callable discriminator for dynamic type resolution

## Worked Example
Model an API that accepts different payment methods:

```python
from typing import Literal
from pydantic import BaseModel, Field

class CreditCard(BaseModel):
    payment_type: Literal['credit_card']
    card_number: str
    expiry: str

class PayPal(BaseModel):
    payment_type: Literal['paypal']
    email: str

class BankTransfer(BaseModel):
    payment_type: Literal['bank_transfer']
    account_number: str
    routing_number: str

class Order(BaseModel):
    total: float
    payment: CreditCard | PayPal | BankTransfer = Field(discriminator='payment_type')

# Valid: Order(total=99.99, payment={'payment_type': 'credit_card', 'card_number': '4111...', 'expiry': '12/25'})
# Invalid: Order(total=99.99, payment={'payment_type': 'bitcoin'}) → ValidationError
```

## Key Takeaways
1. Discriminated unions are almost always the right choice — they're fast, clear, and produce good schemas
2. Left-to-right mode is a footgun with overlapping types — avoid it unless you've thought through the ambiguity
3. The discriminator field must be a `Literal` type for Pydantic to recognize it
4. Smart mode is the safe default when you can't use discriminated unions

## Connects To
- **Ch 6**: Discriminated unions produce cleaner JSON Schema with discriminator mapping
- **Ch 7**: Config affects union behavior (e.g., `from_attributes` with discriminated unions)
- **Ch 9**: Aliases on discriminator fields affect how unions are validated from different key names
- **Ch 10**: Strict mode changes which union member matches when types overlap
