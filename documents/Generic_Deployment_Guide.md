# Generic Microservice Deployment Guide

This document provides step-by-step instructions for building, containerizing, and deploying any microservice in the Edureka E-Commerce platform to Docker and Kubernetes.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Build Application](#build-application)
- [Docker Deployment](#docker-deployment)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Tools
- Java 17 JDK
- Maven 3.6+
- Docker Desktop (with Kubernetes enabled)
- kubectl (Kubernetes CLI) - included with Docker Desktop
- Docker Hub account (or private registry access)

### Required Access
- Docker registry credentials (Docker Hub: `vishal101090`)
- Kubernetes cluster access

---

## Build Application

### Step 1: Navigate to Service Directory
```bash
cd git_src/<service-name>
```

**Example:**
```bash
cd git_src/edureka-ecom-discovery-server
```

### Step 2: Clean and Build with Maven
```bash
mvn clean package -DskipTests
```

**Or with tests:**
```bash
mvn clean package
```

### Step 3: Verify JAR File
```bash
ls -l target/*.jar
```

Expected output: `target/<service-name>-<version>.jar`

---

## Docker Deployment

### Step 1: Build Docker Image

**Generic Command:**
```bash
docker build -t <dockerhub-username>/<service-name>:<tag> .
```

**Example:**
```bash
docker build -t vishal101090/discovery-service:latest .
```

**Build with specific tag:**
```bash
docker build -t vishal101090/discovery-service:v1.0.0 .
```

### Step 2: Verify Image Created
```bash
docker images | grep <service-name>
```

**Example:**
```bash
docker images | grep discovery-service
```

### Step 3: Test Docker Image Locally (Optional)
```bash
docker run -p <host-port>:<container-port> <dockerhub-username>/<service-name>:<tag>
```

**Example:**
```bash
docker run -p 5001:5001 vishal101090/discovery-service:latest
```

### Step 4: Login to Docker Registry
```bash
docker login
```

Enter your Docker Hub username and password when prompted.

### Step 5: Push Image to Registry

**Generic Command:**
```bash
docker push <dockerhub-username>/<service-name>:<tag>
```

**Example:**
```bash
docker push vishal101090/discovery-service:latest
```

### Step 6: Verify Image in Registry
Visit: `https://hub.docker.com/r/<username>/<service-name>/tags`

Or use CLI:
```bash
docker pull <dockerhub-username>/<service-name>:<tag>
```

---

## Kubernetes Deployment

### Step 1: Ensure Kubernetes Cluster is Running

**Verify Docker Desktop Kubernetes is enabled:**
- Open Docker Desktop
- Go to Settings → Kubernetes
- Ensure "Enable Kubernetes" is checked
- Click "Apply & Restart" if needed

**Verify kubectl access:**
```bash
kubectl cluster-info
kubectl get nodes
```

**Verify correct context:**
```bash
kubectl config current-context
```

Should show: `docker-desktop`

### Step 2: Navigate to k8s Directory
```bash
cd k8s
```

**Example:**
```bash
cd git_src/edureka-ecom-discovery-server/k8s
```

### Step 3: Apply Kubernetes Manifests

**Generic Command:**
```bash
kubectl apply -f <service-name>.yaml
```

**Example:**
```bash
kubectl apply -f discovery-server.yaml
```

**Expected Output:**
```
deployment.apps/<deployment-name> created
service/<service-name> created
```

### Step 4: Verify Deployment

**Check Deployment Status:**
```bash
kubectl get deployments
```

**Check Pods:**
```bash
kubectl get pods
```

**Check Services:**
```bash
kubectl get services
```

**Detailed Pod Information:**
```bash
kubectl describe pod <pod-name>
```

**Check Pod Logs:**
```bash
kubectl logs <pod-name>
```

**Follow logs in real-time:**
```bash
kubectl logs -f <pod-name>
```

### Step 5: Access the Service

**For LoadBalancer type services:**

Docker Desktop Kubernetes supports LoadBalancer services directly. The service will be accessible on `localhost`.

```bash
kubectl get service <service-name>
```

Access via: `http://localhost:<port>`

**Example:**
```bash
kubectl get service discovery-service
```

Access at: `http://localhost:5001`

**For NodePort services:**
```bash
kubectl get service <service-name>
```

Access via: `http://localhost:<node-port>`

**Port forwarding (alternative):**
```bash
kubectl port-forward service/<service-name> <local-port>:<service-port>
```

**Example:**
```bash
kubectl port-forward service/discovery-service 5001:5001
```

Then access: `http://localhost:5001`

---

## Verification

### Health Check
```bash
curl http://localhost:<port>/actuator/health
```

**Example:**
```bash
curl http://localhost:5001/actuator/health
```

### Check Application Info
```bash
curl http://localhost:<port>/actuator/info
```

### Verify Service Registration (For Eureka Discovery)
```bash
curl http://localhost:5001/eureka/apps
```

Or open in browser: `http://localhost:5001`

---

## Kubernetes Management Commands

### Update Deployment (After New Build)

**Step 1: Build and push new image with updated tag**
```bash
docker build -t <dockerhub-username>/<service-name>:<new-tag> .
docker push <dockerhub-username>/<service-name>:<new-tag>
```

**Step 2: Update image in Kubernetes**
```bash
kubectl set image deployment/<deployment-name> <container-name>=<dockerhub-username>/<service-name>:<new-tag>
```

**Example:**
```bash
kubectl set image deployment/discovery-deployment discovery-server=vishal101090/discovery-service:v1.0.1
```

**Or reapply the YAML file:**
```bash
kubectl apply -f <service-name>.yaml
```

**Force rollout restart:**
```bash
kubectl rollout restart deployment/<deployment-name>
```

### Scale Deployment
```bash
kubectl scale deployment/<deployment-name> --replicas=<number>
```

**Example:**
```bash
kubectl scale deployment/discovery-deployment --replicas=3
```

### Delete Resources
```bash
kubectl delete -f <service-name>.yaml
```

**Or delete individually:**
```bash
kubectl delete deployment <deployment-name>
kubectl delete service <service-name>
```

### View Resource Usage
```bash
kubectl top pods
kubectl top nodes
```

---

## Troubleshooting

### Pod Not Starting

**Check pod status:**
```bash
kubectl get pods
kubectl describe pod <pod-name>
```

**Common issues:**
- Image pull errors: Check image name and registry access
- Resource limits: Check if cluster has enough resources
- Configuration errors: Check environment variables

### View Pod Logs
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # For crashed containers
```

### Execute Commands in Pod
```bash
kubectl exec -it <pod-name> -- /bin/bash
```

**Check Java process:**
```bash
kubectl exec -it <pod-name> -- ps aux | grep java
```

### Image Pull Issues

**Check image pull policy:**
- `Always`: Always pull from registry
- `IfNotPresent`: Use local image if available
- `Never`: Never pull from registry

**Docker Desktop uses local Docker images automatically:**

If you have built an image locally with Docker Desktop, Kubernetes can use it directly with `imagePullPolicy: Always` or `imagePullPolicy: Never`.

### Service Not Accessible

**Check service endpoints:**
```bash
kubectl get endpoints <service-name>
```

**Check if pods are ready:**
```bash
kubectl get pods -l app=<app-label>
```

**For Docker Desktop LoadBalancer services:**

Docker Desktop automatically handles LoadBalancer services on localhost. No additional tunnel is needed.

### Restart Failed Pods
```bash
kubectl delete pod <pod-name>
# Deployment controller will automatically create a new pod
```

### View Events
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Quick Reference Commands

### Docker
```bash
# Build
docker build -t <image-name>:<tag> .

# Push
docker push <image-name>:<tag>

# Run locally
docker run -p <host-port>:<container-port> <image-name>:<tag>

# List images
docker images

# Remove image
docker rmi <image-name>:<tag>
```

### Kubernetes
```bash
# Apply configuration
kubectl apply -f <file.yaml>

# Get resources
kubectl get all
kubectl get pods
kubectl get services
kubectl get deployments

# Describe resources
kubectl describe pod <pod-name>
kubectl describe service <service-name>

# Logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>

# Delete resources
kubectl delete -f <file.yaml>
kubectl delete pod <pod-name>

# Port forward
kubectl port-forward service/<service-name> <local-port>:<service-port>
```

---

## Best Practices

1. **Tagging Strategy:**
   - Use semantic versioning: `v1.0.0`, `v1.0.1`
   - Maintain `latest` tag for production
   - Use feature/environment tags: `dev`, `staging`, `prod`

2. **Testing:**
   - Test locally with Docker before pushing
   - Test in dev Kubernetes cluster before production
   - Run integration tests after deployment

3. **Resource Management:**
   - Define resource limits in Kubernetes manifests
   - Monitor resource usage regularly
   - Scale based on load requirements

4. **Security:**
   - Never commit credentials to git
   - Use Kubernetes secrets for sensitive data
   - Regularly update base images for security patches

5. **Monitoring:**
   - Enable Spring Boot Actuator endpoints
   - Use kubectl logs for troubleshooting
   - Implement health checks and readiness probes

---

## Next Steps

After deploying a service:
1. Verify service health
2. Check service registration (if using Eureka)
3. Test API endpoints
4. Monitor logs for errors
5. Document any service-specific configurations

For complete deployment workflow, refer to [Deployment_Guide.md](Deployment_Guide.md)
