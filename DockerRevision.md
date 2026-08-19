Absolutely. Here is a **compact but interview-focused Docker revision sheet for AI/ML Engineers**, based around the concepts you just used in your **House Price Prediction project**.

# Docker

## 1. What is Docker?

**Docker is a containerization platform that packages an application along with its dependencies, libraries, and runtime into a portable container.**

### Why do we need it?

The classic problem:

> “It works on my machine.”

An ML application may depend on:

* Python version
* CatBoost / Scikit-learn version
* NumPy / Pandas
* FastAPI
* Model files
* OS-level dependencies

Docker packages the environment so the application behaves consistently across:

```text
Developer Machine
       ↓
Testing
       ↓
CI/CD
       ↓
Cloud / Production
```

---

# 2. Docker Architecture

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

### Important components

| Component        | Meaning                                        |
| ---------------- | ---------------------------------------------- |
| Docker Client    | Where you run Docker commands                  |
| Docker Daemon    | Background service that manages Docker objects |
| Docker Image     | Read-only application template                 |
| Docker Container | Running instance of an image                   |
| Docker Registry  | Stores and distributes images                  |
| Dockerfile       | Instructions for building an image             |

---

# 3. Dockerfile vs Image vs Container

This is one of the **most common interview questions**.

```text
Dockerfile
    ↓ docker build
Docker Image
    ↓ docker run
Docker Container
```

### Dockerfile

Blueprint or instructions.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app"]
```

### Docker Image

A packaged application created from the Dockerfile.

```bash
docker build -t house-price-api .
```

### Docker Container

A running instance of an image.

```bash
docker run -p 8000:8000 house-price-api
```

### Easy analogy

```text
Dockerfile = Recipe
Image      = Prepared food
Container  = Food being served
```

---

# 4. Important Dockerfile Instructions

## `FROM`

Defines the base image.

```dockerfile
FROM python:3.12-slim
```

For AI/ML applications, common base images:

```text
python:3.12-slim
python:3.11-slim
pytorch/pytorch
tensorflow/tensorflow
nvidia/cuda
```

---

## `WORKDIR`

Sets the working directory inside the container.

```dockerfile
WORKDIR /app
```

All subsequent commands execute relative to `/app`.

---

## `COPY`

Copies files from your local machine into the container.

```dockerfile
COPY requirements.txt .
```

Then:

```dockerfile
COPY . .
```

---

## `RUN`

Executes a command while building the image.

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

`RUN` happens during:

```text
docker build
```

---

## `CMD`

Defines the default command when the container starts.

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

`CMD` happens during:

```text
docker run
```

### Interview point

```text
RUN → Image build time
CMD → Container runtime
```

---

# 5. Common Docker Commands

## Build image

```bash
docker build -t house-price-api .
```

## List images

```bash
docker images
```

## Run container

```bash
docker run -p 8000:8000 house-price-api
```

## Run in detached mode

```bash
docker run -d -p 8000:8000 house-price-api
```

## List running containers

```bash
docker ps
```

## List all containers

```bash
docker ps -a
```

## Stop container

```bash
docker stop <container_id>
```

## Remove container

```bash
docker rm <container_id>
```

## Remove image

```bash
docker rmi <image_id>
```

## View logs

```bash
docker logs <container_id>
```

## Execute command inside container

```bash
docker exec -it <container_id> /bin/sh
```

---

# 6. Port Mapping

You used:

```bash
docker run -p 8000:8000 house-price-api
```

Format:

```text
HOST_PORT : CONTAINER_PORT
```

Example:

```text
localhost:8000
      │
      ▼
Container:8000
```

Another example:

```bash
docker run -p 8080:8000 house-price-api
```

Then:

```text
localhost:8080
      ↓
container:8000
```

---

# 7. Why `0.0.0.0` Inside Docker?

For FastAPI:

```dockerfile
CMD [
  "uvicorn",
  "app.main:app",
  "--host",
  "0.0.0.0",
  "--port",
  "8000"
]
```

Inside a container:

```text
127.0.0.1
```

means only the container itself.

```text
0.0.0.0
```

means listen on all network interfaces, allowing access through the mapped host port.

---

# 8. Docker Layers and Caching

Docker builds images in layers.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```

