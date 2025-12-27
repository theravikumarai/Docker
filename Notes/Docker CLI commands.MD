Below is an **industry-standard, interview-ready list of the MOST USED Docker CLI commands**, grouped **exactly the way professionals use Docker in real projects**.

I’ve kept explanations **clear, practical, and concise**, and I’ll highlight **which commands are critical for ML / API / production workflows**.

---

# 🐳 Docker CLI – Most Used Commands (Industry Standard)

Docker CLI is your **primary interface** for building, running, debugging, and shipping containers.

---

## 1️⃣ Docker Installation & System Check (Very Basic but Often Asked)

### Check Docker installation

```bash
docker --version
docker info
```

### Verify Docker is running

```bash
docker ps
```

👉 If this works, Docker daemon is active.

---

## 2️⃣ Image Management Commands (VERY IMPORTANT)

### List local images

```bash
docker images
```

### Pull image from Docker Hub

```bash
docker pull <image_name>
# Example:
docker pull python:3.9-slim
```

### Build image from Dockerfile

```bash
docker build -t <username>/<image_name>:<tag> .
# Example:
docker build -t tws24/fastapi-ml-app:v1 .
```

### Remove image

```bash
docker rmi <image_id>
```

### Remove unused images (cleanup)

```bash
docker image prune
```

📌 **Interview Tip**

> Image management is about **versioning and reproducibility**.

---

## 3️⃣ Container Lifecycle Commands (MOST USED DAILY)

### Run a container

```bash
docker run <image_name>
```

### Run container in detached mode

```bash
docker run -d <image_name>
```

### Run container with port mapping (VERY COMMON)

```bash
docker run -p <host_port>:<container_port> <image_name>
# Example:
docker run -p 8000:8000 fastapi-ml-app
```

### Run container with name

```bash
docker run --name my_container <image_name>
```

---

### List running containers

```bash
docker ps
```

### List all containers (running + stopped)

```bash
docker ps -a
```

---

### Stop a running container

```bash
docker stop <container_id>
```

### Start a stopped container

```bash
docker start <container_id>
```

### Remove container

```bash
docker rm <container_id>
```

📌 **Production Insight**

> Containers are **ephemeral** — delete and recreate, don’t fix manually.

---

## 4️⃣ Debugging & Logs (CRITICAL FOR INTERVIEWS)

### View container logs

```bash
docker logs <container_id>
```

### Follow logs (real-time)

```bash
docker logs -f <container_id>
```

### Execute command inside running container

```bash
docker exec -it <container_id> /bin/bash
```

👉 Used to:

* Inspect files
* Debug runtime issues
* Verify environment

---

## 5️⃣ Docker Hub & Registry Commands

### Login to Docker Hub

```bash
docker login
```

### Push image to Docker Hub

```bash
docker push <username>/<image_name>:<tag>
```

### Logout

```bash
docker logout
```

📌 **Interview Red Flag**
❌ Using `latest` in production
✅ Use versioned tags (`v1`, `v2`, commit hash)

---

## 6️⃣ Cleanup Commands (VERY IMPORTANT IN REAL SYSTEMS)

### Remove stopped containers

```bash
docker container prune
```

### Remove unused images

```bash
docker image prune
```

### Remove everything unused (DANGEROUS)

```bash
docker system prune
```

👉 Frees disk space on servers.

---

## 7️⃣ Networking & Ports (Often Asked)

### Inspect container details

```bash
docker inspect <container_id>
```

### Check port bindings

```bash
docker port <container_id>
```

---

## 8️⃣ Dockerfile & Build Optimization (Senior-Level)

### Build without cache (debugging)

```bash
docker build --no-cache -t myimage .
```

### View image history (layers)

```bash
docker history <image_name>
```

📌 Shows why image is large.

---

## 9️⃣ Docker Compose (Very Common)

### Start services

```bash
docker-compose up
```

### Start in background

```bash
docker-compose up -d
```

### Stop services

```bash
docker-compose down
```

### View running services

```bash
docker-compose ps
```

---

## 🔟 ML / API Specific Docker Commands (INTERVIEW GOLD)

### Run FastAPI container

```bash
docker run -p 8000:8000 fastapi-app
```

### Run with environment variables

```bash
docker run -e ENV=prod fastapi-app
```

### Limit resources (performance tuning)

```bash
docker run --memory="1g" --cpus="1.0" fastapi-app
```

---

## 🧠 MOST ASKED DOCKER INTERVIEW COMMANDS (MEMORIZE THESE)

| Purpose         | Command        |
| --------------- | -------------- |
| Build image     | `docker build` |
| Run container   | `docker run`   |
| View logs       | `docker logs`  |
| Enter container | `docker exec`  |
| List containers | `docker ps`    |
| Push image      | `docker push`  |
| Pull image      | `docker pull`  |
| Stop container  | `docker stop`  |

---

## 🎯 FINAL INTERVIEW TIP (IMPORTANT)

When asked about Docker commands, **don’t just list commands**.

Say:

> “I typically use Docker to build reproducible ML inference images, run containers with proper port mapping, debug using logs and exec, and push versioned images to a registry for deployment.”

That sentence alone signals **senior-level experience**.

---

If you want next, I can:

* Create a **1-page Docker CLI cheat sheet**
* Map **Docker commands → interview questions**
* Explain **real debugging scenarios**
* Compare **Docker CLI vs Kubernetes kubectl**

Just tell me 👍
