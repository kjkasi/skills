# FastAPI Patterns

## Path Operation Decorators
**When to use**: Every API endpoint needs HTTP method and path definition
**How**: Use `@app.get()`, `@app.post()`, `@app.put()`, `@app.delete()` with path string; add `response_model`, `status_code`, `tags` parameters
**Trade-offs**: Provides automatic OpenAPI generation and validation but requires defining Pydantic models for full documentation benefits

## Dependency Injection
**When to use**: Shared logic like authentication, database connections, or parameter validation across multiple endpoints
**How**: Create callable function/class, declare parameter as `Depends(your_dependency)`; supports sub-dependencies and caching
**Trade-offs**: Reduces code duplication but adds indirection; use `use_cache=False` when fresh values needed per request

## Response Model Filtering
**When to use**: Control API response shape, hide sensitive fields (e.g., passwords), or return different models for input/output
**How**: Set `response_model=YourModel` in decorator; FastAPI filters output data to match declared fields
**Trade-offs**: Adds serialization overhead but ensures consistent API contracts and security; use `response_model_exclude_unset=True` to omit defaults

## Async Endpoints
**When to use**: I/O-bound operations (database queries, API calls, file operations) that benefit from non-blocking execution
**How**: Declare with `async def` and use `await` for async libraries; use regular `def` for blocking code (runs in threadpool)
**Trade-offs**: Better concurrency for I/O-bound tasks but requires async-compatible libraries; mixing `def`/`async def` is safe

## Background Tasks
**When to use**: Post-response operations like sending emails, logging, or notifications that shouldn't delay client response
**How**: Declare `BackgroundTasks` parameter, call `background_tasks.add_task(func, *args)` in path operation
**Trade-offs**: Improves perceived performance but tasks may fail silently; not suitable for critical operations requiring confirmation

## Middleware Chains
**When to use**: Cross-cutting concerns like CORS, authentication, logging, or request/response transformation
**How**: Use `app.add_middleware(MiddlewareClass, **kwargs)`; middleware executes in order (last added runs first)
**Trade-offs**: Applies to all requests; use dependencies for endpoint-specific logic; performance impact depends on middleware complexity

## OAuth2 with JWT
**When to use**: Stateless authentication with token-based access control and optional scopes
**How**: Use `OAuth2PasswordBearer` for token extraction, validate JWT with `python-jose` or `PyJWT`, store user data in token
**Trade-offs**: Stateless and scalable but requires secure token storage and refresh mechanism; scopes add complexity

## SQL Database Integration
**When to use**: Persistent data storage with relational databases using SQLAlchemy or SQLModel
**How**: Create async/sync engine, use `yield` dependency for session management, define models with `Base` class
**Trade-offs**: `yield` dependencies handle session lifecycle but require careful resource management; async drivers needed for async endpoints

## File Uploads
**When to use**: Accepting file attachments from clients (images, documents, data files)
**How**: Declare parameter as `UploadFile` or `bytes`, use `File()` for metadata; supports multiple files and form data mixing
**Trade-offs**: `UploadFile` provides async methods and file metadata but requires careful memory management for large files

## WebSocket Handling
**When to use**: Real-time bidirectional communication (chat, live updates, collaborative editing)
**How**: Use `@app.websocket("/path")`, accept connection, `await` messages, send responses; handle `WebSocketDisconnect` exception
**Trade-offs**: Persistent connections increase server resource usage; not HTTP-cacheable; use SSE for server-to-client streaming

## Template Rendering
**When to use**: Server-side HTML rendering with Jinja2 for traditional web applications
**How**: Create `Jinja2Templates` instance, return `TemplateResponse` with template name, request, and context dict
**Trade-offs**: Combines well with FastAPI but adds server-side rendering overhead; use for SSR or legacy apps, not SPAs

## Custom OpenAPI Schema
**When to use**: Extending API documentation with custom responses, security schemes, or metadata
**How**: Use `responses` parameter in decorators, define `openapi_tags`, or override `openapi()` method
**Trade-offs**: Enhances documentation but requires maintenance; keep customizations minimal to avoid drift from implementation

## Sub-Applications
**When to use**: Modular architecture with independent components (admin panel, API versions, microservices)
**How**: Mount using `app.mount("/sub", sub_app)` or `APIRouter` with `prefix` for logical grouping
**Trade-offs**: Promotes code organization but mounted apps have separate OpenAPI schemas; routers share schema

## Behind a Proxy
**When to use**: Production deployments with load balancers (Nginx, Traefik) handling HTTPS and routing
**How**: Enable `--forwarded-allow-ips` for trusted proxy IPs; set `--proxy-headers` for correct URL generation
**Trade-offs**: Required for HTTPS and load balancing but adds complexity; ensure proxy headers are trusted only from known sources
