# Chapter 5: Dependencies

## Core Idea
Dependency injection is a pattern where FastAPI automatically executes shared logic before path operations, handling database connections, authentication, validation, and other common tasks.

## Frameworks Introduced
- **Depends**: FastAPI class for declaring dependencies that should be executed before path operations.
- **APIRouter**: Use for organizing dependencies across multiple files and modules.

## Key Concepts
- **Dependency**: A function that FastAPI executes before your path operation, providing shared logic.
- **Dependency Injection**: FastAPI "injects" the results of dependencies into your path operation functions.
- **Sub-dependencies**: Dependencies that depend on other dependencies, forming a hierarchy.
- **Yield Dependencies**: Dependencies that need cleanup code, using `yield` to split setup and teardown.

## Mental Models
- **Use dependencies for shared logic**: When multiple endpoints need the same validation or database access.
- **Think of dependencies as pre-processors**: They run before path operations and can modify requests.
- **Use yield dependencies for resource management**: Perfect for database connections that need closing.

## Anti-patterns
- **Don't call dependency functions directly**: Pass them to `Depends()` without calling them.
- **Don't create complex dependencies when simple code suffices**: Dependencies are for shared logic.
- **Don't forget that dependencies can be async**: Use `async def` for I/O-bound operations.

## Code Examples
```python
from fastapi import FastAPI, Depends, HTTPException, status

app = FastAPI()

async def common_parameters(q: str | None = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons

@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons

async def verify_token(x_token: str = Header()):
    if x_token != "fake-super-secret-token":
        raise HTTPException(status_code=400, detail="X-Token header invalid")

async def verify_key(x_api_key: str = Header()):
    if x_api_key != "fake-super-secret-api-key":
        raise HTTPException(status_code=400, detail="X-API-Key header invalid")

@app.get("/items/", dependencies=[Depends(verify_token), Depends(verify_key)])
async def read_items():
    return [{"item": "Foo"}, {"item": "Bar"}]
```

## Reference Tables

| Dependency Type | Purpose | Use Case |
|----------------|---------|----------|
| Simple function | Shared validation | Common query parameters |
| Async function | I/O operations | Database queries, API calls |
| Yield dependency | Resource management | Database connections, file handles |
| Class dependency | Complex state | Authentication systems |

## Worked Example
Create a dependency for database simulation:
```python
async def get_db():
    db = DBSessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
async def read_users(db: Session = Depends(get_db)):
    users = db.query(User).all()
    return users
```
The `yield` ensures the database connection is properly closed after the request.

## Key Takeaways
1. Dependencies are functions that FastAPI executes before path operations.
2. Use `Depends()` to declare dependencies in path operation parameters.
3. Yield dependencies allow cleanup code to run after the response is sent.
4. Dependencies can depend on other dependencies, forming a hierarchical system.

## Connects To
- **Ch 6**: Security dependencies handle authentication and authorization.
- **Ch 7**: Dependencies can modify response headers and status codes.
- **Ch 8**: Background tasks run after dependencies complete.
