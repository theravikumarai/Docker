# Docker

## Overview

Docker helps package an application with its dependencies into a portable and reproducible environment.

```text
Application + Dependencies
            ↓
        Dockerfile
            ↓
       Docker Image
            ↓
    Docker Container
            ↓
 Development / CI-CD / Cloud
```

## Why Docker for AI/ML?

Machine learning applications often depend on specific:

* Python versions
* ML libraries
* System dependencies
* Model files
* Runtime environments

Docker helps ensure that the same application runs consistently across different environments.

```text
Developer Machine
        ↓
Testing Environment
        ↓
CI/CD Pipeline
        ↓
Cloud / Production
```

## Core Concepts

### Dockerfile

A file containing instructions to build a Docker image.

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Dockerfile → Image → Container

```text
Dockerfile
    ↓ docker build
Docker Image
    ↓ docker run
Docker Container
```

| Component  | Description                  |
| ---------- | ---------------------------- |
| Dockerfile | Instructions / blueprint     |
| Image      | Packaged application         |
| Container  | Running instance of an image |

## Docker Architecture

```text
Developer
    │
    ▼
Docker CLI
    │
    ▼
Docker Daemon
    │
    ├── Images
    ├── Containers
    ├── Networks
    └── Volumes
```

## Important Dockerfile Instructions

| Instruction | Purpose                                   |
| ----------- | ----------------------------------------- |
| `FROM`      | Defines the base image                    |
| `WORKDIR`   | Sets the working directory                |
| `COPY`      | Copies files into the image               |
| `RUN`       | Executes commands during image build      |
| `CMD`       | Default command when the container starts |
| `EXPOSE`    | Documents the application port            |

### `RUN` vs `CMD`

```text
RUN → Build Time
CMD → Container Runtime
```

Example:

```dockerfile
RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

## Common Docker Commands

```bash
# Build an image
docker build -t image-name .

# List images
docker images

# Run a container
docker run -p 8000:8000 image-name

# Run in detached mode
docker run -d -p 8000:8000 image-name

# Running containers
docker ps

# All containers
docker ps -a

# Stop a container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# Remove an image
docker rmi <image_id>

# View logs
docker logs <container_id>

# Execute a command inside a container
docker exec -it <container_id> /bin/sh
```

## Port Mapping

```bash
docker run -p 8000:8000 image-name
```

Format:

```text
HOST_PORT:CONTAINER_PORT
```

Example:

```text
localhost:8000
      ↓
Container:8000
```

## Why `0.0.0.0`?

Inside a container, binding an application to `0.0.0.0` allows it to listen on all network interfaces.

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

## Docker Layers and Caching

Docker builds images in layers.

Recommended:

```dockerfile
COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```

This allows Docker to reuse the dependency layer when application code changes.

```text
requirements.txt unchanged
        ↓
Use cached layer
        ↓
Faster build
```

## `.dockerignore`

Exclude unnecessary files from the Docker build context.

```text
__pycache__/
*.pyc
.venv/
venv/
.git/
.env
.pytest_cache/
```

Benefits:

* Smaller images
* Faster builds
* Better security

## Docker Compose

Docker Compose is used to define and run multi-container applications.

```text
Frontend
    ↓
Backend API
    ↓
ML Model
```

Example:

```yaml
services:

  backend:
    build: .
    ports:
      - "8000:8000"

  frontend:
    build:
      context: .
      dockerfile: frontend/Dockerfile
    ports:
      - "8501:8501"
    depends_on:
      - backend
```

Run:

```bash
docker compose up --build
```

Stop:

```bash
docker compose down
```

## Docker Networking

Docker Compose creates a network that allows services to communicate.

```text
Streamlit Container
        │
        │ http://backend:8000
        ▼
FastAPI Container
```

The Compose service name acts as the hostname.

### `localhost` vs Service Name

Local development:

```python
API_URL = "http://127.0.0.1:8000/predict"
```

Inside Docker Compose:

```python
API_URL = "http://backend:8000/predict"
```

Inside a container, `localhost` refers to the container itself.

## Docker Volumes

Containers are ephemeral. Data can be lost when a container is removed.

Volumes provide persistent storage.

```text
Container
    ↓
