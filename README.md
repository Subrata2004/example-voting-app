# Example Voting App – Kubernetes Production Improvements

## 📌 Project Overview

This project is a production-ready implementation of the **Example Voting App**, completed as part of the **SquareOps DevOps Take-Home Assignment**.

The original application is a simple distributed system consisting of multiple microservices. My goal was not only to deploy the application on Kubernetes but also to improve it by applying production-grade Kubernetes practices.

The application allows users to vote between **Cats** and **Dogs**. Votes are stored in Redis, processed by a Worker service, saved in PostgreSQL, and displayed in real time on the Result application.

---

# 🏗 Application Architecture

```
                User Browser
                     │
      ┌──────────────┴──────────────┐
      │                             │
 Vote Application             Result Application
      │                             │
      └──────────────┬──────────────┘
                     │
                 Redis Queue
                     │
                  Worker
                     │
               PostgreSQL
```

---

# 🛠 Technologies Used

- Docker
- Docker Compose
- Kubernetes
- Kind Cluster
- NGINX Ingress Controller
- Redis
- PostgreSQL
- Python
- Node.js
- .NET
- GitHub Actions

---

# 📂 Project Structure

```
example-voting-app
│
├── vote/
├── result/
├── worker/
├── k8s-specifications/
│   ├── vote-deployment.yaml
│   ├── result-deployment.yaml
│   ├── worker-deployment.yaml
│   ├── redis-deployment.yaml
│   ├── redis-service.yaml
│   ├── db-statefulset.yaml
│   ├── db-service.yaml
│   ├── db-secret.yaml
│   ├── ingress.yaml
│   └── ...
│
├── docker-compose.yml
└── README.md
```

---

# 🚀 Running with Docker Compose

Clone the repository

```bash
git clone https://github.com/Subrata2004/example-voting-app.git

cd example-voting-app
```

Start the application

```bash
docker compose up --build
```

Open

```
http://localhost:8088
```

```
http://localhost:8081
```

---

# ☸ Running on Kubernetes

Create the Kind cluster

```bash
kind create cluster --name voting-app
```

Deploy the application

```bash
kubectl apply -f k8s-specifications/
```

Verify

```bash
kubectl get pods

kubectl get svc
```

---

# 🌐 Access using Ingress

Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Start port-forward

```bash
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8082:80
```

Open

```
http://voting.local:8082
```

```
http://result.local:8082
```

---

# ✅ Production Improvements

During this assignment I implemented several production-focused improvements.

## Kubernetes Secrets

Created a Secret to securely store PostgreSQL credentials instead of hardcoding them.

```
postgres-secret
```

---

## PostgreSQL StatefulSet

Replaced the PostgreSQL Deployment with a StatefulSet to provide

- Stable Pod identity
- Persistent storage
- Better database management

---

## Persistent Volume

Configured PersistentVolumeClaim so PostgreSQL data survives Pod restarts.

---

## Resource Requests & Limits

Added CPU and Memory requests

```yaml
requests:
  cpu: 100m
  memory: 128Mi
```

Added limits

```yaml
limits:
  cpu: 250m
  memory: 256Mi
```

---

## Health Checks

Implemented

- Readiness Probe
- Liveness Probe

This allows Kubernetes to

- detect unhealthy containers
- restart failed Pods automatically
- send traffic only when Pods are ready

---

## NGINX Ingress

Configured host-based routing

```
voting.local
```

```
result.local
```

---

# ⚠ Challenges Faced

During the implementation I encountered several practical issues.

## Problem 1

Docker failed to start.

```
Port 8080 already in use
```

### Cause

Oracle Listener (TNSLSNR.EXE) was already using port 8080.

### Solution

Changed Docker Compose mapping

Before

```
8080:80
```

After

```
8088:80
```

---

## Problem 2

NodePort was inaccessible.

```
ERR_CONNECTION_REFUSED
```

### Cause

Kind NodePorts are not directly exposed to Windows.

### Solution

Used

```bash
kubectl port-forward
```

---

## Problem 3

Port-forward failed

```
Unable to listen on port 8080
```

### Cause

Port already occupied.

### Solution

Used free ports

```
9090
9091
8082
8085
```

---

## Problem 4

Worker Pod restarted.

### Cause

Redis restarted during rollout.

### Solution

Verified logs.

After Redis became available Kubernetes automatically recovered the Worker Pod.

---

## Problem 5

Browser couldn't resolve

```
voting.local
```

### Cause

Windows hosts file wasn't updated.

### Solution

Added

```
127.0.0.1 voting.local

127.0.0.1 result.local
```

Flushed DNS

```
ipconfig /flushdns
```

---

# GitHub Actions CI/CD

The repository includes a CI/CD pipeline (`.github/workflows/vote-ci.yml`) that runs automatically on every change inside `vote/**`.

## Pipeline Triggers

The workflow runs on:

- Push to `main` when files change in `vote/**`, `k8s-specifications/**`, or the workflow itself
- Pull requests targeting `main`

## Pipeline Steps

On every trigger, the pipeline performs the following steps in order:

1. **Lint the code** — runs `flake8` on the Python vote application
2. **Lint Kubernetes manifests** — runs `yamllint` on `k8s-specifications/`
3. **Build Docker image** — builds the vote service image from `vote/Dockerfile`
4. **Push to container registry** — pushes the image to GitHub Container Registry (GHCR)
5. **Create Kind cluster** — provisions a local Kubernetes cluster for testing
6. **Deploy the application** — applies all Kubernetes manifests and sets the vote deployment to use the newly built image
7. **Run smoke test** — port-forwards the vote service and verifies `http://localhost:8080` responds successfully

If any step fails, the pipeline stops immediately and the workflow is marked as failed.

## Image Registry

Built images are pushed to:

```
ghcr.io/<repository-owner>/example-voting-app-vote:<commit-sha>
```

## Why Lint First?

Linting runs before build and deploy so that style issues and manifest errors are caught early. This prevents broken or poorly formatted code from progressing through the pipeline — the same failure you would see if `flake8` reports PEP 8 violations in `vote/app.py`.

---

# Learning Outcomes

This project helped me gain practical experience with

- Docker
- Docker Compose
- Kubernetes
- Deployments
- Services
- StatefulSets
- Persistent Volumes
- Secrets
- Ingress
- Health Probes
- Resource Management
- Troubleshooting Kubernetes applications
- GitHub Actions

---

# Future Improvements

- Helm Charts
- Horizontal Pod Autoscaler
- Monitoring using Prometheus & Grafana
- TLS using cert-manager
- ArgoCD deployment

---

# Author

**Subrata Dey**

GitHub

https://github.com/Subrata2004