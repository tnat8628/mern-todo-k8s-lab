# MERN Todo Kubernetes DevOps Lab

This project is a DevOps/Kubernetes lab based on a MERN Todo application.
The original app contains:

* React frontend
* Node.js/Express backend
* MongoDB database

This lab extends the original project with:

* Dockerized frontend and backend
* Kubernetes Deployments and Services
* MongoDB StatefulSet with persistent storage
* Nginx Ingress path-based routing
* ConfigMap and Secret management
* Health checks with readiness/liveness probes
* Resource requests and limits
* Backend scaling with multiple replicas
* Horizontal Pod Autoscaler
* Docker Hub image registry
* GitHub Actions CI workflow for building and pushing Docker images

---

## 1. Architecture Overview

```text
Browser
  |
  | http://mern-todo.local
  v
Nginx Ingress Controller
  |
  +-- /api  -> backend-service:8000
  |
  +-- /     -> frontend-service:80

backend-service
  |
  +-- backend Pod 1
  +-- backend Pod 2
        |
        v
   mongo-service:27017
        |
        v
   mongo-0 StatefulSet + PVC
```

---

## 2. Tech Stack

### Application

* Frontend: React
* Backend: Node.js, Express
* Database: MongoDB

### DevOps / Infrastructure

* Docker
* Minikube
* Kubernetes
* Nginx Ingress Controller
* Docker Hub
* GitHub Actions
* Kubernetes ConfigMap
* Kubernetes Secret
* StatefulSet
* PersistentVolumeClaim
* Horizontal Pod Autoscaler

---

## 3. Project Structure

```text
mern-todo-app/
  backend/
    Dockerfile
    .dockerignore
    server.js
    package.json

  frontend/
    Dockerfile
    .dockerignore
    src/
    package.json

  k8s/
    mongo-service.yaml
    mongo-statefulset.yaml
    backend-configmap.yaml
    backend-secret.example.yaml
    backend-deployment.yaml
    backend-service.yaml
    frontend-deployment.yaml
    frontend-service.yaml
    ingress.yaml
    backend-hpa.yaml

  .github/
    workflows/
      docker-build-push.yml

  README.md
```

---

## 4. Docker Images

The application images are pushed to Docker Hub:

```text
tnat8628/mern-todo-backend:latest
tnat8628/mern-todo-frontend:latest
```

The GitHub Actions workflow also pushes images with the commit SHA as a tag.

Example:

```text
tnat8628/mern-todo-backend:<commit-sha>
tnat8628/mern-todo-frontend:<commit-sha>
```

---

## 5. Kubernetes Resources

### MongoDB

MongoDB is deployed using a StatefulSet because it is a stateful component and needs persistent storage.

Files:

```text
k8s/mongo-service.yaml
k8s/mongo-statefulset.yaml
```

Resources:

* `mongo-service`
* `mongo` StatefulSet
* `mongo-data-mongo-0` PVC

MongoDB connection string used by backend:

```text
mongodb://mongo-service:27017/mern-todo
```

---

### Backend

Backend is deployed as a Kubernetes Deployment because it is stateless.

Files:

```text
k8s/backend-deployment.yaml
k8s/backend-service.yaml
k8s/backend-configmap.yaml
k8s/backend-secret.example.yaml
k8s/backend-hpa.yaml
```

Backend features:

* 2 replicas
* Health check endpoint: `/health`
* Readiness probe
* Liveness probe
* Resource requests and limits
* HPA support
* Environment variables from ConfigMap and Secret

---

### Frontend

Frontend is built as a static React application and served by Nginx.

Files:

```text
k8s/frontend-deployment.yaml
k8s/frontend-service.yaml
```

Frontend features:

* Multi-stage Docker build
* Nginx static file serving
* Readiness probe
* Liveness probe
* Resource requests and limits

---

### Ingress

Ingress is used to route traffic by path.

File:

```text
k8s/ingress.yaml
```

Routes:

```text
/      -> frontend-service:80
/api   -> backend-service:8000
```

Domain:

```text
mern-todo.local
```

---

## 6. Prerequisites

Install the following tools:

```text
Docker
Minikube
kubectl
Git
```

Check versions:

```bash
docker --version
minikube version
kubectl version --client
git --version
```

---

## 7. Start Minikube

```bash
minikube start --driver=docker
```

Check cluster status:

