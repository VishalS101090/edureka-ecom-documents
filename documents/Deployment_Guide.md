# Microservices Project: Deployment Guide

**Phase:** Deployment (Docker & Kubernetes)
**Goal:** Containerize the business microservices and deploy them to a Kubernetes cluster.

---

## Prerequisites

- Docker installed & running
- `kubectl` configured (connected to your AKS / Minikube cluster)
- Docker Hub account (for pushing images)
- Maven installed (or use the included `./mvnw` wrapper)

---

## Part 1 — Dockerize Services

Create a `Dockerfile` in the root of each microservice project. Example Dockerfile (use the same pattern for all services; update `EXPOSE` and labels per service):

### Example Dockerfile (product-service)
```dockerfile
FROM eclipse-temurin:17-jdk

LABEL authors="vishal"

WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 5003

RUN apt-get update && apt-get install -y gcc curl

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Repeat the same pattern for other services, replacing the `EXPOSE` port:

- Order Service: `EXPOSE 5004`
- Inventory Service: `EXPOSE 5005`
- Customer Service: `EXPOSE 5006`
- Payment Service: `EXPOSE 5007`

---

## Part 2 — Build & Push Images

Replace `<your-user>` with your Docker Hub username when tagging images.

1) Build the JAR for each service:

```bash
mvn clean package -DskipTests
```

2) Build and push Docker images (example for product-service):

```bash
docker build -t <your-user>/product-service:latest ./product-service
docker push <your-user>/product-service:latest
```

Repeat for each service (order-service, inventory-service, customer-service, payment-service).

---

## Part 3 — Kubernetes Deployment Manifests

Create a `k8s/` folder and add a YAML file per service. Replace `<your-user>` and verify environment values match your cluster services (mongodb, kafka, discovery-service).

### product-service (k8s/product-service.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
    spec:
      containers:
      - name: product-service
        image: <your-user>/product-service:latest
        ports:
        - containerPort: 5003
        env:
        - name: SPRING_DATA_MONGODB_URI
          value: "mongodb://mongodb:27017/product_db"
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://discovery-service:5001/eureka/"
---
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product-service
  ports:
    - protocol: TCP
      port: 5003
      targetPort: 5003
```

### order-service (k8s/order-service.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: <your-user>/order-service:latest
        ports:
        - containerPort: 5004
        env:
        - name: SPRING_DATA_MONGODB_URI
          value: "mongodb://mongodb:27017/order_db"
        - name: SPRING_KAFKA_BOOTSTRAP_SERVERS
          value: "kafka:9092"
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://discovery-service:5001/eureka/"
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - protocol: TCP
      port: 5004
      targetPort: 5004
```

### inventory-service (k8s/inventory-service.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inventory-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: inventory-service
  template:
    metadata:
      labels:
        app: inventory-service
    spec:
      containers:
      - name: inventory-service
        image: <your-user>/inventory-service:latest
        ports:
        - containerPort: 5005
        env:
        - name: SPRING_DATA_MONGODB_URI
          value: "mongodb://mongodb:27017/inventory_db"
        - name: SPRING_KAFKA_BOOTSTRAP_SERVERS
          value: "kafka:9092"
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://discovery-service:5001/eureka/"
---
apiVersion: v1
kind: Service
metadata:
  name: inventory-service
spec:
  selector:
    app: inventory-service
  ports:
    - protocol: TCP
      port: 5005
      targetPort: 5005
```

### customer-service (k8s/customer-service.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: customer-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: customer-service
  template:
    metadata:
      labels:
        app: customer-service
    spec:
      containers:
      - name: customer-service
        image: <your-user>/customer-service:latest
        ports:
        - containerPort: 5006
        env:
        - name: SPRING_DATA_MONGODB_URI
          value: "mongodb://mongodb:27017/customer_db"
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://discovery-service:5001/eureka/"
---
apiVersion: v1
kind: Service
metadata:
  name: customer-service
spec:
  selector:
    app: customer-service
  ports:
    - protocol: TCP
      port: 5006
      targetPort: 5006
```

### payment-service (k8s/payment-service.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: payment-service
  template:
    metadata:
      labels:
        app: payment-service
    spec:
      containers:
      - name: payment-service
        image: <your-user>/payment-service:latest
        ports:
        - containerPort: 5007
        env:
        - name: SPRING_DATA_MONGODB_URI
          value: "mongodb://mongodb:27017/payment_db"
        - name: SPRING_KAFKA_BOOTSTRAP_SERVERS
          value: "kafka:9092"
        - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
          value: "http://discovery-service:5001/eureka/"
---
apiVersion: v1
kind: Service
metadata:
  name: payment-service
spec:
  selector:
    app: payment-service
  ports:
    - protocol: TCP
      port: 5007
      targetPort: 5007
```

---

## Part 4 — Deploy to Cluster

1. Ensure infrastructure services (MongoDB, Kafka, Eureka/discovery-service) are deployed and reachable in the cluster.

2. Apply manifests:

```bash
kubectl apply -f k8s/product-service.yaml
kubectl apply -f k8s/order-service.yaml
kubectl apply -f k8s/inventory-service.yaml
kubectl apply -f k8s/customer-service.yaml
kubectl apply -f k8s/payment-service.yaml
```

3. Verify pods:

```bash
kubectl get pods
```

Expected: each service pod shows `1/1` READY and `Running` status.

4. If a pod fails, check logs:

```bash
kubectl logs deployment/order-service
```
