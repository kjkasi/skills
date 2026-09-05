# Chapter 10: Containers & DevOps

## Core Idea
VS Code provides container-based development through Dev Containers, Docker integration, and Kubernetes support, enabling consistent development environments.

## Frameworks Introduced
- **Dev Containers**: Develop inside Docker containers with consistent environment and tools
- **Docker Integration**: Container management, image building, and docker-compose support
- **Kubernetes**: Cluster management and deployment from VS Code
- **Bridge to Kubernetes**: Local development connected to Kubernetes clusters

## Key Concepts
- **Dev Container**: Docker-based development environment defined in `devcontainer.json`
- **Container Features**: Reusable components that add tools to dev containers
- **Docker Compose**: Multi-container development environments
- **Container Registry**: Push and pull container images from registries
- **Kubernetes**: Container orchestration with VS Code integration

## Mental Models
- Use **Dev Containers** for consistent team environments and reproducible builds
- Use **Docker Compose** for multi-service development (e.g., app + database + cache)
- Use **Bridge to Kubernetes** to test against real cluster without deploying

## Anti-patterns
- **Running containers without resource limits**: Set memory/CPU limits to prevent host slowdown
- **Hardcoding secrets in Dockerfiles**: Use multi-stage builds and secret management
- **Skipping .dockerignore**: Always include to avoid copying unnecessary files

## Code Examples
```json
// .devcontainer/devcontainer.json
{
  "name": "Python Dev",
  "image": "mcr.microsoft.com/devcontainers/python:3.11",
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
  "postCreateCommand": "pip install -r requirements.txt",
  "forwardPorts": [8000],
  "customizations": {
    "vscode": {
      "extensions": ["ms-python.python"]
    }
  }
}
```

## Key Takeaways
1. Use `.devcontainer/devcontainer.json` to define your development environment
2. Container features add tools without creating custom Docker images
3. Docker Compose supports multi-container development workflows
4. Bridge to Kubernetes enables local development against cluster services
5. Always use `.dockerignore` to keep images small and builds fast

## Connects To
- **Ch 11**: Remote development extends container workflows
- **Ch 12**: Debugging inside containers
- **Ch 13**: Azure container deployment
