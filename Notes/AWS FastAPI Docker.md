Below are your **fully revised, industry-standard deployment notes**, rewritten for **clarity, correctness, production relevance, and interview readiness**.
I have **not skipped any topic**. Every section is **expanded, structured, and explained the way real teams document ML API deployments**.

---

# 📘 Deploying a Dockerized FastAPI ML API on AWS EC2

**(Industry-Standard Deployment Notes)**

---

## 1. Video Overview & Deployment Objective

This video represents the **final stage of an end-to-end ML API development pipeline**, focusing on **cloud deployment**.

### What This Video Achieves

* Deploys a **Dockerized FastAPI ML inference API** to the cloud
* Uses **AWS EC2** as the compute layer
* Pulls the API image from **Docker Hub**
* Configures **network security** for public access
* Demonstrates **frontend (Streamlit) → backend (FastAPI) integration**

This workflow reflects a **realistic beginner-to-intermediate production deployment**, commonly used before moving to managed services like ECS or EKS.

---

## 2. Project Recap & Context

### ML Use Case

* **Problem**: Insurance premium prediction
* **Model Output**:

  * `High`, `Medium`, `Low`
* **Input Features**:

  * Age
  * Height / Weight
  * Income
  * Smoking status
  * City
  * Occupation

### API Layer

* Built using **FastAPI**
* Exposes:

  * `/` → Home
  * `/health` → Health check
  * `/predict` → ML inference
  * `/docs` → Swagger UI
* Previously:

  * Refactored to follow industry standards
  * Dockerized and pushed to Docker Hub

### Current Goal

👉 **Run the same Docker image on a cloud server (AWS EC2) and make it publicly accessible**

---

## 3. Why AWS EC2 for This Deployment?

Amazon Web Services provides multiple deployment options.

### Why EC2?

* Full control over the server
* Simple mental model
* Ideal for learning fundamentals
* Eligible for **AWS Free Tier**

⚠️ Note:
This is **not a highly scalable production setup**, but a **foundational deployment pattern**.

---

## 4. Prerequisites

Before deployment, ensure:

### Technical Knowledge

* Basic cloud concepts
* Docker fundamentals
* Linux command-line basics

### Accounts & Access

* Active AWS account (credit/debit card required)
* Docker Hub account
* Docker image already pushed

---

## 5. Launching an EC2 Instance

### Step 1: Access EC2 Service

* Login to AWS Console
* Search for **EC2**
* Open EC2 Dashboard

---

### Step 2: Launch Instance

#### Instance Configuration

| Setting       | Value                   | Reason                   |
| ------------- | ----------------------- | ------------------------ |
| Name          | FastAPI Server          | Identification           |
| AMI           | Ubuntu Server 22.04 LTS | Stable, widely supported |
| Instance Type | `t2.micro`              | Free tier eligible       |
| Storage       | 8 GB                    | Sufficient for API       |
| Key Pair      | Required (.pem)         | Secure access            |

---

### Key Concepts Explained

#### Amazon Machine Image (AMI)

* Template containing OS + base config
* Used to launch instances

#### Instance Type (`t2.micro`)

* 1 vCPU, 1 GB RAM
* Suitable for small APIs
* Larger types incur cost

#### Key Pair

* SSH authentication mechanism
* Required even if web-based SSH is used

---

### Network Settings (Initial)

* Allow SSH (port `22`) from anywhere
* API port (`8000`) will be opened later

Launch instance and wait until **Status = Running**.

---

## 6. Connecting to the EC2 Instance

### Method Used

* **EC2 Instance Connect** (browser-based SSH)

This avoids local SSH configuration for beginners.

---

## 7. Installing Docker on EC2

Once connected, prepare the server.

### Step 1: Update System Packages

```bash
sudo apt update
```

Ensures latest package metadata.

---

### Step 2: Install Docker

```bash
sudo apt install docker.io -y
```

Installs Docker Engine.

---

### Step 3: Start & Enable Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

* `start` → runs Docker now
* `enable` → auto-start after reboot

---

### Step 4: Allow Non-Root Docker Access

