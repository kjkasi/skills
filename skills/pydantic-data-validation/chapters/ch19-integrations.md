# Chapter 19: Integrations

## Core Idea
Pydantic's ecosystem extends far beyond standalone validation. Integration with frameworks and tools amplifies its value across the entire development stack.

## Frameworks Introduced
- **Framework Integration Map**: Pydantic connects to web frameworks, ORMs, testing, and developer tools
  - When to use: Building production applications with multiple dependencies
  - How: Use native integration points rather than manual conversion
- **Validation Layer Architecture**: Pydantic models serve as the single source of truth for data contracts
  - When to use: APIs, database models, configuration, and inter-service communication
  - How: Define Pydantic models first, then derive other representations (DB, API, config)

## Key Concepts
- **FastAPI integration**: Native request/response validation via Pydantic models in function signatures
- **mypy plugin**: Enhanced type checking for model fields and validators
- **dataclasses integration**: Validate stdlib dataclasses with validate_dataclass()
- **ORM integration**: from_attributes=True enables automatic ORM object conversion
- **Hypothesis integration**: Generate test data automatically from model schemas
- **Rich integration**: Beautiful, readable error output for development
- **Logfire integration**: Observability for validation errors in production
- **Pyright integration**: VS Code type checking via Pyright language server
- **AWS Lambda**: Lightweight validation without FastAPI overhead

## Mental Models
- Pydantic models are data contracts: they define what data looks like across all boundaries
- Integrations are adapters: they translate between Pydantic's world and external tools
- Single source of truth: define data structure once, use everywhere
- Type safety flows through the stack: Pydantic → mypy → FastAPI → database

## Anti-patterns
- **Manual conversion**: Don't convert between Pydantic and ORM objects manually; use from_attributes
- **Ignoring mypy plugin**: Missing type errors that could be caught at development time
- **Skipping validation in tests**: Use Hypothesis to generate edge cases you didn't think of
- **Not using native FastAPI features**: Don't parse request bodies manually; let FastAPI handle it

## Code Examples
```python
# FastAPI integration
from fastapi import FastAPI
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

app = FastAPI()

@app.post("/users")
def create_user(user: User):
    return {"name": user.name}
```
- **What it demonstrates**: Automatic request validation via Pydantic model in FastAPI

## Worked Example
Building a REST API with SQLAlchemy ORM:
1. Define Pydantic model with fields matching database columns
2. Set model_config = ConfigDict(from_attributes=True)
3. Use model_validate(orm_object) to convert ORM objects to Pydantic
4. Use .model_dump() to convert Pydantic objects back to dicts for DB operations
5. FastAPI automatically validates requests using the Pydantic model

## Key Takeaways
1. FastAPI natively validates request/response using Pydantic models
2. Enable from_attributes=True for automatic ORM object conversion
3. The mypy plugin catches type errors specific to Pydantic models
4. Hypothesis integration generates edge-case test data from model schemas

## Connects To
- **Ch 1**: Basic model definition
- **Ch 5**: Pydantic Settings for configuration
- **Ch 16**: Troubleshooting integration issues
- **Ch 20**: Practical recipes combining integrations
