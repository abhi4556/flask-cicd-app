________________________________________
🚀 Flask CI/CD with Jenkins + Docker + GitOps (Argo CD + Minikube)
📌 Project Overview
This project implements a complete CI/CD pipeline with GitOps deployment using:
•	Jenkins (CI)
•	Docker
•	Docker Hub
•	Kubernetes (Minikube)
•	Argo CD (GitOps CD)
🔄 Workflow Architecture
Developer → GitHub (App Repo)
        → Jenkins CI
            → Build Docker Image
            → Push to Docker Hub
            → Update Kubernetes Manifest Repo
        → Argo CD (GitOps)
            → Detect Manifest Change
            → Deploy to Kubernetes (Minikube)
________________________________________
🏗️ Tech Stack
•	Jenkins (Docker containerized)
•	Docker Engine
•	Docker Hub
•	Minikube (Kubernetes cluster)
•	Argo CD (GitOps controller)
•	GitHub
________________________________________
🛠️ Step-by-Step Setup Guide
________________________________________
1️⃣ Install Docker
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
Verify:
docker --version
________________________________________
2️⃣ Run Jenkins in Docker (Correct Way)
Create custom Jenkins image with Docker CLI
Create Dockerfile.jenkins:
FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y docker.io && \
    apt-get clean

USER jenkins
Build image:
docker build -t my-jenkins -f Dockerfile.jenkins .
________________________________________
Get Docker group ID
getent group docker
Example output:
docker:x:1001:username
GID = 1001
________________________________________
Run Jenkins container
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --group-add 1001 \
  my-jenkins
Access Jenkins:
http://localhost:8080
________________________________________
3️⃣ Configure Jenkins
Install Plugins
•	Docker Pipeline
•	Git
•	Pipeline
________________________________________
Add Docker Hub Credentials
Manage Jenkins → Credentials → Add:
•	Type: Username/Password
•	ID: dockerhub-creds
•	Username: DockerHub username
•	Password: DockerHub password or token
________________________________________
4️⃣ Jenkins Pipeline (CI)
Jenkinsfile
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "abhimukh/flask-app"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        dockerImage.push("${BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                script {
                    sh """
                    git config --global user.email "jenkins@example.com"
                    git config --global user.name "Jenkins CI"

                    git clone https://github.com/yourusername/k8s-manifests.git
                    cd k8s-manifests

                    sed -i "s|image: abhimukh/flask-app:.*|image: abhimukh/flask-app:${BUILD_NUMBER}|" deployment.yaml

                    git add deployment.yaml
                    git commit -m "Update image tag to ${BUILD_NUMBER}"
                    git push origin main
                    """
                }
            }
        }
    }
}
________________________________________
5️⃣ Install Minikube (Kubernetes)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
Start cluster:
minikube start --nodes=2 --cpus=2 --memory=4096 --driver=docker
Verify:
kubectl get nodes
________________________________________
6️⃣ Install Argo CD (GitOps)
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
________________________________________
Access Argo CD
kubectl port-forward svc/argocd-server -n argocd 8081:443
Open:
http://localhost:8081
________________________________________
Get Admin Password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
Username:
admin
________________________________________
7️⃣ Create Kubernetes Manifest Repo
deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: abhimukh/flask-app:latest
        ports:
        - containerPort: 5000
service.yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  ports:
    - port: 5000
      targetPort: 5000
  type: NodePort
Push to GitHub.
________________________________________
8️⃣ Create Argo CD Application
In Argo CD UI:
•	App Name: flask-app
•	Repo URL: your manifest repo
•	Path: .
•	Cluster: In-cluster
•	Namespace: default
•	Sync Policy: Automatic
Click Create.
________________________________________
🎯 Final GitOps Flow
1.	Code pushed to GitHub
2.	Jenkins builds Docker image
3.	Jenkins pushes image to Docker Hub
4.	Jenkins updates deployment.yaml
5.	Git commit triggers Argo CD
6.	Argo CD auto-syncs
7.	Kubernetes deploys new version
________________________________________
🐞 Issues Faced & Resolutions
________________________________________
❌ Issue 1: fatal: not in a git directory
Cause:
Jenkins workspace corrupted due to permission mismatch (ran container as root earlier).
Fix:
Reset ownership of volume:
docker run --rm \
  -v jenkins_home:/var/jenkins_home \
  alpine \
  chown -R 1000:1000 /var/jenkins_home
Explanation:
Jenkins runs as UID 1000. If files are owned by root, Git cannot initialize repo.
________________________________________
❌ Issue 2: Failed to create temporary file
Cause:
Permission mismatch in /var/jenkins_home.
Fix:
Same ownership correction as above.
________________________________________
❌ Issue 3: docker: not found
Cause:
Docker CLI not installed inside Jenkins container.
Fix:
Build custom Jenkins image with docker installed.
________________________________________
❌ Issue 4: Cannot push Docker image
Cause:
No Docker authentication.
Fix:
Used Jenkins credentials + docker.withRegistry().
________________________________________
❌ Issue 5: Argo CD port 8080 already in use
Cause:
Jenkins using port 8080.
Fix:
kubectl port-forward svc/argocd-server -n argocd 8081:443
________________________________________
❌ Issue 6: Cannot create Argo CD application
Cause:
Missing required fields or invalid repo path.
Fix:
Ensure:
•	Repo URL correct
•	Path correct
•	Namespace exists
________________________________________
📈 What This Project Demonstrates
•	Containerized Jenkins
•	Secure Docker Hub authentication
•	Automated CI pipeline
•	GitOps workflow
•	Kubernetes deployment
•	Argo CD continuous deployment
•	Real-world debugging & troubleshooting
________________________________________
🚀 Future Improvements
•	Helm charts instead of raw YAML
•	Add SonarQube stage
•	Add Kubernetes namespace isolation
•	Add Ingress + domain
•	Add SSL (cert-manager)
•	Add Monitoring (Prometheus + Grafana)
________________________________________
🏆 Conclusion
This project demonstrates a complete modern DevOps pipeline implementing:
•	CI with Jenkins
•	Docker image management
•	GitOps-based CD with Argo CD
•	Kubernetes deployment via Minikube
It reflects real-world DevOps architecture and troubleshooting experience.

