# MERN Todo Kubernetes DevOps Lab

This project is a DevOps/Kubernetes lab based on a MERN Todo application.

The original application contains:

* React frontend
* Node.js/Express backend
* MongoDB database

This lab extends the original project with Docker, Kubernetes, Ingress, persistent storage, CI/CD automation, DevSecOps scanning, and GitHub Actions self-hosted runner deployment.

---

## 1. Project Goals

The goal of this lab is to practice a real DevOps workflow for deploying a MERN application on Kubernetes.

This lab covers:

* Containerizing frontend and backend applications with Docker
* Deploying MongoDB using StatefulSet and PersistentVolumeClaim
* Deploying backend and frontend using Kubernetes Deployments and Services
* Routing traffic using Nginx Ingress Controller
* Managing runtime configuration using ConfigMap and Secret
* Adding readiness and liveness probes
* Adding resource requests and limits
* Scaling backend replicas
* Adding Horizontal Pod Autoscaler
* Using Docker Hub as a container registry
* Building and pushing images with GitHub Actions
* Deploying to Minikube through a GitHub Actions self-hosted runner
* Using immutable image tags based on Git commit SHA
* Automatically rolling back failed Kubernetes rollouts
* Scanning Docker images with Trivy before pushing and deploying

---

## 2. Architecture Overview

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

## 3. Tech Stack

### Application

* Frontend: React
* Backend: Node.js, Express
* Database: MongoDB

### DevOps / Infrastructure

* Docker
* Docker Hub
* Minikube
* Kubernetes
* Nginx Ingress Controller
* GitHub Actions
* GitHub Actions self-hosted runner
* Trivy
* Kubernetes ConfigMap
* Kubernetes Secret
* Kubernetes Deployment
* Kubernetes Service
* Kubernetes StatefulSet
* PersistentVolumeClaim
* Horizontal Pod Autoscaler

---

## 4. Project Structure

```text
mern-todo-app/
  backend/
    Dockerfile
    .dockerignore
    server.js
    package.json
    package-lock.json

  frontend/
    Dockerfile
    .dockerignore
    src/
    package.json
    package-lock.json

  k8s/
    mongo-service.yaml
    mongo-statefulset.yaml
    backend-configmap.yaml
    backend-secret.example.yaml
    backend-deployment.yaml
    backend-service.yaml
    backend-hpa.yaml
    frontend-deployment.yaml
    frontend-service.yaml
    ingress.yaml

  .github/
    workflows/
      backend-docker-build.yml
      frontend-docker-build.yml
      test-self-hosted-runner.yml

  README.md
```

---

## 5. Docker Images

The application images are pushed to Docker Hub.

Backend image:

```text
tnat8628/mern-todo-backend:latest
tnat8628/mern-todo-backend:<commit-sha>
```

Frontend image:

```text
tnat8628/mern-todo-frontend:latest
tnat8628/mern-todo-frontend:<commit-sha>
```

The pipeline still pushes the `latest` tag for convenience, but Kubernetes deployments are updated using the immutable Git commit SHA tag.

Example:

```text
tnat8628/mern-todo-backend:62befaaf3ce6a0abae20d8b5e259faf4579db0d9
tnat8628/mern-todo-frontend:8289ee4f1610dd77e69f0e493ff0f50e9927c26d
```

Using immutable tags makes deployments easier to audit, debug, and roll back.

---

## 6. Kubernetes Resources

### MongoDB

MongoDB is deployed with a StatefulSet because it is a stateful component and needs stable storage.

Files:

```text
k8s/mongo-service.yaml
k8s/mongo-statefulset.yaml
```

Resources:

* `mongo` StatefulSet
* `mongo-service`
* `mongo-data-mongo-0` PVC

MongoDB connection string used by the backend:

```text
mongodb://mongo-service:27017/mern-todo
```

---

### Backend

The backend is deployed as a Kubernetes Deployment because it is stateless.

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
* Horizontal Pod Autoscaler support
* Environment variables from ConfigMap and Secret
* Automated deployment through GitHub Actions self-hosted runner
* Image scanning with Trivy before push and deploy
* Deployment using immutable commit SHA image tag
* Automatic rollback when rollout fails

---

### Frontend

The frontend is built as a static React application and served by Nginx.

Files:

```text
k8s/frontend-deployment.yaml
k8s/frontend-service.yaml
```

Frontend features:

* Multi-stage Docker build
* Static file serving with Nginx
* Readiness probe
* Liveness probe
* Resource requests and limits
* Automated deployment through GitHub Actions self-hosted runner
* Image scanning with Trivy before push and deploy
* Deployment using immutable commit SHA image tag
* Automatic rollback when rollout fails

---