Docker Volume
    ↓
Persistent Storage
```

Common AI/ML use cases:

* MLflow artifacts
* Model files
* Databases
* Logs
* Training data

## Docker Registry

A Docker registry stores and distributes images.

```text
Dockerfile
    ↓
docker build
    ↓
Local Image
    ↓
docker push
    ↓
Docker Registry
    ↓
Deployment
```

Examples:

* Docker Hub
* AWS ECR
* Google Artifact Registry
* Azure Container Registry

## Docker in an AI/ML Pipeline

```text
Training
    ↓
Trained Model
    ↓
Model Storage / Registry
    ↓
FastAPI / ML Service
    ↓
Docker Image
    ↓
Docker Registry
    ↓
CI/CD Pipeline
    ↓
Cloud / Kubernetes
```

## AI/ML Docker Challenges

### Large Models

Model files can significantly increase image size.

Possible solutions:

* Store models in object storage
* Download models at startup
* Use model registries
* Mount models using volumes

### Dependency Compatibility

The training and serving environments should use compatible library versions.

Example:

```text
Training Environment
scikit-learn == X.X.X

        ↓

Serving Environment
scikit-learn == X.X.X
```

Use pinned dependency versions for reproducibility.

### Image Size

Use smaller base images when possible:

```dockerfile
FROM python:3.12-slim
```

## Production Best Practices

* Use specific dependency versions
* Use `.dockerignore`
* Do not store secrets inside images
* Use environment variables or secret managers
* Use non-root users
* Minimize image size
* Use Docker layer caching
* Use multi-stage builds when appropriate
* Scan images for vulnerabilities

## Multi-Stage Builds

Multi-stage builds separate build dependencies from the production image.

```text
Build Stage
    ↓
Production Stage
```

Example:

```dockerfile
FROM python:3.12 AS builder

WORKDIR /app

COPY requirements.txt .

RUN pip install --user -r requirements.txt


FROM python:3.12-slim

WORKDIR /app

COPY --from=builder /root/.local /root/.local

COPY . .

CMD ["python", "app.py"]
```

## Docker Interview Questions

### What is Docker?

Docker is a containerization platform that packages applications and dependencies into portable containers.

### Image vs Container?

An image is a packaged template, while a container is a running instance of that image.

### Dockerfile vs Image?

A Dockerfile contains instructions for building an image.

### `RUN` vs `CMD`?

`RUN` executes during image build time, while `CMD` executes when the container starts.

### Why use Docker for ML?

Docker provides reproducibility, dependency consistency, portability, and easier deployment.

### Why Docker Compose?

Docker Compose simplifies running and managing multi-container applications.

### Why does `localhost` not work between containers?

Inside a container, `localhost` refers only to that same container. Containers communicate using service names through Docker networking.

### What is a Docker Registry?

A service that stores and distributes Docker images.

### What are Docker Volumes?

Persistent storage that exists independently of a container's lifecycle.

## Quick Cheat Sheet

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker run
    ↓
Container
```

```bash
# Build
docker build -t image-name .

# Run
docker run -p 8000:8000 image-name

# List containers
docker ps

# Logs
docker logs <container_id>

# Enter container
docker exec -it <container_id> /bin/sh

# Docker Compose
docker compose up --build

# Run in background
docker compose up -d

# Stop services
docker compose down
```

## Key Takeaways

Focus on these concepts for AI/ML interviews:

```text
1. Why Docker is needed for ML
2. Dockerfile → Image → Container
3. RUN vs CMD
4. Docker layers and caching
5. Port mapping
6. .dockerignore
7. Docker Compose
8. Container networking
9. localhost vs service name
10. Docker volumes
11. Docker registries
12. Docker in CI/CD
13. Docker → Registry → Kubernetes
```

## License

This repository is intended for learning, revision, and interview preparation.
