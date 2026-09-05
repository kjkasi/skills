# FastAPI Cheatsheet

## Async vs Def Decision
| Scenario | Use | Reason |
|----------|-----|--------|
| Calls `await` (async library) | `async def` | Required for await syntax |
| Blocking I/O (DB, file, API) | `def` | Runs in threadpool, avoids blocking |
| No I/O, pure computation | `async def` | Better performance, no overhead |
| Uncertain | `def` | Safe default, FastAPI handles correctly |

## Response Model vs Response Class
| Need | Use | Notes |
|------|-----|-------|
| Filter/hide fields | `response_model=Model` | Auto-filters output |
| Custom status code | `status_code=201` in decorator | Simpler than Response |
| Headers/cookies | `response_class=Response` | Manual control needed |
| Streaming | `StreamingResponse` | For large data/files |
| JSON direct | Return `dict` | FastAPI wraps in JSONResponse |

## Depends vs Global Dependencies
| Scope | Use | Example |
|-------|-----|---------|
| Single endpoint | `Depends()` in parameter | `user = Depends(get_user)` |
| All router paths | `router = APIRouter(dependencies=[...])` | Auth for entire module |
| All app paths | `app = FastAPI(dependencies=[...])` | Global auth/logging |
| Multiple levels | Both | Dependencies compose hierarchically |

## Status Code Selection
| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid client input |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Valid auth, insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 422 | Validation Error | Invalid request data (auto) |
| 500 | Server Error | Unhandled exceptions |

## Error Handling Patterns
| Pattern | Use | Implementation |
|---------|-----|----------------|
| HTTPException | Standard API errors | `raise HTTPException(404, "Not found")` |
| Custom handler | App-specific errors | `@app.exception_handler(CustomError)` |
| Validation errors | Request validation | Override `RequestValidationError` handler |
| WebSocket errors | WS-specific | `raise WebSocketException(code)` |

## Deployment Choices
| Scenario | Server | Command |
|----------|--------|---------|
| Development | Uvicorn | `fastapi dev` or `uvicorn app:app` |
| Single process prod | Uvicorn | `fastapi run --host 0.0.0.0 --port 80` |
| Multi-core CPU | Uvicorn workers | `uvicorn app:app --workers 4` |
| UNIX production | Gunicorn + Uvicorn | `gunicorn app:app -k uvicorn.workers.UvicornWorker` |
| Containerized | Docker + Uvicorn | Multi-stage Dockerfile |

## WebSocket vs SSE
| Feature | WebSocket | SSE |
|---------|-----------|-----|
| Direction | Bidirectional | Server → Client only |
| Protocol | ws:// / wss:// | HTTP/HTTPS |
| Connection | Persistent | Long-lived HTTP |
| Use case | Chat, gaming, collab | Notifications, streaming |
| Complexity | Higher | Lower |
| Browser support | Excellent | Excellent (IE no) |
| Auto-reconnect | Manual | Built-in |
| FastAPI support | `@app.websocket()` | `EventSourceResponse` |

## File Upload Quick Reference
| Type | Declaration | Use Case |
|------|-------------|----------|
| Single file | `file: UploadFile = File()` | Image, document |
| Multiple files | `files: list[UploadFile] = File()` | Batch upload |
| Raw bytes | `data: bytes = File()` | Small binary data |
| Mixed form | `file: UploadFile, name: str = Form()` | File + metadata |

## Testing Patterns
| Tool | Use | Import |
|------|-----|--------|
| TestClient | Sync tests | `from fastapi.testclient import TestClient` |
| AsyncClient | Async tests | `from httpx import AsyncClient` |
| pytest fixtures | Setup/teardown | Standard pytest |
| Dependency override | Mock dependencies | `app.dependency_overrides[dep] = mock` |
