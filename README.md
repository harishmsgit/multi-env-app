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

---

## Kubernetes Deployment with Minikube

To deploy the microservices on Kubernetes using Minikube, use the separate manifests in the `k8s/` folder.

1. Start Minikube:
    ```bash
    minikube start
    ```
2. Use Minikube's Docker daemon so the cluster can access local images:
    ```bash
    minikube -p minikube docker-env
    ```
3. Build each service image inside Minikube:
    ```bash
    docker build -t user-service:latest ./Microservices/user-service
    docker build -t product-service:latest ./Microservices/product-service
    docker build -t order-service:latest ./Microservices/order-service
    docker build -t gateway-service:latest ./Microservices/gateway-service
    ```
4. Apply the Kubernetes manifests:
    ```bash
    kubectl apply -f k8s/user-service.yaml
    kubectl apply -f k8s/product-service.yaml
    kubectl apply -f k8s/order-service.yaml
    kubectl apply -f k8s/gateway-service.yaml
    ```
5. Confirm the deployments and services:
    ```bash
    kubectl get pods,svc
    ```
6. Access the Gateway service locally with port forwarding:
    ```bash
    kubectl port-forward svc/gateway-service 3003:3003
    ```

Then use the local endpoint at `http://localhost:3003/api` and the API routes below:

- `GET /api/users`
- `GET /api/products`
- `GET /api/orders`
- `POST /api/orders`