```bash
minikube status
kubectl get nodes -o wide
```

---

## 8. Create Namespace

```bash
kubectl create namespace mern-todo
```

Check:

```bash
kubectl get ns
```

---

## 9. Deploy MongoDB

```bash
kubectl apply -f k8s/mongo-service.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
```

Check:

```bash
kubectl get pods -n mern-todo
kubectl get pvc -n mern-todo
kubectl get svc -n mern-todo
```

Expected:

```text
mongo-0   1/1   Running
```

Test MongoDB:

```bash
kubectl exec -it mongo-0 -n mern-todo -- mongosh
```

Inside Mongo shell:

```javascript
show dbs
exit
```

---

## 10. Configure Backend Environment

The backend uses:

```text
PORT
MONGO_URI
JWT_SECRET
GMAIL_USERNAME
GMAIL_PASSWORD
```

Apply ConfigMap:

```bash
kubectl apply -f k8s/backend-configmap.yaml
```

Create a real local Secret file from the example:

```bash
cp k8s/backend-secret.example.yaml k8s/backend-secret.yaml
nano k8s/backend-secret.yaml
```

Apply Secret:

```bash
kubectl apply -f k8s/backend-secret.yaml
```

Important:

```text
k8s/backend-secret.yaml is ignored by Git.
Only k8s/backend-secret.example.yaml should be committed.
```

---

## 11. Deploy Backend

```bash
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
kubectl apply -f k8s/backend-hpa.yaml
```

Check:

```bash
kubectl get deployment -n mern-todo
kubectl get pods -n mern-todo -l app=backend -o wide
kubectl get svc -n mern-todo
kubectl get hpa -n mern-todo
```

Check logs:

```bash
kubectl logs -l app=backend -n mern-todo
```

Expected log:

```text
Listening on localhost:8000
DB Connected
```

---

## 12. Deploy Frontend

```bash
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml
```

Check:

```bash
kubectl get deployment -n mern-todo
kubectl get pods -n mern-todo -l app=frontend -o wide
kubectl get svc -n mern-todo
```

---

## 13. Enable Minikube Ingress

```bash
minikube addons enable ingress
```

Check:

```bash
kubectl get pods -n ingress-nginx
```

Expected:

```text
ingress-nginx-controller   1/1   Running
```

---

## 14. Deploy Ingress

```bash
kubectl apply -f k8s/ingress.yaml
```

Check:

```bash
kubectl get ingress -n mern-todo
```

Expected:

```text
mern-todo-ingress   nginx   mern-todo.local   192.168.49.2   80
```

---

## 15. Configure Local DNS

Get Minikube IP:

```bash
minikube ip
```

Edit `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Add:

```text
<MINIKUBE_IP> mern-todo.local
```

Example:

```text
192.168.49.2 mern-todo.local
```

Test:

```bash
ping -c 2 mern-todo.local
```

---

## 16. Test Application

Test frontend:

```bash
curl -I http://mern-todo.local
```

Expected:

```text
HTTP/1.1 200 OK
```

Test backend route through Ingress:

```bash
curl -i http://mern-todo.local/api/task/getTask
```

Expected:

```text
HTTP/1.1 401 Unauthorized
{"message":"Unauthorized"}
```

This is expected because the route requires JWT authentication.

---

## 17. Health Checks

Backend health endpoint:

```bash
kubectl run curl-test \
  -n mern-todo \
  --image=curlimages/curl:latest \
  --rm -it \
  --restart=Never \
  -- sh
