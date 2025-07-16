# MERN Stack Kubernetes Deployment with Helm and Jenkins

This repository contains a complete Kubernetes deployment setup for a MERN (MongoDB, Express.js, React, Node.js) application using:
- **Kubernetes** for container orchestration
- **Helm** for templated, reusable Kubernetes manifests
- **Docker** for containerizing the applications
- **Jenkins** (optional) for CI/CD automation
---

## Directory Structure
kube-assignments/<br>
├── backend_kube/         # Kubernetes manifests for the backend<br>
│   └── ...               # (From https://github.com/tanujbhatia24/backend_kube.git)<br>
├── frontend_kube/        # Kubernetes manifests for the frontend<br>
│   └── ...               # (From https://github.com/tanujbhatia24/frontend_kube.git)<br>
├── mern-chart/           # Helm chart for the MERN stack<br>
│   ├── templates/<br>
│   │   ├── backend.yaml<br>
│   │   ├── frontend.yaml<br>
│   │   └── mongo.yaml<br>
│   ├── Chart.yaml<br>
│   └── values.yaml<br>
├── jenkinsfile           # CI/CD pipeline using Jenkins<br>
└── README.md             # You're here!<br>

- You need to clone backend & frontend repos inside project directory if you want it to work locally.
---

## Docker Image Build & Push

### Frontend

```bash
go to frontend_kube dir
docker build -t tanujbhatia24/frontend:latest .
docker push tanujbhatia24/frontend:latest
```
### Backend
```bash
go to backend_kube dir
docker build -t tanujbhatia24/backend:latest .
docker push tanujbhatia24/backend:latest
```
### Deploy with Helm
```bash
got mern-chart dir
helm upgrade --install mern-app . --namespace mern --create-namespace
```
### Verify Deployment
```bash
kubectl get all -n mern
```
- Make sure all pods are 1/1 READY and services are running (frontend-service, backend-service, mongo).  
---

## Access the App 
### Test via Port Forwarding
```bash
kubectl port-forward svc/frontend-service 8080:80 -n mern
kubectl port-forward svc/backend-service 3000:3000 -n mern
kubectl port-forward svc/mongo 28017:27017 -n mern
```
Frontend App available at: http://localhost:8080<br>
Backend App available at: http://localhost:3000<br>
Mongo DB available at: mongodb://localhost:28017<br>

### Test via Minikube (OPTIONAL)
```bash
minikube service frontend-service -n mern
```
- This will open your browser or give you a URL — check the site.
---

## Configuration
### .env file in Backend
```bash
ATLAS_URI=mongodb://mongo:27017/blog_mern
```
### .env file in Frontend
Based on your setup you need to update.
```bash
REACT_APP_API_BASE_URL=http://backend-service:3000
OR
REACT_APP_API_BASE_URL=http://localhost:3000
```
---

## Jenkins configuration
### Pipeline setup
1. Create Jenkinsfile inside your project directory.
2. Create dockerhub credentials in jenkins.
3. Create jenkins pipeline.
### Validation commands
Compare deploy version of helm
```bash
helm history chart_name -n namespace_name
```
EXAMPLE 
```bash
helm history mern-chart -n mern
```
---


## Validation Snapshots
- Docker Images validation snanshot<br>
![image](https://github.com/user-attachments/assets/d7f5e3b1-aec3-4596-91a1-8942e1104090)<br>

- Kubectl pod & service validation snapshot<br>
![image](https://github.com/user-attachments/assets/4ca580b9-5af7-41c5-bb53-1f1716fd75e5)<br>

- Frontend url validation snapshot<br>
Port forwarding<br>
![image](https://github.com/user-attachments/assets/ae1dc45c-b973-4d02-ae69-ca070b67a5c4)<br>
![image](https://github.com/user-attachments/assets/427ecbd7-764e-4b37-9ebd-7f48cdefea41)<br>
Minikube<br>
![image](https://github.com/user-attachments/assets/23ede8ee-b997-403a-85f0-b7e2c46f90e0)<br>
![image](https://github.com/user-attachments/assets/0c89fb1b-1c47-4eeb-b65d-b8aa4a37adc1)<br>

- Backend url validation snapshot<br>
![image](https://github.com/user-attachments/assets/2cb7d179-68c7-4236-838c-f4b0c77fda12)<br>
![image](https://github.com/user-attachments/assets/feea7608-bf82-4880-a9c6-1ec16218b0e8)<br>

- Mongo DB validation snapshot<br>
![image](https://github.com/user-attachments/assets/d1826381-a98f-4425-a899-62e492cdbc61)<br>
![image](https://github.com/user-attachments/assets/7f8d9cd8-0581-4d4b-81e7-62bf65daffc1)<br>

- Pods count before jenkins pipeline trigger snapshot<br>
![image](https://github.com/user-attachments/assets/dda1f1b9-3343-42ac-9958-392a0f5b147c)<br>

- Pods Count after jenkins pipeline trigger snapshot (replicas set to 2)<br>
![image](https://github.com/user-attachments/assets/7f11ce62-5e55-4da6-9581-9382b2f6e254)<br>

- Jenkins validation snapshot<br>
![image](https://github.com/user-attachments/assets/48aa3ed1-9e76-4298-be63-d1f8eed72d56)<br>
![image](https://github.com/user-attachments/assets/f03795ee-8278-4255-9059-c689cdca9e24)<br>
---

## Author<br>
Tanuj Bhatia<br>
DockerHub: tanujbhatia24<br>
NOTE - This application doesn't have any footprints left on AWS.