Why copy `requirements.txt` first?

Because Docker can cache the dependency installation layer.

```text
requirements.txt unchanged
        ↓
Use cached dependencies
        ↓
Faster build
```

If you copied everything before installing dependencies:

```dockerfile
COPY . .

RUN pip install -r requirements.txt
```

A small code change could invalidate the dependency layer and trigger installation again.

### Interview answer

> Docker layer caching improves build performance by reusing unchanged layers.

---

# 9. `.dockerignore`

Similar to `.gitignore`.

Example:

```text
__pycache__/
*.pyc
.venv/
venv/
.git/
.env
.pytest_cache/
```

Why?

Without `.dockerignore`, Docker may copy unnecessary files into the image.

Benefits:

* Smaller image
* Faster build
* Better security
* Cleaner production images

---

# 10. Docker Compose

You used Docker Compose because your application has multiple services:

```text
Streamlit
    │
    ▼
FastAPI
    │
    ▼
ML Model
```

Instead of manually running multiple containers:

```bash
docker run ...
docker run ...
```

you define them in:

```text
docker-compose.yml
```

Then run:

```bash
docker compose up --build
```

---

# 11. Docker Compose Architecture

Your project:

```text
Docker Compose
│
├── frontend
│     └── Streamlit
│           Port 8501
│
└── backend
      └── FastAPI
            Port 8000
```

Docker Compose automatically creates a network.

Therefore:

```text
frontend
    │
    │ http://backend:8000
    ▼
backend
```

The service name becomes the hostname.

This is why you used:

```python
API_URL = "http://backend:8000/predict"
```

inside Docker.

---

# 12. `localhost` vs Docker Service Name

This is an important interview concept.

### Local development

```python
API_URL = "http://127.0.0.1:8000/predict"
```

### Inside Docker Compose

```python
API_URL = "http://backend:8000/predict"
```

Why?

Inside a container:

```text
localhost
```

means:

> This same container.

It does **not** mean your host machine or another container.

For communication between containers:

```text
Use Docker service name
```

Example:

```text
frontend → backend
api → database
ml-service → redis
```

---

# 13. `docker compose up` vs `docker compose up --build`

### Start existing images

```bash
docker compose up
```

### Rebuild images before starting

```bash
docker compose up --build
```

Use `--build` when:

* Dockerfile changed
* Dependencies changed
* Application files need rebuilding

---

# 14. Docker Compose Commands

## Start services

```bash
docker compose up
```

## Build and start

```bash
docker compose up --build
```

## Detached mode

```bash
docker compose up -d
```

## Stop and remove services

```bash
docker compose down
```

## View services

```bash
docker compose ps
```

## View logs

```bash
docker compose logs
```

---

# 15. Docker Volumes

Containers are temporary.

If a container is deleted:

```text
Container Data
     ❌ Lost
```

Volumes provide persistent storage.

Example:

```yaml
volumes:
  model_data:
```

Use cases in ML:

* MLflow artifacts
* Model files
* Databases
* Training data
* Logs

Concept:

```text
Container
    │
    ▼
Docker Volume
    │
    ▼
Persistent Storage
```

---

# 16. Docker Networks

Docker networks allow containers to communicate.

Example:

```text
Frontend Container
       │
       │
Docker Network
       │
       ▼
Backend Container
```

With Docker Compose, a default network is generally created automatically.

For larger systems:

```text
Frontend
    ↓
API Gateway
    ↓
ML Services
    ↓
Databases
    ↓
Monitoring
```

Networks help isolate and connect these services.

---

# 17. Docker Registry

A registry stores Docker images.

Flow:

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
Cloud / Kubernetes
```

Examples:

* Docker Hub
* AWS ECR
* Google Artifact Registry
* Azure Container Registry

This is the next step after local Docker.

---

# 18. Docker in an AI/ML Pipeline

A typical AI/ML architecture:

```text
Training
   ↓
Trained Model
   ↓
Model Registry / Storage
   ↓
FastAPI / ML Service
   ↓
Docker Image
   ↓