```

Inside the test Pod:

```sh
curl http://backend-service:8000/health
exit
```

Expected:

```json
{"status":"ok","service":"backend"}
```

---

## 18. Self-Healing Test

Delete one backend Pod:

```bash
kubectl get pods -n mern-todo -l app=backend
kubectl delete pod <BACKEND_POD_NAME> -n mern-todo
```

Watch Kubernetes recreate it:

```bash
kubectl get pods -n mern-todo -l app=backend -w
```

Kubernetes will automatically create a new Pod because the Deployment declares the desired number of replicas.

---

## 19. Scaling

Backend uses 2 replicas:

```yaml
replicas: 2
```

Check:

```bash
kubectl get deployment backend -n mern-todo
kubectl get pods -n mern-todo -l app=backend -o wide
```

Check Service endpoints:

```bash
kubectl get endpoints backend-service -n mern-todo
```

Expected:

```text
backend-service   10.244.x.x:8000,10.244.x.y:8000
```

---

## 20. Horizontal Pod Autoscaler

Enable Metrics Server:

```bash
minikube addons enable metrics-server
```

Check metrics:

```bash
kubectl top pods -n mern-todo
```

Check HPA:

```bash
kubectl get hpa -n mern-todo
```

Expected:

```text
backend-hpa   Deployment/backend   cpu: x%/50%   2   5   2
```

---

## 21. GitHub Actions CI

The workflow file is located at:

```text
.github/workflows/docker-build-push.yml
```

The workflow runs on push to `master`, but only when these paths change:

```text
backend/**
frontend/**
.github/workflows/docker-build-push.yml
```

Workflow tasks:

```text
1. Checkout source code
2. Set up Docker Buildx
3. Login to Docker Hub
4. Build and push backend image
5. Build and push frontend image
```

Images pushed:

```text
tnat8628/mern-todo-backend:latest
tnat8628/mern-todo-backend:<commit-sha>

tnat8628/mern-todo-frontend:latest
tnat8628/mern-todo-frontend:<commit-sha>
```

---

## 22. Required GitHub Secrets

The workflow requires these repository secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

Set them in:

```text
GitHub Repository
-> Settings
-> Secrets and variables
-> Actions
-> New repository secret
```

---

## 23. Deploy Latest Images

The Kubernetes Deployments use:

```text
tnat8628/mern-todo-backend:latest
tnat8628/mern-todo-frontend:latest
```

After GitHub Actions builds and pushes new images, restart deployments:

```bash
kubectl rollout restart deployment/backend -n mern-todo
kubectl rollout restart deployment/frontend -n mern-todo
```

Check rollout:

```bash
kubectl rollout status deployment/backend -n mern-todo
kubectl rollout status deployment/frontend -n mern-todo
```

Test:

```bash
curl -I http://mern-todo.local
curl -i http://mern-todo.local/api/task/getTask
```

---

## 24. Useful Commands

View all resources:

```bash
kubectl get all -n mern-todo
```

View Pods:

```bash
kubectl get pods -n mern-todo -o wide
```

View Services:

```bash
kubectl get svc -n mern-todo
```

View Ingress:

```bash
kubectl get ingress -n mern-todo
```

View logs:

```bash
kubectl logs -l app=backend -n mern-todo
kubectl logs -l app=frontend -n mern-todo
```

Describe Pod:

```bash
kubectl describe pod <POD_NAME> -n mern-todo
```

Check HPA:

```bash
kubectl get hpa -n mern-todo
```

Check Docker Hub image:

```bash
docker manifest inspect tnat8628/mern-todo-backend:latest
docker manifest inspect tnat8628/mern-todo-frontend:latest
```

---

## 25. DevOps Concepts Practiced

This lab demonstrates:

* Containerization with Docker
* Multi-stage Docker build
* Kubernetes Deployment
* Kubernetes Service
* StatefulSet for database
* PersistentVolumeClaim for MongoDB data
* ConfigMap and Secret management
* Nginx Ingress path-based routing
* Readiness and liveness probes
* Resource requests and limits
* Horizontal scaling with replicas
* Horizontal Pod Autoscaler
* Self-healing behavior
* Rolling restart
* Docker Hub registry usage
* GitHub Actions CI pipeline
* GitHub Secrets for CI credentials

---

## 26. Current CI/CD Model

Current model:

```text
CI: Automated
CD: Manual
```

Flow:

```text
git push
  |
  v
GitHub Actions
  |
  +-- build Docker images
  +-- push images to Docker Hub
  |
  v
Manual deploy:
  kubectl rollout restart deployment/backend -n mern-todo
  kubectl rollout restart deployment/frontend -n mern-todo
```

Future improvements:

```text
- Automated deployment to Kubernetes
- Helm chart packaging
- GitOps with Argo CD
- HTTPS Ingress with TLS
- Prometheus and Grafana monitoring
- Loki or ELK logging
- Image security scanning
```

---

## 27. Security Notes

Do not commit real secrets.

This file is safe to commit:

```text
k8s/backend-secret.example.yaml
```

This file must not be committed:

```text
k8s/backend-secret.yaml
```

If a real password or token is accidentally committed, rotate it immediately.

