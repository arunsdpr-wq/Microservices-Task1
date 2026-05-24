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

## Instructions
1. Start all services using the `docker-compose` file:
   ```
   docker-compose up
   ```
2. Once the services are running, use the above endpoints to verify the functionality.

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

```powershell
kubectl apply -f k8s/all.yaml
```

4) Validate pods and services

```powershell
kubectl get pods
kubectl get svc
kubectl get deploy
```

5) Test inter-service communication via the Gateway

Port-forward the Gateway service to localhost:

```powershell
kubectl port-forward svc/gateway-service 3003:3003
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
kubectl logs -l app=gateway-service
kubectl logs -l app=order-service
```

7) (Optional Bonus) Enable Ingress and apply routing

```powershell
minikube addons enable ingress
kubectl apply -f k8s/ingress.yaml
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
- `k8s/all.yaml` : Deployments + Services for all services
- `k8s/ingress.yaml` : Optional Ingress resource (requires `minikube addons enable ingress`)

