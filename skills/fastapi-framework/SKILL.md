---
name: fastapi-framework
description: "Knowledge base from \"FastAPI Documentation\" by Sebastián Ramírez. Use when building APIs with FastAPI, applying its dependency injection system, designing Pydantic models, implementing security, deploying services, or referencing FastAPI patterns."
---

<!-- argument-hint: [topic, feature name, or chapter number] -->

# FastAPI Documentation
**Author**: Sebastián Ramírez (tiangolo) | **Chapters**: 10 | **Generated**: 2026-09-03

## How to Use This Skill

- **Without arguments** — load core frameworks for reference
- **With a topic** — ask about `dependencies`, `security`, `pydantic`, or another indexed topic; I find and read the relevant chapter
- **With chapter** — ask for `ch05`; I load that specific chapter
- **Browse** — ask "what chapters do you have?" to see the full index

When you ask about a topic not covered in Core Frameworks below, I will read
the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

- **Dependency Injection (DI)**: FastAPI's central architecture pattern. Use `Depends()` to declare reusable logic (DB sessions, auth checks, validation). Dependencies can be functions, classes, or generators. Nesting is automatic — sub-dependencies resolve transitively. Use yield-based dependencies for setup/teardown (DB connections, transactions).

- **Path Operation Decorators**: `@app.get()`, `@app.post()`, etc. are not just routing — they define the HTTP semantics, generate OpenAPI docs, and enable automatic validation. Each decorator maps one function to one HTTP method + path combination.

- **Pydantic Model Delegation**: FastAPI delegates all data validation to Pydantic. Define `BaseModel` subclasses for request bodies, query params (via `Query`), path params (via `Path`), and headers. The same model can be reused across multiple endpoints.

- **Async-first Architecture**: FastAPI supports both `def` (sync, run in threadpool) and `async def` (native async). Use `async def` when doing I/O-bound work (DB queries, HTTP calls). Use `def` for CPU-bound or synchronous libraries. Mixing is safe — FastAPI handles the bridge.

- **Starlette Foundation**: FastAPI extends Starlette for HTTP handling, routing, middleware, and ASGI. Understanding Starlette's request/response cycle helps with custom middleware, exception handlers, and raw Request access.

- **OpenAPI Auto-Generation**: Every path operation, parameter, and response model automatically generates OpenAPI schema. Use `response_model` to control output schema, `responses` for additional status codes, and `tags` for documentation grouping.

- **Security as Dependencies**: Security is implemented as dependency injection chains. `OAuth2PasswordBearer` → `get_current_user` → `require_scopes`. JWT tokens, password hashing (bcrypt/passlib), and OAuth2 flows are all DI-based.

- **Response Control Hierarchy**: `response_model` filters output → `status_code` sets HTTP status → `Response` class overrides everything. Use `response_model_exclude_unset=True` to omit default values.

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch01](chapters/ch01-getting-started.md) | Getting Started | FastAPI app, path operations, async/await |
| [ch02](chapters/ch02-path-params.md) | Path Parameters | Type conversion, Enum validation, path converters |
| [ch03](chapters/ch03-query-params.md) | Query Parameters | Defaults, validation, bool conversion |
| [ch04](chapters/ch04-request-body.md) | Request Body | Pydantic models, nested models, body updates |
| [ch05](chapters/ch05-dependencies.md) | Dependencies | DI system, yield deps, sub-dependencies, classes |
| [ch06](chapters/ch06-security.md) | Security | OAuth2, JWT, password hashing, scopes |
| [ch07](chapters/ch07-responses.md) | Responses | Response models, status codes, headers, cookies |
| [ch08](chapters/ch08-advanced.md) | Advanced | Middleware, events, WebSockets, testing, templates |
| [ch09](chapters/ch09-deployment.md) | Deployment | Docker, server workers, HTTPS, reverse proxy |
| [ch10](chapters/ch10-openapi.md) | OpenAPI & Clients | Schema customization, docs UI, client generation |

## Topic Index

- **async/await** → ch01, ch08
- **Background Tasks** → ch08
- **CORS** → ch08
- **Custom Responses** → ch07
- **Dependency Injection** → ch05
- **Docker** → ch09
- **Error Handling** → ch08
- **File Upload** → ch04
- **Form Data** → ch04
- **HTTPS** → ch09
- **Middleware** → ch08
- **OAuth2** → ch06
- **OpenAPI** → ch10
- **Path Parameters** → ch02
- **Pydantic** → ch04
- **Query Parameters** → ch03
- **Request Body** → ch04
- **Response Model** → ch07
- **Security** → ch06
- **Server-Sent Events** → ch08
- **Status Codes** → ch07
- **Static Files** → ch08
- **Sub-applications** → ch09
- **Testing** → ch08
- **Templates** → ch08
- **WebSocket** → ch08

## Supporting Files

- [glossary.md](glossary.md) — all key terms with definitions
- [patterns.md](patterns.md) — all techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — quick reference tables and decision guides

---

## Scope & Limits

This skill covers the official FastAPI documentation only. For hands-on implementation in your codebase,
combine with project-specific tools. For topics beyond FastAPI (SQLModel, Typer, Starlette internals),
check related skills or ask the agent directly.
