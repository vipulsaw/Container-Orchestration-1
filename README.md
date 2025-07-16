# Kubernetes Deployment with Helm and Jenkins [MERN Stack]

This repository contains a complete Kubernetes deployment setup for a MERN (MongoDB, Express.js, React, Node.js) application using:
- **Kubernetes** for container orchestration
- **Helm** for templated, reusable Kubernetes manifests
- **Docker** for containerizing the applications
- **Jenkins** (optional) for CI/CD automation
---

## Directory Structure
kube-assignments/<br>
├── backend_kube/         # Kubernetes manifests for the backend<br>
│   
├── frontend_kube/        # Kubernetes manifests for the frontend<br>
│   
├── mern-chart/           # Helm chart for the MERN stack<br>
│   ├── templates/<br>
│   │   ├── backend.yaml<br>
│   │   ├── frontend.yaml<br>
│   │   └── mongo.yaml<br>
│   ├── Chart.yaml<br>
│   └── values.yaml<br>
├── jenkinsfile           
└── README.md             

- You need to clone backend & frontend repos inside project directory if you want it to work locally.
---

## Docker Image Build & Push


### Frontend

```bash
go to frontend_kube dir
docker build -t vipulsaw123/frontend:latest .
docker push vipulsaw123/frontend:latest
```
<img width="944" height="441" alt="image" src="https://github.com/user-attachments/assets/ef1191ce-0114-461f-a9bf-e604704851ef" />


### Backend
```bash
go to backend_kube dir
docker build -t vipulsaw123/backend:latest .
docker push vipulsaw123/backend:latest
```
<img width="944" height="434" alt="image" src="https://github.com/user-attachments/assets/ea646965-0d78-47ac-9080-3c044fe3ef77" />
<img width="941" height="177" alt="image" src="https://github.com/user-attachments/assets/ecb46d95-6905-4f39-8f0c-846770bbfe25" />


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
#### Minikube [local]

<img width="939" height="238" alt="image" src="https://github.com/user-attachments/assets/7a4b26e1-69f7-47ad-a970-b5f6e741252d" />

#### Minikube [EC-2/Ubuntu]

<img width="939" height="254" alt="image" src="https://github.com/user-attachments/assets/6e02e259-0119-43c0-88d2-e2430cba0f1b" />


## Access the App 
### Test via Port Forwarding [Local]
```bash
kubectl port-forward svc/frontend-service 8080:80 -n mern
kubectl port-forward svc/backend-service 3000:3000 -n mern
kubectl port-forward svc/mongo 28017:27017 -n mern
```
Frontend App available at: http://localhost:8080<br>

<img width="955" height="472" alt="image" src="https://github.com/user-attachments/assets/e76ba76f-db73-4c74-b985-d398d8d0da07" />


Backend App available at: http://localhost:3000<br>

<img width="953" height="169" alt="image" src="https://github.com/user-attachments/assets/ebb491d0-8f66-4abb-b1c5-49b3d1f6453d" />


Mongo DB available at: mongodb://localhost:28017<br>

### Test via Port Forwarding [Ec-2/Ubuntu]
```bash
kubectl port-forward svc/frontend-service -n mern 8080:80 --address=0.0.0.0
kubectl port-forward svc/backend-service -n mern 3000:3000 --address=0.0.0.0
kubectl port-forward svc/mongo 28017:27017 -n mern
```
Frontend App available at: http://public-ip:8080<br>

<img width="959" height="467" alt="image" src="https://github.com/user-attachments/assets/c0041147-ddd2-42e7-90c4-2b68b40ab7b9" />

Backend App available at: http://public-ip:3000<br>

<img width="959" height="176" alt="image" src="https://github.com/user-attachments/assets/35b885da-00ac-4da7-b25d-54b27c3e5a43" />

Mongo DB available at: mongodb://public-ip:28017<br>

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

```
pipeline {
    agent any
    
    environment {
        REPO_URL = 'https://github.com/vipulsaw/Container-Orchestration-1.git'
        REPO_NAME = 'Container-Orchestration-1'
        HELM_CHART_DIR = 'mern-chart'
        NAMESPACE = 'mern'
        RELEASE_NAME = 'mern-app'
        EC2_INSTANCE_IP = '54.145.162.64'
    }
    
    stages {
        stage('Checkout Source Code') {
            steps {
                checkout([$class: 'GitSCM', 
                         branches: [[name: '*/main']], 
                         userRemoteConfigs: [[url: env.REPO_URL]]])
                echo "Repository cloned successfully"
            }
        }
        
        stage('Copy Repository to Target EC2') {
            steps {
                sshagent(credentials: ['vipul']) {
                    script {
                        // Create directory on remote if it doesn't exist
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@${env.EC2_INSTANCE_IP} \
                            'mkdir -p ~/${env.REPO_NAME}'
                        """
                        
                        // Copy files using rsync
                        sh """
                            rsync -avz -e "ssh -o StrictHostKeyChecking=no" \
                            --exclude='.git' \
                            --delete \
                            ./ ubuntu@${env.EC2_INSTANCE_IP}:~/${env.REPO_NAME}/
                        """
                        
                        echo "Files copied to EC2 instance successfully"
                    }
                }
            }
        }
        
        stage('Install/Upgrade Helm Release') {
            steps {
                sshagent(credentials: ['vipul']) {
                    script {
                        // Run helm upgrade
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@${env.EC2_INSTANCE_IP} \
                            'cd ~/${env.REPO_NAME}/${env.HELM_CHART_DIR} && \
                            helm upgrade --install ${env.RELEASE_NAME} . \
                            --namespace ${env.NAMESPACE} \
                            --create-namespace'
                        """
                        
                        echo "Helm release upgraded successfully"
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['vipul']) {
                    script {
                        sh """
                            ssh -o StrictHostKeyChecking=no ubuntu@${env.EC2_INSTANCE_IP} \
                            'helm list -n ${env.NAMESPACE} && \
                            kubectl get pods -n ${env.NAMESPACE}'
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}

```

<img width="959" height="440" alt="image" src="https://github.com/user-attachments/assets/c82945fa-2d64-45ce-8548-69a006d47d03" />


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

