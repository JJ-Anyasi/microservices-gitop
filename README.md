# Cloud-Native Microservices with GitOps

A complete 3-service microservices application built with Node.js and Express, containerized with Docker, packaged with Helm charts, and deployed declaratively to Kubernetes using GitOps with Argo CD.

---

## Overview

This project demonstrates a modern DevOps workflow for cloud-native microservices.

### Services

* **Frontend**
  A simple UI that calls the backend for data.

* **Backend**
  Provides a basic API endpoint at `/api/message`.

* **User Service**
  Exposes mock user data via `/users` and `/users/:id`.

### Key Features

* Docker containerization with images pushed to Docker Hub
* Helm charts for templated Kubernetes deployments
* Deployment to a local Kind Kubernetes cluster in the `microservices` namespace
* Declarative GitOps workflow using Argo CD with auto-sync and self-healing
* Inter-service communication via Kubernetes DNS, for example `http://backend:80`
* CI pipeline using GitHub Actions to automatically build and push images on every push to `main`

---

## Architecture
---
config:
  theme: redux
---
sequenceDiagram
    participant Developer as Developer
    participant GitHub as GitHub Repo
    participant Actions as GitHub Actions CI
    participant DockerHub as Docker Hub
    participant ArgoCD as Argo CD
    participant K8s as Kubernetes Cluster (Kind)
    participant Frontend as Frontend Pod
    participant Backend as Backend Pod
    participant UserService as User Service Pod

    Developer->>GitHub: Push code changes to main
    GitHub->>Actions: Trigger CI on push
    Actions->>DockerHub: Build & push images (backend, frontend, user-service)
    GitHub->>ArgoCD: Argo CD watches repo for changes
    ArgoCD->>K8s: Render Helm charts & sync Deployments/Services
    Note over K8s: Namespace: microservices
    Frontend->>Backend: GET /api/message (via DNS http://backend:80)
    Backend->>UserService: GET /users (via DNS http://user-service:80)
    UserService-->>Backend: Return mock users JSON
    Backend-->>Frontend: Return message + timestamp JSON
    Frontend-->>Developer: Render UI with backend data

## Technologies

* **Languages and Frameworks:** Node.js, Express
* **Containerization:** Docker
* **Orchestration:** Kubernetes (Kind for local development)
* **Packaging:** Helm v3
* **GitOps and CD:** Argo CD
* **CI:** GitHub Actions
* **Container Registry:** Docker Hub (`anyasi/*:latest`)

---

## Project Structure

```
microservices-gitops/
├── argocd/                 # Argo CD application manifests
├── charts/
│   ├── backend/
│   ├── frontend/
│   └── user-service/
└── README.md

microservices-app/
├── backend/
├── frontend/
├── user-service/
└── .github/workflows/      # CI pipeline (build-push.yaml)
```

---

## Local Setup

### Prerequisites

* Docker
* Kind
* kubectl
* Helm
* Argo CD CLI (optional)

### Step 1: Create Kubernetes Cluster

```bash
kind create cluster --name dev-project
```

### Step 2: Create Namespace

```bash
kubectl create namespace microservices
```

### Step 3: Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd get pods --watch
```

### Step 4: Access Argo CD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```

Login at:

```
https://localhost:8080
Username: admin
Password: <retrieved password>
```

### Step 5: Deploy Applications

```bash
kubectl apply -f argocd/
kubectl get applications -n argocd -w
kubectl get pods -n microservices -w
```

### Step 6: Access Services

Frontend:

```bash
kubectl port-forward svc/frontend 8080:80 -n microservices
```

Open: http://localhost:8080

User Service:

```bash
kubectl port-forward svc/user-service 8081:80 -n microservices
```

Examples:

* http://localhost:8081/health
* http://localhost:8081/users
* http://localhost:8081/users/1

---

## CI/CD Pipeline

The GitHub Actions workflow located at:

```
.github/workflows/build-push.yaml
```

Performs the following:

* Triggers on push or pull request to `main`
* Builds Docker images for backend, frontend, and user-service
* Pushes `latest` tags to Docker Hub
* Uses build cache to speed up pipelines

Argo CD continuously watches the repository and automatically deploys updated images.

---

## Screenshots

### Argo CD Dashboard
All three applications (backend, frontend, user-service) are **Synced** and **Healthy**.

![Argo CD Overview](screenshots/argocd-dashboard.png)

---

### Pods Running
All microservices pods are in Running state (1/1 Ready) in the `microservices` namespace.

![Pods Running](screenshots/running-pods.png)

---

### Frontend in Action
Frontend UI successfully fetching and displaying data from the backend service.

![Frontend UI](screenshots/frontend-ui.png)

---

### CI/CD Pipeline
GitHub Actions build and Docker image push completed successfully.

![GitHub Actions](screenshots/github-actions.png)

---

### User Service Response
Mock users endpoint returning data.

![User Service Response](screenshots/user-service-users.png)

---

## Challenges and Learnings

* Resolving Helm template helper mismatches
* Configuring Kubernetes DNS for service-to-service communication
* Debugging Argo CD sync issues and namespace conflicts
* Using Helm values and environment variables instead of hard-coded URLs
* Leveraging force sync and pruning for consistent deployments

---

## Future Improvements

* Add an Ingress controller for clean external access
* Implement end-to-end testing in CI
* Add monitoring with Prometheus and Grafana
* Support multiple environments using ApplicationSets
* Introduce secure secret management

---

## Conclusion

This project demonstrates a full GitOps workflow for deploying and managing microservices on Kubernetes, combining containerization, CI/CD automation, and declarative infrastructure practices.