# Microservices Kubernetes Submission

This folder contains the Kubernetes deployment and service manifests required to run the User, Product, Order, and Gateway microservices on Minikube.

## Folder structure

- `deployments/`: Deployment manifests for each service
- `services/`: ClusterIP Service manifests for each service
- `screenshots/`: place screenshots of validation outputs here

## Minikube setup

1. Start Minikube:
    ```bash
    minikube start
    ```
2. Use Minikube's Docker daemon so the cluster can access locally built images:
    ```bash
    eval $(minikube -p minikube docker-env)
    ```

## Build images

Build each service image from its service folder:

```bash
docker build -t user-service:latest ./Microservices/user-service
docker build -t product-service:latest ./Microservices/product-service
docker build -t order-service:latest ./Microservices/order-service
docker build -t gateway-service:latest ./Microservices/gateway-service
```

## Deploy the application

Apply the service and deployment manifests:

```bash
kubectl apply -f submission/services
kubectl apply -f submission/deployments
```

## Validate deployment

Check pod and service status:

```bash
kubectl get pods,svc
```

Validate cluster DNS and service discovery with an in-cluster curl pod:

```bash
kubectl run curl-test --image=curlimages/curl:8.4.0 --rm -i --restart=Never -- /usr/bin/curl http://user-service:3000/users
```

Forward the Gateway service locally:

```bash
kubectl port-forward svc/gateway-service 3003:3003
```

Then test the Gateway API:

```bash
curl http://localhost:3003/api/users
curl http://localhost:3003/api/products
curl http://localhost:3003/api/orders
```

## Validate inter-service communication

If you want to test service discovery directly from a pod, use the curl image instead of relying on `curl` inside the service container:

```bash
kubectl run curl-test --image=curlimages/curl:8.4.0 --rm -i --restart=Never -- /usr/bin/curl http://user-service:3000/users
kubectl run curl-test --image=curlimages/curl:8.4.0 --rm -i --restart=Never -- /usr/bin/curl http://product-service:3001/products
kubectl run curl-test --image=curlimages/curl:8.4.0 --rm -i --restart=Never -- /usr/bin/curl http://order-service:3002/orders
```

## Optional Ingress Bonus

Enable the Minikube ingress controller:

```bash
minikube addons enable ingress
```

Apply the ingress manifest:

```bash
kubectl apply -f submission/ingress/ingress.yaml
```

Test the ingress using the host header and Minikube IP:

```bash
curl -H "Host: microservices.local" http://$(minikube ip)/api/users
curl -H "Host: microservices.local" http://$(minikube ip)/api/products
curl -H "Host: microservices.local" http://$(minikube ip)/api/orders
curl -H "Host: microservices.local" http://$(minikube ip)/
```

Host-based routing uses the request Host header to select the correct ingress rule. It is suitable when you want domain-based routing across multiple services.

Path-based routing uses the request URL path itself to route traffic. It is suitable when one hostname needs multiple service routes like `/users`, `/products`, and `/orders`.

For path-based ingress, use direct URLs:

```bash
curl http://$(minikube ip)/users
curl http://$(minikube ip)/products
curl http://$(minikube ip)/orders
curl http://$(minikube ip)/
```
## Troubleshooting

- `ImagePullBackOff`: build images inside Minikube or use `minikube image load`.
- If pods fail, inspect details:
  ```bash
  kubectl describe pod <pod-name>
  kubectl logs <pod-name>
  ```
- Ensure the `gateway-service` Service type remains `ClusterIP` so service discovery works inside the cluster.

## Notes

- All services are configured using `ClusterIP` for cluster-level discovery.
- The Gateway service forwards to backend services by service DNS names: `user-service`, `product-service`, and `order-service`.
