# Todo Platform

This repo is the "platform" for the todo app. It wires together:

- `backend/` – Rails API with GraphQL and Postgres
- `frontend/` – Next.js app that talks to the Rails GraphQL API
- `k8s/` – Kubernetes manifests for Postgres, backend, frontend, and DB seeding

Both `backend` and `frontend` are Git submodules.

---

## Prerequisites

- Docker Desktop installed
- Kubernetes enabled in Docker Desktop (Settings → Kubernetes → "Enable Kubernetes")
- `docker` CLI available
- `kubectl` CLI available

---

## One-time repo setup (first-time dev)

```bash
git clone git@github.com:danbriechle/todo-platform.git
cd todo-platform

# Initialize submodules
git submodule update --init --recursive
```

---

## Build images for Kubernetes

From the todo-platform root:

```bash
docker build -t todo-platform-backend:v1 ./backend
docker build -t todo-platform-frontend:v1 ./frontend
```

These images are what the Kubernetes manifests reference.

## Apply Kubernetes manifests

All resources live in the todo-platform namespace.

Apply them in this order:

```bash
# Namespace
kubectl apply -f k8s/namespace.yaml

# Postgres
kubectl apply -f k8s/postgres.yaml

# Backend (Rails API)
kubectl apply -f k8s/backend.yaml

# Frontend (Next.js)
kubectl apply -f k8s/frontend.yaml

# Run Rails migrations + seeds as a one-off Job
kubectl apply -f k8s/rails-migrate-seed-job.yaml
```

You can check everything with:

```bash
kubectl get pods -n todo-platform
kubectl get jobs -n todo-platform
```

You should eventually see:

postgres-... pod Running

backend-... pod Running

frontend-... pod Running

rails-migrate-seed Job 1/1 Completed

## Accessing the app (dev)

```bash
kubectl port-forward -n todo-platform svc/frontend 3001:3000
```

Then open:
http://localhost:3001/

The frontend talks to the backend inside the cluster at http://backend:3000/graphql.

Note: The rails-migrate-seed Job runs rails db:prepare db:seed inside the cluster,
so you don’t need to run any Rails commands manually to set up the DB.

## Rerunning the seed Job (optional)

```bash
kubectl delete job -n todo-platform rails-migrate-seed
kubectl apply -f k8s/rails-migrate-seed-job.yaml
```
