# Chapter 8: Advanced Features

## Core Idea
FastAPI provides advanced features like middleware, background tasks, WebSockets, and testing tools that extend the framework beyond basic request/response handling.

## Frameworks Introduced
- **Middleware**: Functions that process every request/response. Use for logging, CORS, authentication.
- **BackgroundTasks**: Run code after sending response. Use for emails, notifications, processing.
- **WebSocket**: Bidirectional communication. Use for real-time features like chat, notifications.
- **TestClient**: Testing tool based on HTTPX. Use for writing API tests with pytest.

## Key Concepts
- **Middleware**: Functions that run before and after every request/response.
- **Background Tasks**: Operations that run after returning a response to the client.
- **WebSocket**: Protocol for bidirectional communication between client and server.
- **APIRouter**: Organize routes into modular groups for larger applications.
- **TestClient**: Tool for testing FastAPI applications without running a server.

## Mental Models
- **Use middleware for cross-cutting concerns**: Logging, CORS, authentication should be middleware.
- **Think of background tasks as fire-and-forget**: They run after the response is sent.
- **Use APIRouter for modular applications**: Group related routes together in separate files.

## Anti-patterns
- **Don't use background tasks for critical operations**: They may fail silently after response.
- **Don't forget that middleware runs for every request**: Keep it lightweight to avoid performance impact.
- **Don't test with production databases**: Use test databases for testing.

## Code Examples
```python
import time
from fastapi import FastAPI, WebSocket, WebSocketDisconnect, Request
from fastapi.responses import HTMLResponse
from fastapi.middleware.cors import CORSMiddleware
from fastapi import BackgroundTasks

app = FastAPI()

# Middleware
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()
    response = await call_next(request)
    process_time = time.perf_counter() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Background Tasks
def write_notification(email: str, message=""):
    with open("log.txt", mode="a") as email_file:
        content = f"notification for {email}: {message}\n"
        email_file.write(content)

@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent"}

# WebSocket
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

## Reference Tables

| Feature | Purpose | Use Case |
|---------|---------|----------|
| Middleware | Request/response processing | Logging, CORS, authentication |
| BackgroundTasks | Post-response operations | Email notifications, data processing |
| WebSocket | Real-time communication | Chat, live updates, notifications |
| APIRouter | Route organization | Modular application structure |
| TestClient | API testing | Unit and integration tests |

## Worked Example
Create a complete testing setup:
```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}

def test_create_item():
    response = client.post(
        "/items/",
        json={"name": "Foo", "price": 50.2},
    )
    assert response.status_code == 200
    assert response.json()["name"] == "Foo"
```
Run tests with `uv run pytest`.

## Key Takeaways
1. Middleware runs for every request/response, making it ideal for cross-cutting concerns.
2. Background tasks are perfect for operations that don't need to block the response.
3. WebSockets enable real-time bidirectional communication.
4. APIRouter helps organize large applications into modular components.
5. TestClient provides a simple way to test FastAPI applications without a server.

## Connects To
- **Ch 5**: Dependencies can also provide middleware-like functionality.
- **Ch 6**: Security middleware handles authentication globally.
- **Ch 9**: Deployment considerations affect how middleware and background tasks work.
