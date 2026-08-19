# How to Write a Dockerfile for Any Application

Think in this order:

```text
1. What is my application runtime?
        ↓
2. What dependencies are needed?
        ↓
3. Which files are required?
        ↓
4. Which port does the application use?
        ↓
5. What command starts the application?
```

This gives you the basic Dockerfile structure.

---

# Universal Dockerfile Template

```dockerfile
# 1. Choose base image
FROM <base-image>

# 2. Set working directory
WORKDIR /app

# 3. Copy dependency files
COPY <dependency-file> .

# 4. Install dependencies
RUN <install-command>

# 5. Copy application code
COPY . .

# 6. Document application port
EXPOSE <port>

# 7. Start application
CMD ["<command>", "<argument>"]
```

So the main question becomes:

> **What should I put in each placeholder for my application?**

---

# Step 1: Identify the Technology

First ask:

```text
What is my application built with?
```

Examples:

| Application | Base Image               |
| ----------- | ------------------------ |
| Python      | `python:3.12-slim`       |
| Node.js     | `node:22-alpine`         |
| Java        | `eclipse-temurin:21-jdk` |
| Go          | `golang:1.24`            |
| Nginx       | `nginx:alpine`           |

For example, your FastAPI application uses Python:

```dockerfile
FROM python:3.12-slim
```

A Node.js application might use:

```dockerfile
FROM node:22-alpine
```

---

# Step 2: Set the Working Directory

Usually:

```dockerfile
WORKDIR /app
```

This means Docker will work inside:

```text
/app
```

Conceptually:

```text
Your Local Project

HousePrice-Prediction/
    │
    ▼

Docker Container

/app/
```

---

# Step 3: Find the Dependency File

Every technology has a dependency file.

### Python

```text
requirements.txt
pyproject.toml
```

### Node.js

```text
package.json
package-lock.json
```

### Java Maven

```text
pom.xml
```

### Java Gradle

```text
build.gradle
```

For Python:

```dockerfile
COPY requirements.txt .
```

For Node.js:

```dockerfile
COPY package*.json ./
```

---

# Step 4: Install Dependencies

For Python:

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

For Node.js:

```dockerfile
RUN npm install
```

For Java Maven:

```dockerfile
RUN mvn install
```

---

# Step 5: Copy Application Code

After installing dependencies:

```dockerfile
COPY . .
```

This copies your project into the container.

For example:

```text
Local Project
│
├── app/
├── models/
├── requirements.txt
└── Dockerfile

        ↓

Container

/app/
│
├── app/
├── models/
├── requirements.txt
└── Dockerfile
```

---

# Step 6: Identify the Application Port

Ask:

> On which port does my application run?

Examples:

| Application | Common Port |
| ----------- | ----------: |
| FastAPI     |        8000 |
| Streamlit   |        8501 |
| Flask       |        5000 |
| React/Vite  |        5173 |
| Node.js     |        3000 |
| Jupyter     |        8888 |

Then:

```dockerfile
EXPOSE 8000
```

Important:

`EXPOSE` does **not actually publish the port to your machine**. It documents the port the container application uses.

Actual port mapping happens with:

```bash
docker run -p 8000:8000 image-name
```

---

# Step 7: Identify the Start Command

This is the most important question:

> **How do I normally start my application locally?**

For example:

### FastAPI

Locally:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Therefore:

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Streamlit

Locally:

```bash
streamlit run frontend/streamlit_app.py
```

Therefore:

```dockerfile
CMD ["streamlit", "run", "frontend/streamlit_app.py", "--server.address=0.0.0.0", "--server.port=8501"]
```

### Node.js

Locally:

```bash
npm start
```

Therefore:

```dockerfile
CMD ["npm", "start"]
```

---

# Your Mental Formula

Whenever you see a new application, follow this:

```text
Application
    │
    ├── What language/runtime?
    │        ↓
    │     FROM
    │
    ├── Where should code live?
    │        ↓
    │     WORKDIR
    │
    ├── What dependencies?
    │        ↓
    │     COPY dependency file
    │     RUN install command
    │
    ├── What application files?
    │        ↓
    │     COPY code
    │
    ├── What port?
    │        ↓
    │     EXPOSE
    │
    └── How does it start?
             ↓
            CMD
```

---

# Example 1: FastAPI

Suppose your project is:

```text
project/
│
├── app/
│   └── main.py
│
├── requirements.txt
└── Dockerfile
```

You know:

* Runtime → Python
* Dependencies → `requirements.txt`
* Port → `8000`
* Start command → `uvicorn app.main:app`

Therefore:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

# Example 2: Streamlit

Project:

```text
project/
│
├── streamlit_app.py
├── requirements.txt
└── Dockerfile
```

Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8501

CMD ["streamlit", "run", "streamlit_app.py", "--server.address=0.0.0.0", "--server.port=8501"]
```

---

# Example 3: Flask

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# Example 4: Node.js Application

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

---

# Example 5: ML Model API

For an AI/ML project, first check:

```text
Project
│
├── app/
│   ├── main.py
│   └── model.py
│
├── models/
│   └── model.pkl
│
└── requirements.txt
```

Questions:

```text
Runtime?        → Python
Dependencies?   → requirements.txt
Model needed?   → Yes, models/model.pkl
Port?           → 8000
Start command?  → uvicorn
```

Therefore:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

The model gets copied because:

```dockerfile
COPY . .
```

includes:

```text
models/model.pkl
```

---

# The Most Important Principle: Copy Dependencies First

Prefer this:

```dockerfile
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .
```

Instead of:

```dockerfile
COPY . .

RUN pip install --no-cache-dir -r requirements.txt
```

Why?

Docker uses caching.

```text
requirements.txt unchanged
        ↓
Docker uses cached dependency layer
        ↓
No need to reinstall packages
        ↓
Faster build
```

---

# How to Debug a Dockerfile

If Docker fails, check these five things:

### 1. Is the file path correct?

For example:

```dockerfile
COPY frontend/streamlit_app.py ./streamlit_app.py
```

Does this file actually exist?

```text
frontend/
    └── streamlit_app.py
```

---

### 2. Is the dependency file correct?

```dockerfile
COPY requirements.txt .
```

Make sure `requirements.txt` is in the build context.

---

### 3. Is the start command correct?

If locally you run:

```bash
uvicorn app.main:app
```

Your Docker command should match:

```dockerfile
CMD ["uvicorn", "app.main:app"]
```

---

### 4. Is the application listening on `0.0.0.0`?

For web applications:

```text
Inside Docker → use 0.0.0.0
```

For example:

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### 5. Is the port mapped correctly?

If the application runs inside the container on:

```text
8000
```

Then:

```bash
docker run -p 8000:8000 image-name
```

Format:

```text
HOST_PORT : CONTAINER_PORT
```

---

# My Recommended Process for You

Whenever you create a new AI/ML project, do this **before writing the Dockerfile**:

```text
STEP 1
How do I run this application locally?

        ↓

STEP 2
Which language/runtime is required?

        ↓

STEP 3
Which dependency file is used?

        ↓

STEP 4
Which files must exist inside the container?
- Code
- Model
- Config
- Data

        ↓

STEP 5
Which port does it use?

        ↓

STEP 6
Convert the local run command into CMD
```

## One-line memory trick

> **FROM → WORKDIR → COPY dependencies → RUN install → COPY code → EXPOSE → CMD**

For almost every Python AI/ML application, start with:

```dockerfile
FROM python:<version>-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE <port>

CMD ["your", "startup", "command"]
```

---