### Ingress

Ingress is used to route HTTP traffic by path.

File:

```text
k8s/ingress.yaml
```

Routes:

```text
/      -> frontend-service:80
/api   -> backend-service:8000
```

Internal lab domain:

```text
mern-todo.local
```

---

## 7. Prerequisites

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

## 8. Start Minikube

Start Minikube with Docker driver:

```bash
minikube start --driver=docker
```

Check cluster status:

```bash
minikube status
kubectl get nodes -o wide
```

Expected result:

```text
minikube   Ready
```

---

## 9. Create Namespace

Create a namespace for the application:

```bash
kubectl create namespace mern-todo
```

Check namespaces:

```bash
kubectl get ns
```

---

## 10. Deploy MongoDB

Apply MongoDB Service and StatefulSet:

```bash
kubectl apply -f k8s/mongo-service.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
```

Check resources:

```bash
kubectl get pods -n mern-todo
kubectl get pvc -n mern-todo
kubectl get svc -n mern-todo
```

Expected Pod:

```text
mongo-0   1/1   Running
```

Test MongoDB:

```bash
kubectl exec -it mongo-0 -n mern-todo -- mongosh
```

Inside MongoDB shell:

```javascript
show dbs
exit
```

---

## 11. Configure Backend Environment

The backend uses these environment variables:

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

Do not commit real passwords, tokens, or application secrets.

---

## 12. Deploy Backend Manually

Apply backend resources:

```bash
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
kubectl apply -f k8s/backend-hpa.yaml
```

Check backend Deployment, Pods, Service, and HPA:

```bash
kubectl get deployment backend -n mern-todo
kubectl get pods -n mern-todo -l app=backend -o wide
kubectl get svc -n mern-todo
kubectl get hpa -n mern-todo
```

Check backend logs:

```bash
kubectl logs -l app=backend -n mern-todo
```

Expected logs:

```text
Listening on localhost:8000
DB Connected
```

---

## 13. Deploy Frontend Manually

Apply frontend resources:

```bash
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml
```

Check frontend resources:

```bash
kubectl get deployment frontend -n mern-todo
kubectl get pods -n mern-todo -l app=frontend -o wide
kubectl get svc -n mern-todo
```

---

## 14. Enable Minikube Ingress

Enable the Minikube Ingress addon:

```bash
minikube addons enable ingress
```

Check Ingress Controller:

```bash
kubectl get pods -n ingress-nginx
```

Expected result:

```text
ingress-nginx-controller   1/1   Running
```

---

## 15. Deploy Ingress

Apply the Ingress resource:

```bash
kubectl apply -f k8s/ingress.yaml
```

Check Ingress:

```bash
kubectl get ingress -n mern-todo
```

Expected result:

```text
mern-todo-ingress   nginx   mern-todo.local   192.168.49.2   80
```

---

## 16. Configure Local DNS

Get Minikube IP:

```bash
minikube ip
```

