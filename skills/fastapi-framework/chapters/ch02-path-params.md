# Chapter 2: Path Parameters

## Core Idea
Path parameters are variables extracted from the URL path and validated using Python type annotations. They enable type conversion, validation, and automatic documentation for URL segments.

## Frameworks Introduced
- **Enum**: Python enumeration class. Use when you need predefined valid values for path parameters.
- **Pydantic**: Handles data validation and serialization. Use for complex validation scenarios.

## Key Concepts
- **Path Parameter**: A variable in the URL path like `/items/{item_id}` that gets passed to your function.
- **Type Conversion**: FastAPI automatically converts string path parameters to declared Python types.
- **Data Validation**: Invalid types return clear error messages with specific location and error details.
- **Predefined Values**: Use Python Enum to restrict path parameter values to a set of valid options.

## Mental Models
- **Use type annotations to validate path parameters**: Declaring `item_id: int` ensures only integers are accepted.
- **Think of path parameters as required by default**: Unlike query parameters, they have no default values.
- **Use Enums when you have a fixed set of valid values**: This provides better documentation and validation.

## Anti-patterns
- **Don't forget path parameter order matters**: Fixed paths like `/users/me` must come before variable paths like `/users/{user_id}`.
- **Don't redefine path operations**: The first matching path operation will always be used.
- **Don't use string types when you need numbers**: Use `int`, `float` for automatic conversion.

## Code Examples
```python
from fastapi import FastAPI
from enum import Enum

app = FastAPI()

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name == ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}
    return {"model_name": model_name, "message": "Have some residuals"}

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

## Reference Tables

| Path Pattern | Example URL | Parameter Type | Use Case |
|-------------|-------------|----------------|----------|
| `/items/{item_id}` | `/items/3` | `int` | Numeric IDs |
| `/users/{user_id}` | `/users/abc` | `str` | String identifiers |
| `/files/{file_path:path}` | `/files/home/johndoe/file.txt` | `str` | File paths with slashes |
| `/models/{model_name}` | `/models/alexnet` | `Enum` | Predefined values |

## Worked Example
Create an API with enum validation:
```python
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name == ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}
    return {"model_name": model_name, "message": "Have some residuals"}
```
Accessing `/models/alexnet` returns the enum member, while `/models/invalid` returns a validation error.

## Key Takeaways
1. Path parameters are automatically validated and converted to declared types.
2. Use `str` Enum classes to restrict path parameters to predefined valid values.
3. The `:path` converter allows path parameters to contain slashes (e.g., file paths).
4. Order matters: declare fixed paths before variable paths to avoid conflicts.

## Connects To
- **Ch 1**: Path parameters are the first type of parameter covered in the tutorial.
- **Ch 3**: Query parameters are optional and have defaults, unlike path parameters.
- **Ch 7**: Response models can filter output data from path parameters.
