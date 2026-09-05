# Chapter 9: Deployment

## Core Idea
Deploying FastAPI applications involves running ASGI servers like Uvicorn, configuring workers for production, and implementing security measures like HTTPS.

## Frameworks Introduced
- **Uvicorn**: High-performance ASGI server. Use for running FastAPI in production.
- **Gunicorn**: Process manager. Use with Uvicorn workers for multi-process deployment.
- **Docker**: Containerization platform. Use for consistent deployment across environments.
- **Traefik**: Reverse proxy. Use for load balancing and HTTPS termination.

## Key Concepts
- **ASGI Server**: Server that handles asynchronous Python web applications (Uvicorn, Hypercorn).
- **Workers**: Multiple processes handling requests concurrently for better performance.
- **HTTPS**: Encrypted communication protocol required for secure APIs.
- **Process Manager**: Tool that manages multiple worker processes (Gunicorn, Supervisor).

## Mental Models
- **Use `fastapi run` for development**: It provides auto-reload and debugging features.
- **Use Uvicorn with workers for production**: Multiple processes handle concurrent requests.
- **Use Docker for consistent environments**: Same container runs in development and production.

## Anti-patterns
- **Don't use `--reload` in production**: It consumes too many resources and is unstable.
- **Don't run as root**: Use a non-root user for security.
- **Don't skip HTTPS**: Always use HTTPS in production to protect sensitive data.

## Code Examples
```bash
# Development
uv run fastapi dev main.py

# Production with single worker
uv run fastapi run main.py --host 0.0.0.0 --port 80

# Production with Uvicorn directly
uv run uvicorn main:app --host 0.0.0.0 --port 80 --workers 4

# Docker
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

## Reference Tables

| Deployment Method | Use Case | Pros | Cons |
|------------------|----------|------|------|
| `fastapi dev` | Development | Auto-reload, debugging | Not for production |
| `fastapi run` | Simple production | Easy setup | Single process |
| Uvicorn + workers | Production | Multi-process, performance | More configuration |
| Docker | Containerized | Consistent environments | Learning curve |
| Kubernetes | Orchestration | Scalable, managed | Complex setup |

## Worked Example
Deploy with Docker:
1. Create Dockerfile:
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```
2. Build and run:
```bash
docker build -t myapi .
docker run -p 80:80 myapi
```

## Key Takeaways
1. Use `fastapi run` for production deployments (not `fastapi dev`).
2. Uvicorn is the default ASGI server that comes with FastAPI.
3. Multiple workers handle concurrent requests for better performance.
4. Always use HTTPS in production to protect sensitive data.
5. Docker provides consistent deployment environments across development and production.

## Connects To
- **Ch 1**: Development server is different from production deployment.
- **Ch 6**: HTTPS is required for secure token transmission.
- **Ch 8**: Middleware and background tasks may behave differently in production.