```bash
sudo usermod -aG docker $USER
exit
```

Reconnect to apply permissions.

✅ This avoids running Docker commands with `sudo`.

---

### Verify Installation

```bash
docker ps
```

Confirms Docker daemon is running.

---

## 8. Pulling the FastAPI Docker Image

### Pull Image from Docker Hub

```bash
docker pull niteshkg/fastapi_ml_api:latest
```

This downloads:

* FastAPI app
* ML model
* Dependencies
* Uvicorn server

---

## 9. Running the API Container

```bash
docker run -d -p 8000:8000 niteshkg/fastapi_ml_api:latest
```

### Command Explanation

* `-d` → Detached mode (background)
* `-p 8000:8000` → Port mapping
* Container runs Uvicorn internally

At this point:

* API is running
* BUT not accessible externally yet

---

## 10. Security Group Configuration (Critical Step)

Amazon EC2 instances are protected by **Security Groups**.

### Why API Is Not Accessible Yet

* EC2 firewall blocks inbound traffic
* Port `8000` is closed by default

---

### Step 1: Identify Failure

* Visit:

  ```
  http://<PUBLIC_IP>:8000
  ```
* Result: Timeout / failure

---

### Step 2: Open Inbound Port

#### Security Group Rules

* Type: `Custom TCP`
* Port: `8000`
* Source: `0.0.0.0/0` (Anywhere)

⚠️ For learning only — production should restrict IPs.

Save rules.

---

## 11. Verifying Deployment

### Public API Endpoints

| Endpoint   | Purpose      |
| ---------- | ------------ |
| `/`        | Home         |
| `/health`  | Health check |
| `/docs`    | Swagger UI   |
| `/predict` | ML inference |

Example:

```
http://<PUBLIC_IP>:8000/docs
```

---

### Testing Prediction Endpoint

* Open Swagger UI
* Click `/predict`
* Enter sample inputs
* Execute request
* Receive ML prediction

This confirms:
✔ Docker image works
✔ EC2 deployment successful
✔ Public access enabled

---

## 12. Streamlit Frontend Integration

### Problem

Streamlit frontend was calling:

```python
http://localhost:8000/predict
```

This works only locally.

---

### Solution: Update API URL

```python
EC2_PUBLIC_IP = "<YOUR_EC2_PUBLIC_IPV4_ADDRESS>"
API_URL = f"http://{EC2_PUBLIC_IP}:8000/predict"
```

No other frontend changes required.

---

### Result

* Streamlit UI remains local
* Backend API runs on AWS
* End-to-end ML inference works

This separation mirrors **real production architectures**.

---

## 13. Why This Deployment Matters (Industry View)

![Image](https://miro.medium.com/0%2AJy60MddjBbL4Qrct)

![Image](https://miro.medium.com/0%2AJ8gUqACVn8wmPjGd.png)

![Image](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2021/04/27/tholane_arch_1000.png)

### Key Industry Outcomes

* Cloud-hosted ML inference
* Reusable Docker image
* Stateless API service
* Frontend-backend decoupling
* Foundation for CI/CD & Kubernetes

---

## 14. Limitations of This Approach (Honest Assessment)

This setup:

* ❌ Single instance (no scaling)
* ❌ Manual deployment
* ❌ No load balancing
* ❌ No auto-healing

But it is:
✅ Perfect for learning
✅ Interview-ready foundation
✅ Stepping stone to ECS / EKS

---

## 15. Conclusion

This video completes a **full ML API lifecycle**:

1. Build ML model
2. Create FastAPI inference API
3. Dockerize application
4. Push image to Docker Hub
5. Deploy to AWS EC2
6. Configure security
7. Integrate frontend

---

### 🧠 Industry Takeaway

> **Docker ensures reproducibility.
> EC2 provides compute.
> Security groups control access.
> APIs enable ML at scale.**

---

If you want next, I can:

* Upgrade this to **ECS or EKS**
* Add **Nginx + Load Balancer**
* Introduce **CI/CD (GitHub Actions)**
* Convert this into **FAANG interview answers**
* Provide **production hardening checklist**

Just tell me what’s next 🚀
