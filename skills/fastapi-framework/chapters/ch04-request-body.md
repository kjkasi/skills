# Chapter 4: Request Body

## Core Idea
Request bodies contain data sent from the client to the API, typically for creating or updating resources. FastAPI uses Pydantic models to validate and serialize request data automatically.

## Frameworks Introduced
- **Pydantic BaseModel**: Base class for defining data models with type validation.
- **Body**: FastAPI class for explicit body parameter declaration and validation.

## Key Concepts
- **Request Body**: Data sent by the client in the HTTP request body, typically as JSON.
- **Pydantic Model**: A class that inherits from `BaseModel` and defines data structure with types.
- **Nested Models**: Models that contain other models as fields for complex data structures.
- **Body Updates**: Using `PUT` or `PATCH` to update existing resources with partial data.

## Mental Models
- **Use Pydantic models for complex data**: When you need multiple fields with validation, create a model.
- **Think of models as contracts**: They define what data your API expects and returns.
- **Use response models to filter output**: Return more data internally but expose only what's needed.

## Anti-patterns
- **Don't send request bodies with GET requests**: This has undefined behavior in HTTP specifications.
- **Don't return sensitive data in responses**: Use response models to filter passwords and secrets.
- **Don't forget that optional fields need defaults**: Use `None` or specific default values.

## Code Examples
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.model_dump()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.model_dump()}

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

## Reference Tables

| HTTP Method | Purpose | Body Required | Use Case |
|-------------|---------|---------------|----------|
| `GET` | Read data | No | Fetch resources |
| `POST` | Create data | Yes | Create new resources |
| `PUT` | Update data | Yes | Replace entire resource |
| `PATCH` | Partial update | Yes | Update specific fields |
| `DELETE` | Delete data | No | Remove resources |

## Worked Example
Create an API with request body validation:
```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.model_dump()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```
Send JSON body:
```json
{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```
Response includes computed `price_with_tax` field.

## Key Takeaways
1. Pydantic models provide automatic validation, serialization, and documentation.
2. Fields with defaults (`= None`) are optional in the request body.
3. Use `model_dump()` to convert Pydantic models to dictionaries.
4. FastAPI automatically generates JSON Schema for request bodies in OpenAPI docs.

## Connects To
- **Ch 2-3**: Path and query parameters handle simple data, request bodies handle complex data.
- **Ch 5**: Dependencies can validate request bodies before path operations.
- **Ch 7**: Response models filter what data is returned to clients.
