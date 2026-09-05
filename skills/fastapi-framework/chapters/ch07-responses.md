# Chapter 7: Responses

## Core Idea
FastAPI provides multiple ways to define response models, status codes, headers, and cookies, enabling data filtering, validation, and customization of API responses.

## Frameworks Introduced
- **Response**: Base class for HTTP responses. Use when you need full control over response content.
- **JSONResponse**: Standard JSON response class. Use for most API responses.
- **HTMLResponse**: HTML response class. Use for serving HTML content.

## Key Concepts
- **Response Model**: Defines the structure and validation of response data.
- **Status Code**: HTTP status code indicating success or failure of the request.
- **Response Headers**: Metadata sent with the response (e.g., caching, content type).
- **Cookies**: Data stored in the client's browser for session management.

## Mental Models
- **Use response models to filter output**: Return more data internally but expose only what's needed.
- **Think of status codes as communication**: They tell the client what happened (200 OK, 404 Not Found).
- **Use headers for metadata**: Pass information that's not part of the response body.

## Anti-patterns
- **Don't return sensitive data without filtering**: Use response models to exclude passwords and secrets.
- **Don't use wrong status codes**: Use `201 Created` for successful resource creation, not `200 OK`.
- **Don't forget that cookies have browser restrictions**: JavaScript has limited access to cookies.

## Code Examples
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse, RedirectResponse
from pydantic import BaseModel

app = FastAPI()

class UserOut(BaseModel):
    username: str
    email: str

class UserIn(BaseModel):
    username: str
    password: str
    email: str

@app.post("/users/", response_model=UserOut, status_code=201)
async def create_user(user: UserIn):
    return user

@app.get("/redirect")
async def redirect():
    return RedirectResponse(url="https://fastapi.tiangolo.com")

@app.get("/custom-header")
async def custom_header():
    return JSONResponse(
        content={"message": "Hello World"},
        headers={"X-Custom-Header": "custom-value"}
    )
```

## Reference Tables

| Response Class | Content Type | Use Case |
|---------------|--------------|----------|
| `JSONResponse` | `application/json` | Standard API responses |
| `HTMLResponse` | `text/html` | Serving HTML pages |
| `PlainTextResponse` | `text/plain` | Plain text responses |
| `RedirectResponse` | N/A | URL redirects |
| `StreamingResponse` | Various | Large file downloads |

## Worked Example
Create an API with response filtering:
```python
class UserIn(BaseModel):
    username: str
    password: str
    email: str

class UserOut(BaseModel):
    username: str
    email: str

@app.post("/users/", response_model=UserOut)
async def create_user(user: UserIn):
    # Return the user with password, but response_model filters it out
    return user
```
Even though `user` contains `password`, the response only includes `username` and `email`.

## Key Takeaways
1. Response models automatically filter output data to match the declared type.
2. Use `status_code` parameter in decorators to set appropriate HTTP status codes.
3. Response headers can be added using `JSONResponse` or by returning a `Response` object.
4. Cookies require explicit declaration using `Cookie` parameter to avoid being treated as query parameters.

## Connects To
- **Ch 4**: Request bodies define input, response models define output.
- **Ch 6**: Security responses should filter sensitive data like passwords.
- **Ch 8**: Middleware can modify responses after path operations.
