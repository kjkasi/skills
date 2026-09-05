# Chapter 17: Architecture

## Core Idea
Pydantic V2 is a Rust-powered validation engine wrapped in a Python interface. Understanding the architecture explains why it's fast and how to debug complex validation issues.

## Frameworks Introduced
- **Schema Compilation Pipeline**: Type annotations → Python parsing → Core schema generation → Rust validation
  - When to use: Understanding validation performance bottlenecks
  - How: Inspect __pydantic_core_schema__ to see compiled validation logic
- **Two-Phase Model Building**: Class definition phase (Python) followed by schema compilation phase (Rust)
  - When to use: Debugging model creation issues
  - How: Understand that model_rebuild() triggers the second phase

## Key Concepts
- **Core schema**: JSON-like structure defining validation logic; compiled into efficient Rust code
- **SchemaValidator**: Rust-based engine that executes validation against core schemas
- **SchemaSerializer**: Rust-based engine for converting validated data back to Python/JSON
- **pydantic-core**: The Rust library that performs all validation and serialization
- **Model building**: Automatic process converting type annotations into executable core schemas
- **Validation flow**: Raw input → core schema validation → validated Python objects
- **Schema rebuilding**: model_rebuild() recompiles the core schema (needed for forward refs)
- **Type resolution**: Process of resolving string annotations to actual type objects

## Mental Models
- Pydantic is a compiler: it compiles type annotations into optimized validation code
- The core schema is the blueprint: it describes what validation to perform
- Rust does the heavy lifting: Python handles model definition, Rust handles execution
- Schema rebuilding is recompilation: when types change, the schema must be regenerated

## Anti-patterns
- **Assuming Python-speed validation**: pydantic-core runs in Rust; bottlenecks are usually in Python callbacks
- **Ignoring core schema**: Complex validation issues require inspecting the generated schema
- **Unnecessary model_construct**: Bypasses validation; only use when you're certain data is valid

## Code Examples
```python
from pydantic import BaseModel

class Model(BaseModel):
    x: int

# Internally: Model.__pydantic_core_schema__ contains compiled validation logic
print(Model.__pydantic_core_schema__)
```
- **What it demonstrates**: Inspecting the compiled validation schema

## Worked Example
A model with complex nested types takes longer to validate than expected. By inspecting __pydantic_core_schema__, you discover the schema includes deep recursion. Simplifying the type hierarchy or using model_config = ConfigDict(validate_default=True) optimizes the compiled schema.

## Key Takeaways
1. Pydantic V2 uses Rust (pydantic-core) for all validation and serialization
2. The core schema is the intermediate representation between Python types and validation logic
3. model_rebuild() triggers recompilation of the core schema
4. Most performance issues come from Python callbacks, not Rust validation

## Connects To
- **Ch 1**: Basic model definition
- **Ch 16**: Troubleshooting schema compilation issues
- **Ch 18**: Migration differences in schema handling
