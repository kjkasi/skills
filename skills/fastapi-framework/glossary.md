# FastAPI Glossary

**APIRouter** — Modular router for grouping related path operations with shared prefix, tags, and dependencies (Ch 9)

**async/await** — Python syntax for asynchronous code that allows non-blocking I/O operations; use `async def` when calling libraries that support `await` (Ch 8)

**Background Tasks** — Functions scheduled to run after sending response, useful for emails, logging, or notifications (Ch 10)

**CORS** — Cross-Origin Resource Sharing; browser security mechanism for cross-domain requests, handled via `CORSMiddleware` (Ch 7)

**cookie** — Client state stored in browser, set via `Set-Cookie` header; FastAPI provides `Cookie()` parameter helper (Ch 7)

**dependency injection** — Pattern where FastAPI resolves and injects dependencies (functions/classes) into path operations automatically (Ch 5)

**Depends** — Function used to declare dependencies in path operation parameters or decorators (Ch 5)

**Docker** — Container platform for deploying FastAPI applications with consistent environments (Ch 10)

**error handling** — Custom exception handlers for `HTTPException`, `ValidationError`, and application-specific exceptions (Ch 7)

**fastapi** — Modern Python web framework for building APIs with automatic documentation, validation, and type hints (Ch 1)

**file upload** — Sending files via multipart form data using `UploadFile` or `bytes` type parameters (Ch 4)

**form data** — URL-encoded or multipart data sent in request body, used for file uploads and form submissions (Ch 4)

**gunicorn** — WSGI HTTP server for UNIX; use with Uvicorn workers for production multi-process deployment (Ch 10)

**header** — HTTP request/response metadata; FastAPI provides `Header()` parameter helper for extraction (Ch 7)

**HTTPS** — Secure HTTP protocol; use reverse proxy (Nginx/Traefik) for TLS termination in production (Ch 10)

**JSON Schema** — Standard for describing JSON data structure; auto-generated from Pydantic models for OpenAPI (Ch 1)

**JWT** — JSON Web Token; compact, URL-safe token format for securely transmitting information between parties (Ch 6)

**middleware** — Code that runs before/after request processing, e.g., CORS, logging, authentication (Ch 7)

**OAuth2** — Authorization framework for token-based authentication; FastAPI provides `OAuth2PasswordBearer` and related utilities (Ch 6)

**OpenAPI** — Specification for describing APIs; FastAPI auto-generates this from code for interactive documentation (Ch 1)

**path operation** — Function decorated with `@app.get()`, `@app.post()`, etc., handling specific HTTP requests (Ch 1)

**path parameter** — Variable extracted from URL path, e.g., `/items/{item_id}`; supports type conversion and validation (Ch 2)

**Pydantic** — Data validation library using Python type hints; defines request/response models for automatic validation (Ch 1)

**query parameter** — URL query string parameters, e.g., `?q=search`; supports defaults and validation (Ch 3)

**request body** — JSON data sent in HTTP request body; validated against Pydantic model (Ch 4)

**response model** — Pydantic model defining response structure; filters output data and generates OpenAPI schema (Ch 7)

**reverse proxy** — Server (Nginx/Traefik) sitting in front of FastAPI; handles HTTPS, load balancing, and forwarded headers (Ch 10)

**Server-Sent Events** — HTTP streaming from server to client using `EventSourceResponse` with `yield` for real-time updates (Ch 8)

**Starlette** — ASGI toolkit underlying FastAPI; provides routing, requests, responses, and middleware (Ch 1)

**static files** — Static assets (CSS, JS, images) served via `StaticFiles` mount (Ch 7)

**status code** — HTTP response code (200, 404, 500, etc.); set via `status_code` parameter or `Response` class (Ch 7)

**template** — Server-rendered HTML using Jinja2 engine; FastAPI provides `Jinja2Templates` for rendering (Ch 8)

**test client** — `TestClient` class (based on HTTPX) for testing FastAPI applications without actual HTTP connections (Ch 10)

**uvicorn** — ASGI server for running FastAPI applications; supports async and worker processes (Ch 10)

**validation** — Automatic data validation using Pydantic models and type hints; returns 422 errors for invalid data (Ch 1)

**WebSocket** — Full-duplex communication protocol for real-time bidirectional messaging between client and server (Ch 8)
