# Container & Kubernetes Assessment – Evidence of Understanding

## Project Overview

This project demonstrates containerization and Kubernetes deployment of a simple backend application with MongoDB. The goal is to show understanding of Docker, Docker Compose, and Kubernetes basics (Deployments, Services, ConfigMaps, Secrets, and health checks).

The backend exposes a `/health` endpoint used for readiness and liveness checks.

---

## Folder Structure

```
container-assessment/
├── app/                     # Application source code
├── Dockerfile               # Docker image definition
├── docker-compose.yml       # Local multi-container setup
├── scripts/
│   ├── docker-build.sh      # Builds backend image
│   └── docker-run.sh        # Runs containers using compose
├── kubernetes/
│   ├── namespace.yaml
│   ├── backend/
│   │   ├── backend-deployment.yaml
│   │   ├── backend-service.yaml
│   │   ├── backend-configmap.yaml
│   │   └── backend-secret.yaml
│   └── mongodb/
│       ├── mongodb-deployment.yaml
│       ├── mongodb-service.yaml
│       └── mongodb-pvc.yaml
└── README.md
```

---

## Docker (Local Containerization)

### Build the backend image

```bash
cd container-assessment
./scripts/docker-build.sh
```

### Run backend and MongoDB with Docker Compose

```bash
./scripts/docker-run.sh
```

### Verify running containers

```bash
docker compose ps
```

### Test health endpoint

```bash
curl http://localhost:3000/health
```

Expected output:

```json
{"status":"OK"}
```

---

## Kubernetes Deployment

### Create Kind cluster

```bash
kind create cluster --name muchtodo-cluster
```

### Apply namespace

```bash
kubectl apply -f kubernetes/namespace.yaml
```

### Deploy MongoDB

```bash
kubectl apply -f kubernetes/mongodb/
```

### Deploy backend application

```bash
kubectl apply -f kubernetes/backend/
```

### Check running resources

```bash
kubectl get pods -n muchtodo
kubectl get svc -n muchtodo
```

---

## Accessing the Application

Because the cluster is running locally with **Kind**, NodePort is not directly reachable via `localhost`. To access the backend service, port-forwarding is used.

### Port-forward backend service

```bash
kubectl port-forward svc/backend-service 8080:3000 -n muchtodo
```

### Test health endpoint

```bash
curl http://localhost:8080/health
```

Expected output:

```json
{"status":"OK"}
```

---

## Kubernetes Concepts Demonstrated

* **Deployment**: Used for backend and MongoDB to manage pods and replicas
* **Service**:

  * NodePort for backend exposure
  * ClusterIP for internal MongoDB communication
* **ConfigMap & Secret**: Used to inject environment variables securely
* **Probes**:

  * Liveness probe checks `/health`
  * Readiness probe ensures traffic only reaches healthy pods
* **PersistentVolumeClaim**: Ensures MongoDB data persistence

---

## Screenshots Submitted

1. Docker image build success
2. Docker Compose running containers
3. `kubectl get pods -n muchtodo`
4. `kubectl get svc -n muchtodo`
5. Port-forward command running
6. Successful `/health` endpoint response

---

## Conclusion

This project shows a complete flow from local containerization to Kubernetes deployment. All services run successfully, health checks pass, and the application is accessible through Kubernetes using port-forwarding.

## Problems Encountered and How They Were Resolved

### 1. `npm ci` Failing During Docker Build

**Problem:**
The Docker build failed because `npm ci` requires a `package-lock.json` file, which was missing.

**Solution:**
Switched to `npm install` and ensured the correct `package.json` path was copied into the image.

---

### 2. Backend Image Not Found in Kubernetes (ImagePullBackOff)

**Problem:**
Kubernetes could not pull the backend image because it was built locally and not available inside the Kind cluster.

**Solution:**
Rebuilt the image within the Kind environment and set `imagePullPolicy: IfNotPresent`.

---

### 3. MongoDB CrashLoopBackOff (AVX CPU Issue)

**Problem:**
MongoDB versions 5+ require CPU AVX support, which is not available in the Vagrant/Kind environment.

**Solution:**
Changed the MongoDB image to a version compatible with non-AVX CPUs.

---

### 4. NodePort Service Not Accessible via Browser

**Problem:**
Accessing the backend via NodePort (`localhost:30080`) did not work when using Kind.

**Solution:**
Used port forwarding to expose the service locally:

```bash
kubectl port-forward svc/backend-service 8080:3000 -n muchtodo
```

---

### 5. Port Already in Use Error

**Problem:**
Port `3000` was already in use on the host machine, causing port-forwarding to fail.

**Solution:**
Mapped the backend service to port `8080` instead.

---

These issues and fixes demonstrate real-world troubleshooting with Docker, Kubernetes, and containerized applications.
