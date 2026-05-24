# Microservices-Task

## Overview
This document provides details on testing various services after running the `docker-compose` file. These services include User, Product, Order, and Gateway Services. Each service has its own endpoints for testing purposes.

---

## Services and Endpoints

### **User Service**
- **Base URL:** `http://localhost:3000`
- **Endpoints:**
  - **List Users:**  
    ```
    curl http://localhost:3000/users
    ```
    Or open in your browser: [http://localhost:3000/users](http://localhost:3000/users)

---

### **Product Service**
- **Base URL:** `http://localhost:3001`
- **Endpoints:**
  - **List Products:**  
    ```
    curl http://localhost:3001/products
    ```
    Or open in your browser: [http://localhost:3001/products](http://localhost:3001/products)

---

### **Order Service**
- **Base URL:** `http://localhost:3002`
- **Endpoints:**
  - **List Orders:**  
    ```
    curl http://localhost:3002/orders
    ```
    Or open in your browser: [http://localhost:3002/orders](http://localhost:3002/orders)

---

### **Gateway Service**
- **Base URL:** `http://localhost:3003/api`
- **Endpoints:**
  - **Users:**  
    ```
    curl http://localhost:3003/api/users
    ```
  - **Products:**  
    ```
    curl http://localhost:3003/api/products
    ```
  - **Orders:**  
    ```
    curl http://localhost:3003/api/orders
    ```

---

## Docker Compose Deployment

### Prerequisites
- Docker and Docker Compose installed
- PowerShell (Windows) or a POSIX shell

### Setup Steps

1) **Build and start all services:**

```powershell
docker-compose up --build
```

The `--build` flag rebuilds images from Dockerfiles. Omit it on subsequent runs if images are already built.

2) **Verify services are running:**

```powershell
docker-compose ps
```

Expected output: all four containers should show `Up` status.

3) **Test individual services:**

User Service:
```powershell
curl http://localhost:3000/health
curl http://localhost:3000/users
```

Product Service:
```powershell
curl http://localhost:3001/health
curl http://localhost:3001/products
```

Order Service:
```powershell
curl http://localhost:3002/health
curl http://localhost:3002/orders
```

4) **Test gateway inter-service communication:**

```powershell
curl http://localhost:3003/health
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders
curl -X POST http://localhost:3003/api/orders -H "Content-Type: application/json" -d '{"userId":1,"productId":1}'
```

5) **View service logs:**

All services:
```powershell
docker-compose logs -f
```

Specific service (e.g., gateway):
```powershell
docker-compose logs -f gateway-service
```

6) **Stop and clean up:**

```powershell
docker-compose down
```

### Docker Compose File Details

The `docker-compose.yaml` includes:
- **Build context:** Each service builds from its Dockerfile
- **Networking:** All services on `microservices-network` bridge network for inter-service communication
- **Port mappings:** User (3000), Product (3001), Order (3002), Gateway (3003)
- **Health checks:** Each service has HTTP `/health` endpoint probes
- **Dependency ordering:** Gateway depends on other services being healthy before starting
- **Environment variables:** Service URLs passed to Gateway for internal communication

Happy testing!

## Kubernetes (Minikube) Deployment

Follow these steps to deploy the four Node.js microservices to Minikube and validate inter-service communication.

Prerequisites:
- `minikube`, `kubectl`, and `docker` installed and on PATH
- PowerShell (Windows) or a POSIX shell

1) Start Minikube

```powershell
minikube start
```

2) Build or load container images

Method A — build images inside Minikube's Docker daemon (recommended):

```powershell
minikube -p minikube docker-env --shell powershell | Invoke-Expression
docker build -t user-service:latest ./Microservices/user-service
docker build -t product-service:latest ./Microservices/product-service
docker build -t order-service:latest ./Microservices/order-service
docker build -t gateway-service:latest ./Microservices/gateway-service
```

Method B — build locally then load into Minikube (alternative):

```powershell
docker build -t user-service:latest ./Microservices/user-service
docker build -t product-service:latest ./Microservices/product-service
docker build -t order-service:latest ./Microservices/order-service
docker build -t gateway-service:latest ./Microservices/gateway-service
minikube image load user-service:latest
minikube image load product-service:latest
minikube image load order-service:latest
minikube image load gateway-service:latest
```

3) Apply Kubernetes manifests

First, create the namespace:

```powershell
kubectl apply -f k8s/namespace.yaml
```

Then apply all services and deployments:

```powershell
kubectl apply -f k8s/all.yaml
```

4) Validate pods and services

```powershell
kubectl get pods -n microservices
kubectl get svc -n microservices
kubectl get deploy -n microservices
```

5) Test inter-service communication via the Gateway

Port-forward the Gateway service to localhost:

```powershell
kubectl port-forward -n microservices svc/gateway-service 3003:3003
```

In another shell, call the gateway endpoints (these forward to other services inside cluster):

```powershell
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders
curl -X POST http://localhost:3003/api/orders -H "Content-Type: application/json" -d '{"userId":1,"productId":1}'
```

6) Inspect logs to validate inter-service calls

```powershell
kubectl logs -n microservices -l app=gateway-service
kubectl logs -n microservices -l app=order-service
```

7) (Optional Bonus) Enable Ingress and apply routing

```powershell
minikube addons enable ingress
kubectl apply -f k8s/ingress.yaml
kubectl get ingress -n microservices
minikube ip    # add mapping for micro.local in your hosts file: <MINIKUBE_IP> micro.local
```

Then open `http://micro.local/api/users` etc. (you may need to add hosts entry as instructed above).

Troubleshooting tips
- If a pod is CrashLoopBackOff: `kubectl describe pod <pod>` and `kubectl logs <pod>` to see errors.
- If services are unreachable, ensure deployments are `Running` and `READY` has at least one ready pod.
- If images are not found, make sure images are built in Minikube's Docker daemon or loaded with `minikube image load`.

Screenshots
- Capture `kubectl get pods` output and logs from the Gateway demonstrating successful requests and add them to this repository or include them in your submission.

Files added
- `k8s/namespace.yaml` : Namespace resource for microservices isolation
- `k8s/all.yaml` : Deployments + Services for all services
- `k8s/ingress.yaml` : Optional Ingress resource (requires `minikube addons enable ingress`)

