# Three-Tier Application on Kubernetes

A simple three-tier application deployed on Kubernetes (practised on Killercoda) using
Deployments, ClusterIP Services and an NGINX Ingress controller with path-based routing.

**Author:** Jotiba Bhate

## Architecture

```
                         Client (Browser / curl)
                                  |
                          NGINX Ingress Controller
              ------------------------------------------------
              |                      |                       |
              /                    /api                   /admin
              |                      |                       |
      frontend-service        backend-service          admin-service
         (80 -> 80)             (5000 -> 5000)           (80 -> 80)
              |                      |                       |
   frontend-deployment      backend-deployment       admin-deployment
   nginx:latest (2 pods)    hashicorp/http-echo      httpd:2.4-alpine
                            :1.0.0 (2 pods)          (2 pods)
```

| Tier     | Image                       | Replicas | Container Port | Service Type | URL Path  |
|----------|-----------------------------|----------|----------------|--------------|-----------|
| Frontend | `nginx:latest`              | 2        | 80             | ClusterIP    | `/`       |
| Backend  | `hashicorp/http-echo:1.0.0` | 2        | 5000           | ClusterIP    | `/api`    |
| Admin    | `httpd:2.4-alpine`          | 2        | 80             | ClusterIP    | `/admin`  |

## Project Structure

```
k8s-three-tier-project/
├── README.md
├── k8s/
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── admin-deployment.yaml
│   ├── admin-service.yaml
│   └── ingress.yaml
└── screenshots/
    ├── 01-frontend-browser.jpeg
    ├── 02-backend-api-browser.jpeg
    ├── 03-admin-browser.jpeg
    ├── 04-curl-frontend.png
    ├── 05-curl-backend-api.png
    └── 06-curl-admin.png
```

## Prerequisites

- A running Kubernetes cluster (Killercoda Kubernetes playground, minikube, kind, etc.)
- `kubectl` configured for the cluster
- NGINX Ingress Controller installed (the manifests use `ingressClassName: nginx`)

## Deploy

```bash
# Apply all manifests
kubectl apply -f k8s/

# Check everything
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl get ingress
```

## Test

```bash
curl -i http://localhost:8080/        # Frontend  -> NGINX welcome page
curl -i http://localhost:8080/api     # Backend   -> Backend is working successfully!
curl -i http://localhost:8080/admin   # Admin     -> It works! (Apache httpd)
```

Change `8080` to the port where your Ingress controller is exposed.

## How the Routing Works

- `app-ingress` sends `/api` to `backend-service` and `/` to `frontend-service`.
- `admin-ingress` uses a regex path `/admin(/|$)(.*)` with
  `rewrite-target: /$2`, so `/admin` reaches the admin pod as `/`.

## Clean Up

```bash
kubectl delete -f k8s/
```

## Screenshots

| Browser: Frontend | Browser: Backend | Browser: Admin |
|---|---|---|
| ![](screenshots/01-frontend-browser.jpeg) | ![](screenshots/02-backend-api-browser.jpeg) | ![](screenshots/03-admin-browser.jpeg) |

curl outputs: see `screenshots/04` to `06`.
