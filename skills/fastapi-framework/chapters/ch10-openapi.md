# Chapter 10: OpenAPI Schema

## Core Idea
FastAPI automatically generates an OpenAPI schema from your code, enabling interactive documentation, client generation, and integration with API tools.

## Frameworks Introduced
- **OpenAPI**: Standard for describing APIs. Use for documentation, testing, and client generation.
- **Swagger UI**: Interactive API documentation. Use at `/docs` for testing endpoints.
- **ReDoc**: Alternative API documentation. Use at `/redoc` for clean documentation.

## Key Concepts
- **OpenAPI Schema**: Machine-readable description of your API endpoints, parameters, and responses.
- **JSON Schema**: Standard for describing JSON data structures used in OpenAPI.
- **Callbacks**: Document how your API calls external APIs (webhooks).
- **Webhooks**: Notifications your API sends to external systems.
- **operationId**: Unique identifier for each endpoint in the OpenAPI schema.

## Mental Models
- **Think of OpenAPI as your API's contract**: It defines what your API does and how to use it.
- **Use tags to organize endpoints**: Group related endpoints in the documentation.
- **Document callbacks to help external developers**: Show how they should implement webhook receivers.

## Anti-patterns
- **Don't skip documentation**: Good OpenAPI docs make your API usable.
- **Don't use vague descriptions**: Be specific about what each endpoint does.
- **Don't forget to test with the generated docs**: They reveal issues with your API design.

## Code Examples
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(
    title="My API",
    description="This is a very fancy API",
    version="0.1.0",
)

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.get(
    "/items/{item_id}",
    response_model=Item,
    summary="Get an item",
    description="Get an item by its ID",
    response_description="The item",
    tags=["items"],
)
async def read_item(item_id: int):
    """
    Get an item by its ID.

    With this you can:
    * Get an item
    * See its name
    * Ensure it has a description
    """
    return {"item_id": item_id}

@app.get("/users/", tags=["users"])
async def get_users():
    return ["Rick", "Morty"]
```

## Reference Tables

| OpenAPI Feature | Purpose | Use Case |
|----------------|---------|----------|
| Tags | Group related endpoints | Organize documentation |
| Summary | Short endpoint description | Quick overview in docs |
| Description | Detailed explanation | Full documentation |
| Response model | Define response structure | Automatic docs and validation |
| Callbacks | Document webhook APIs | Help external developers |

## Worked Example
Document a webhook callback:
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Invoice(BaseModel):
    id: str
    customer: str
    total: float

class InvoiceEvent(BaseModel):
    description: str
    paid: bool

invoices_callback_router = APIRouter()

@invoices_callback_router.post("/{$callback_url}/invoices/{$request.body.id}")
async def invoice_callback(body: InvoiceEvent):
    pass  # No actual code needed, this is for documentation

@app.post("/invoices/", callbacks=invoices_callback_router.routes)
async def create_invoice(invoice: Invoice):
    return invoice
```
The callback documentation shows external developers how to implement the webhook receiver.

## Key Takeaways
1. FastAPI automatically generates OpenAPI schema from your code.
2. Use tags, summaries, and descriptions to improve documentation quality.
3. The `/docs` endpoint provides interactive Swagger UI for testing.
4. Callbacks help document webhook APIs for external developers.
5. Client generation tools can use the OpenAPI schema to create typed clients.

## Connects To
- **Ch 1**: OpenAPI is generated automatically from the first endpoint.
- **Ch 7**: Response models define the schema for response documentation.
- **Ch 6**: Security schemes are documented in the OpenAPI schema.
