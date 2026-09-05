# Chapter 20: Examples & Recipes

## Core Theory
Common Pydantic patterns solve recurring problems. These recipes provide battle-tested solutions for real-world scenarios.

## Frameworks Introduced
- **Pattern Library**: Reusable solutions for frequent validation challenges
  - When to use: Implementing common data patterns (APIs, configs, dynamic models)
  - How: Adapt these recipes to your specific requirements
- **Composition Strategy**: Build complex models from simple, validated primitives
  - When to use: Models with nested structures, inheritance, or generic types
  - How: Start with simple models, compose them using field types and inheritance

## Key Concepts
- **Custom validators**: Complex business logic beyond simple type constraints
- **Dynamic model creation**: build models programmatically with create_model()
- **File upload validation**: Validate file types, sizes, and contents
- **Environment settings**: pydantic-settings for typed configuration from env vars
- **API contracts**: Request/response models for type-safe APIs
- **Nested models**: Composing models with complex field types
- **Generic models**: Type-parameterized models for reusable patterns
- **Model inheritance**: Extending base models with additional fields and validators

## Mental Models
- Models are data contracts: define what data looks like, let Pydantic enforce it
- Composition over complexity: build complex models from simple, validated parts
- Validators are business rules: they encode domain logic in a declarative way
- Generic models are templates: they define patterns that work with any type

## Anti-patterns
- **Over-validating**: Don't validate everything; focus on external boundaries
- **Ignoring model inheritance**: Reuse base models instead of duplicating fields
- **Skipping edge cases**: Use custom validators for complex business rules
- **Hardcoding configuration**: Use pydantic-settings for environment-specific values

## Code Examples
```python
from pydantic import BaseModel, create_model, Field

# Dynamic model
DynamicModel = create_model(
    'DynamicModel',
    name=(str, Field(...)),
    age=(int, Field(ge=0)),
)

# Generic model
from typing import TypeVar, Generic
T = TypeVar('T')
class Response(BaseModel, Generic[T]):
    data: T
    success: bool = True
```
- **What it demonstrates**: Dynamic model creation and generic model patterns

## Worked Example
Building a type-safe API with nested models:
1. Define base User model with common fields
2. Create UserCreate, UserRead, UserUpdate models inheriting from User
3. Use generic Response[T] for consistent API responses
4. Add custom validators for business rules (e.g., email format, age ranges)
5. Use create_model() for admin-specific fields at runtime

## Key Takeaways
1. Use create_model() for runtime-generated validation schemas
2. Generic models enable reusable, type-safe patterns
3. Model inheritance reduces duplication while maintaining type safety
4. Custom validators encode complex business rules declaratively

## Connects To
- **Ch 1**: Basic model definition
- **Ch 3**: Field constraints and validation
- **Ch 5**: Pydantic Settings for configuration
- **Ch 19**: Integration with FastAPI and other tools