Edit `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Add this line:

```text
<MINIKUBE_IP> mern-todo.local
```

Example:

```text
192.168.49.2 mern-todo.local
```

Test DNS resolution:

```bash
ping -c 2 mern-todo.local
```

---

## 17. Test Application

Test frontend through Ingress:

```bash
curl -I http://mern-todo.local
```

Expected response:

```text
HTTP/1.1 200 OK
```

Test backend route through Ingress:

```bash
curl -i http://mern-todo.local/api/task/getTask
```

Expected response:

```text
HTTP/1.1 401 Unauthorized
{"message":"Unauthorized"}
```

This is expected because the route requires JWT authentication.

---

## 18. Health Checks

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

Expected response:

```json
{"status":"ok","service":"backend"}
```

If a version field is added during testing, the response may look like this:

```json
{"status":"ok","service":"backend","version":"ci-backend-test"}
```

---

## 19. Self-Healing Test

Delete one backend Pod:

```bash
kubectl get pods -n mern-todo -l app=backend
kubectl delete pod <BACKEND_POD_NAME> -n mern-todo
```

Watch Kubernetes recreate it:

```bash
kubectl get pods -n mern-todo -l app=backend -w
```

Kubernetes automatically creates a new Pod because the Deployment declares the desired number of replicas.

---

## 20. Scaling

Backend uses 2 replicas:

```yaml
replicas: 2
```

Check backend Deployment:

```bash
kubectl get deployment backend -n mern-todo
kubectl get pods -n mern-todo -l app=backend -o wide
```

Check Service endpoints:

```bash
kubectl get endpoints backend-service -n mern-todo
```

Expected result:

```text
backend-service   10.244.x.x:8000,10.244.x.y:8000
```

---

## 21. Horizontal Pod Autoscaler

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

Expected result:

```text
backend-hpa   Deployment/backend   cpu: x%/50%   2   5   2
```

---

## 22. GitHub Actions CI/CD

This project uses separate GitHub Actions workflows for backend and frontend CI/CD.

Workflow files:

```text
.github/workflows/backend-docker-build.yml
.github/workflows/frontend-docker-build.yml
.github/workflows/test-self-hosted-runner.yml
```

---

### Backend Workflow

The backend workflow runs when these paths change:

```text
backend/**
.github/workflows/backend-docker-build.yml
```

Backend workflow tasks:

```text
1. Checkout source code
2. Set up Docker Buildx
3. Login to Docker Hub
4. Build backend image locally
5. Scan backend image with Trivy
6. Push backend image to Docker Hub if the scan passes
7. Deploy backend to Minikube using immutable commit SHA image tag
8. Roll back automatically if Kubernetes rollout fails
```

Images pushed:

```text
tnat8628/mern-todo-backend:latest
tnat8628/mern-todo-backend:<commit-sha>
```

Deployment command executed by the self-hosted runner:

```bash
kubectl set image deployment/backend backend=tnat8628/mern-todo-backend:<commit-sha> -n mern-todo
```

Rollout check:

```bash
kubectl rollout status deployment/backend -n mern-todo --timeout=180s
```

Rollback command if rollout fails:

```bash
kubectl rollout undo deployment/backend -n mern-todo
```

---

### Frontend Workflow

The frontend workflow runs when these paths change:

```text
frontend/**
.github/workflows/frontend-docker-build.yml
```

Frontend workflow tasks:

```text
1. Checkout source code
2. Set up Docker Buildx
3. Login to Docker Hub
4. Build frontend image locally
5. Scan frontend image with Trivy
6. Push frontend image to Docker Hub if the scan passes
7. Deploy frontend to Minikube using immutable commit SHA image tag
8. Roll back automatically if Kubernetes rollout fails
```

Images pushed:

```text
tnat8628/mern-todo-frontend:latest
tnat8628/mern-todo-frontend:<commit-sha>
```

Deployment command executed by the self-hosted runner:

```bash
kubectl set image deployment/frontend frontend=tnat8628/mern-todo-frontend:<commit-sha> -n mern-todo
```

Rollout check:

```bash
kubectl rollout status deployment/frontend -n mern-todo --timeout=180s
```

Rollback command if rollout fails:

```bash
kubectl rollout undo deployment/frontend -n mern-todo
```

---

## 23. DevSecOps Enhancements

This lab includes several DevSecOps improvements to make the CI/CD pipeline safer and more production-like.

### Immutable Image Tags

The pipeline does not deploy images using only the `latest` tag.

Each backend and frontend image is tagged with the Git commit SHA.

Example:

```text
tnat8628/mern-todo-backend:<commit-sha>
tnat8628/mern-todo-frontend:<commit-sha>
```

Kubernetes Deployments are updated with the commit SHA image tag using `kubectl set image`.

This makes deployments easier to audit, debug, and roll back.

---

### Automatic Rollback

The deployment jobs wait for Kubernetes rollout status after updating the image.

If the rollout fails or times out, the workflow automatically runs:

```bash
kubectl rollout undo deployment/backend -n mern-todo
kubectl rollout undo deployment/frontend -n mern-todo
```

The workflow still exits with failure after rollback, so the team knows that the new release was unsuccessful.

---

### Trivy Image Scanning

Both backend and frontend images are scanned with Trivy before being pushed to Docker Hub.

Current scan policy:

```text
Severity: CRITICAL
Ignore unfixed vulnerabilities: true
Vulnerability types: OS packages and application libraries
```

Pipeline behavior:

```text
Build image locally
  |
  v
Scan image with Trivy
  |
  +-- If CRITICAL vulnerabilities are found:
  |     fail pipeline
  |     do not push image
  |     do not deploy
  |
  +-- If scan passes:
        push image to Docker Hub
        deploy to Minikube
```

During testing:

* Trivy detected critical vulnerabilities in `mongoose`.
* The backend issue was fixed by updating `mongoose`.
* Trivy detected critical vulnerabilities in Alpine/OpenSSL packages in the frontend runtime image.
* The frontend issue was fixed by updating Alpine runtime packages in the Nginx stage.

---

## 24. Required GitHub Secrets

The workflows require these repository secrets:

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

These secrets are used by GitHub Actions to log in to Docker Hub and push Docker images.

---

## 25. Self-Hosted Runner

A GitHub Actions self-hosted runner is installed on the Ubuntu VM that also runs Minikube.

The runner allows GitHub Actions deployment jobs to run `kubectl` commands directly against the local Minikube cluster.

Runner purpose:

```text
GitHub Actions
  |
  v
Self-hosted runner on Ubuntu VM
  |
  v
kubectl set image / rollout status / rollout undo
  |
  v
Minikube cluster
```

Check runner service status:

```bash
cd ~/actions-runner
sudo ./svc.sh status
```

Expected status:

```text
active (running)
Listening for Jobs
```

---

## 26. Deploy Images with Immutable Tags

The Kubernetes Deployments may initially define:

```text
tnat8628/mern-todo-backend:latest
tnat8628/mern-todo-frontend:latest
```

However, during CI/CD deployment, GitHub Actions updates the running Kubernetes Deployments with immutable commit SHA image tags.

Normal backend or frontend changes are deployed automatically by GitHub Actions through the self-hosted runner.

Manual rollout can still be used for debugging:

```bash
kubectl rollout restart deployment/backend -n mern-todo
kubectl rollout restart deployment/frontend -n mern-todo
```

Check rollout manually:

```bash
kubectl rollout status deployment/backend -n mern-todo
kubectl rollout status deployment/frontend -n mern-todo
```

Test application:

```bash
curl -I http://mern-todo.local
curl -i http://mern-todo.local/api/task/getTask
```

---

## 27. Useful Commands

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

View backend logs:

```bash
kubectl logs -l app=backend -n mern-todo
```

View frontend logs:

```bash
kubectl logs -l app=frontend -n mern-todo
```

Describe a Pod:

```bash
kubectl describe pod <POD_NAME> -n mern-todo
```

Check HPA:

```bash
kubectl get hpa -n mern-todo
```

Check current backend image:

```bash
kubectl get deployment backend -n mern-todo -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

Check current frontend image:

```bash
kubectl get deployment frontend -n mern-todo -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

Check Docker Hub images:

```bash
docker manifest inspect tnat8628/mern-todo-backend:latest
docker manifest inspect tnat8628/mern-todo-frontend:latest
```

Check rollout history:

```bash
kubectl rollout history deployment/backend -n mern-todo
kubectl rollout history deployment/frontend -n mern-todo
```

Rollback manually:

```bash
kubectl rollout undo deployment/backend -n mern-todo
kubectl rollout undo deployment/frontend -n mern-todo
```

---

## 28. DevOps Concepts Practiced

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
* Kubernetes self-healing behavior
* Rolling restart
* Immutable image tags
* Automated rollback
* Docker Hub registry usage
* GitHub Actions CI pipeline
* GitHub Actions CD pipeline
* GitHub Actions self-hosted runner
* GitHub Secrets for CI credentials
* Trivy image scanning
* Automated deployment to Minikube

---

## 29. Current CI/CD Model

Current model:

```text
CI: Automated
CD: Automated through a GitHub Actions self-hosted runner
DevSecOps: Trivy image scanning before push and deploy
```

Backend flow:

```text
Change in backend/
  |
  v
GitHub Actions: Build and Push Backend Image
  |
  +-- Build backend Docker image locally
  +-- Scan backend image with Trivy
  +-- Push image to Docker Hub if scan passes
  |
  v
GitHub Actions self-hosted runner
  |
  +-- kubectl set image deployment/backend backend=tnat8628/mern-todo-backend:<commit-sha>
  +-- kubectl rollout status deployment/backend -n mern-todo
  +-- kubectl rollout undo deployment/backend -n mern-todo if rollout fails
```

Frontend flow:

```text
Change in frontend/
  |
  v
GitHub Actions: Build and Push Frontend Image
  |
  +-- Build frontend Docker image locally
  +-- Scan frontend image with Trivy
  +-- Push image to Docker Hub if scan passes
  |
  v
GitHub Actions self-hosted runner
  |
  +-- kubectl set image deployment/frontend frontend=tnat8628/mern-todo-frontend:<commit-sha>
  +-- kubectl rollout status deployment/frontend -n mern-todo
  +-- kubectl rollout undo deployment/frontend -n mern-todo if rollout fails
```

The self-hosted runner is installed on the Ubuntu VM that also runs Minikube. This allows the deployment jobs to access the local Kubernetes cluster using `kubectl`.

---

## 30. Security Notes

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

GitHub Actions secrets should be stored in GitHub repository secrets, not in source code.

Docker Hub credentials should be stored in:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

---

## 31. Future Improvements

Possible improvements:

* Package Kubernetes manifests with Helm
* Use GitOps with Argo CD
* Add HTTPS Ingress with TLS
* Add Prometheus and Grafana monitoring
* Add Loki or ELK logging
* Add image signing and verification
* Add SBOM generation
* Add automated rollback notifications
* Split development, staging, and production environments