Docker Registry
   ↓
CI/CD
   ↓
Kubernetes / Cloud Deployment
```

Your project currently covers:

```text
CatBoost Model
      ↓
FastAPI
      ↓
Docker Image
      ↓
Container
      ↓
Docker Compose
      ↓
Streamlit + FastAPI
```

---

# 19. Important ML-Specific Docker Challenges

## Large model files

Models can significantly increase image size.

Solutions:

* Store models in S3
* Download model at startup
* Use model registries
* Use volumes
* Optimize model format

## Dependency consistency

The model may fail if training and serving libraries are incompatible.

Example:

```text
Training:
scikit-learn 1.x

Production:
scikit-learn incompatible version
```

Solution:

```text
Pin dependency versions
```

Example:

```text
fastapi==...
catboost==...
pandas==...
```

## Image size

Avoid unnecessary packages.

Prefer:

```dockerfile
FROM python:3.12-slim
```

when suitable.

---

# 20. Production Best Practices

### Use specific versions

Instead of:

```text
fastapi
catboost
```

prefer pinned versions after validating compatibility:

```text
fastapi==x.y.z
catboost==x.y.z
```

### Use `.dockerignore`

Avoid copying:

```text
venv/
.git/
.env
__pycache__/
```

### Don't store secrets inside images

Use:

* Environment variables
* GitHub Secrets
* AWS Secrets Manager
* Kubernetes Secrets

### Use smaller base images

When compatible:

```text
python:3.12-slim
```

### Use non-root users

For production security.

### Separate build and runtime stages

Use multi-stage builds when needed.

---

# 21. Multi-Stage Builds

A multi-stage build separates:

```text
Build Environment
       ↓
Production Environment
```

Conceptually:

```dockerfile
FROM python:3.12 AS builder

# Install/build dependencies


FROM python:3.12-slim

# Copy only required artifacts
```

Benefits:

* Smaller image
* Fewer unnecessary dependencies
* Improved security

For your current small project, this is an optimization rather than a requirement.

---

# 22. Docker Interview Questions and Short Answers

### What is Docker?

> Docker is a containerization platform that packages an application and its dependencies into portable containers, ensuring consistent execution across environments.

### Docker image vs container?

> An image is an immutable template, while a container is a running instance of that image.

### Dockerfile vs Docker image?

> A Dockerfile contains instructions for building an image. The image is the packaged result.

### `RUN` vs `CMD`?

> `RUN` executes commands during image build time, while `CMD` defines the default command executed when the container starts.

### Why use Docker in ML?

> Docker ensures reproducibility, dependency consistency, portability, easier deployment, and integration with CI/CD and orchestration platforms.

### Why use Docker Compose?

> Docker Compose defines and runs multi-container applications using a single configuration file.

### Why doesn't `localhost` work between containers?

> Inside a container, localhost refers to that same container. Containers communicate through Docker networking, typically using service names.

### What does `-p 8000:8000` mean?

> It maps port 8000 on the host machine to port 8000 inside the container.

### What is a Docker registry?

> A Docker registry is a repository for storing and distributing Docker images, such as Docker Hub or AWS ECR.

### What are Docker volumes?

> Volumes provide persistent storage independent of the container lifecycle.

---

# 23. Your 30-Second Interview Explanation

You can explain your project like this:

> “I built a house price prediction application using a CatBoost model. The backend was implemented using FastAPI, which exposes prediction and health-check endpoints. I created a Streamlit frontend that communicates with the FastAPI service through REST APIs. I containerized both services using separate Dockerfiles and used Docker Compose to orchestrate them. Docker Compose provides service discovery and networking, allowing the Streamlit container to communicate with the backend using the service name instead of localhost.”

---

# Final Docker Cheat Sheet

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

# Images
docker images

# Run
docker run -p 8000:8000 image-name

# Running containers
docker ps

# All containers
docker ps -a

# Logs
docker logs <container_id>

# Enter container
docker exec -it <container_id> /bin/sh

# Stop
docker stop <container_id>

# Docker Compose
docker compose up --build

# Background mode
docker compose up -d

# Stop Compose
docker compose down
```
---
