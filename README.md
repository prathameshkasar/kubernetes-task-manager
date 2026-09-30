# Kubernetes Task Manager

A three-tier Task Management application deployed on Kubernetes using Minikube.

## Architecture

```text
                    Browser
                       |
                       v
              Frontend NodePort
                 Port 30001
                       |
                       v
             Frontend Deployment
                Nginx :8088
                       |
                /api requests
                       |
                       v
              Backend Service
                 ClusterIP
                  :8080
                       |
                       v
             Backend Deployment
             Spring Boot :8080
                       |
                       v
               MySQL Service
                 ClusterIP
                  :3306
                       |
                       v
              MySQL Deployment
                 MySQL 8
                       |
                       v
               MySQL PVC
                       |
                       v
             Persistent Storage
```

## Project Overview

This project demonstrates how to deploy a complete three-tier application on Kubernetes.

The application contains:

* Frontend using HTML and Nginx
* Backend using Java and Spring Boot
* MySQL database
* Docker containers
* Kubernetes Deployments and Services
* ConfigMap and Secret
* PersistentVolumeClaim for MySQL storage
* Kubernetes Job for database initialization
* Kubernetes CronJob for database backups
* Minikube for the Kubernetes environment

## Technologies Used

* Java 17
* Spring Boot
* Maven
* Docker
* Kubernetes
* Minikube
* MySQL 8
* Nginx
* Git
* GitHub

## Kubernetes Concepts Used

* Pods
* Labels
* Selectors
* ReplicaSets
* Deployments
* ClusterIP
* NodePort
* ConfigMaps
* Secrets
* PersistentVolume
* PersistentVolumeClaim
* StorageClass
* Readiness Probes
* Liveness Probes
* Namespaces
* Jobs
* CronJobs
* Resource Requests
* Resource Limits
* Kubernetes DNS
* Rolling Updates

## Project Structure

```text
kubernetes-task-manager/
│
├── backend/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
│       └── main/
│           ├── java/
│           │   └── com/
│           │       └── example/
│           │           └── taskmanager/
│           │               ├── TaskManagerApplication.java
│           │               ├── Task.java
│           │               ├── TaskRepository.java
│           │               └── TaskController.java
│           │
│           └── resources/
│               └── application.properties
│
├── frontend/
│   ├── Dockerfile
│   ├── index.html
│   └── nginx.conf
│
├── k8s/
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── mysql-pvc.yaml
│   ├── mysql-deployment.yaml
│   ├── mysql-service.yaml
│   ├── db-init-job.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backup-pvc.yaml
│   └── backup-cronjob.yaml
│
├── .gitignore
└── README.md
```

## Application Components

### Frontend

The frontend is served using Nginx on port `8088`.

The frontend Service uses Kubernetes NodePort `30001` to make the application accessible outside the cluster.

### Backend

The backend is a Spring Boot REST API running on port `8080`.

Two backend replicas are created using a Kubernetes Deployment.

### Database

The application uses MySQL 8.

The MySQL database is called:

```text
taskdb
```

The task data is stored in the `tasks` table.

### Storage

MySQL uses a PersistentVolumeClaim:

```text
mysql-pvc
```

The PVC is mounted inside the MySQL container at:

```text
/var/lib/mysql
```

This allows database data to remain in persistent storage instead of depending only on the MySQL Pod.

### ConfigMap

The ConfigMap stores non-sensitive configuration such as:

```text
DB_HOST
DB_PORT
DB_NAME
DB_USERNAME
```

### Secret

The Kubernetes Secret stores database passwords.

The credentials in this project are dummy credentials intended only for local Minikube practice.

Do not use these credentials in a production environment.

### Database Initialization

A Kubernetes Job initializes the database and creates the required `tasks` table.

It also inserts initial sample tasks.

### Database Backup

A Kubernetes CronJob periodically creates MySQL database backups using `mysqldump`.

The backup files are stored in a separate PersistentVolumeClaim.

## Deployment

### 1. Start Minikube

```bash
minikube start
```

Check:

```bash
minikube status
```

### 2. Use Minikube Docker Environment

```bash
eval $(minikube docker-env)
```

This allows Docker images to be built directly inside the Minikube Docker environment.

### 3. Build Backend Image

```bash
docker build -t task-backend:v1 ./backend
```

### 4. Build Frontend Image

```bash
docker build -t task-frontend:v1 ./frontend
```

Check images:

```bash
docker images
```

## Deploy Kubernetes Resources

### Create Namespace

```bash
kubectl apply -f k8s/namespace.yaml
```

### Create ConfigMap and Secret

```bash
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
```

### Create MySQL Storage

```bash
kubectl apply -f k8s/mysql-pvc.yaml
```

### Deploy MySQL

```bash
kubectl apply -f k8s/mysql-deployment.yaml
kubectl apply -f k8s/mysql-service.yaml
```

### Initialize Database

```bash
kubectl apply -f k8s/db-init-job.yaml
```

### Deploy Backend

```bash
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
```

### Deploy Frontend

```bash
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml
```

### Create Backup Storage

```bash
kubectl apply -f k8s/backup-pvc.yaml
```

### Create Backup CronJob

```bash
kubectl apply -f k8s/backup-cronjob.yaml
```

## Verify Deployment

Check Pods:

```bash
kubectl get pods -n task-app
```

Check Deployments:

```bash
kubectl get deployments -n task-app
```

Check Services:

```bash
kubectl get services -n task-app
```

Check PVCs:

```bash
kubectl get pvc -n task-app
```

Check Persistent Volumes:

```bash
kubectl get pv
```

Check Jobs:

```bash
kubectl get jobs -n task-app
```

Check CronJobs:

```bash
kubectl get cronjobs -n task-app
```

Check all resources:

```bash
kubectl get all -n task-app
```

## Access the Application

The frontend Service uses NodePort:

```text
30001
```

Allow TCP port `30001` in the EC2 Security Group.

Then access the application using:

```text
http://YOUR_EC2_PUBLIC_IP:30001
```

Depending on the Minikube-on-EC2 networking setup, you may need:

```bash
kubectl port-forward --address=0.0.0.0 \
  service/frontend \
  30001:8088 \
  -n task-app
```

Then open:

```text
http://YOUR_EC2_PUBLIC_IP:30001
```

## Application Request Flow

```text
Browser
   |
   v
EC2 Public IP :30001
   |
   v
Frontend NodePort Service
   |
   v
Frontend Pod
   |
   | /api
   v
Backend Service :8080
   |
   v
Backend Pod
   |
   v
MySQL Service :3306
   |
   v
MySQL Pod
   |
   v
taskdb
   |
   v
tasks table
```

## Database Storage Flow

```text
MySQL Pod
    |
    | /var/lib/mysql
    v
mysql-pvc
    |
    v
PersistentVolume
    |
    v
Minikube Persistent Storage
```

## Backup Flow

```text
CronJob
   |
   v
mysqldump
   |
   v
taskdb backup
   |
   v
backup-pvc
   |
   v
Persistent Storage
```

## Useful Commands

Check Pods:

```bash
kubectl get pods -n task-app
```

Describe a Pod:

```bash
kubectl describe pod <pod-name> -n task-app
```

Check Services:

```bash
kubectl get svc -n task-app
```

Check Endpoints:

```bash
kubectl get endpoints -n task-app
```

Check PVC:

```bash
kubectl get pvc -n task-app
```

Check Logs:

```bash
kubectl logs <pod-name> -n task-app
```

Enter MySQL Pod:

```bash
kubectl exec -it <mysql-pod-name> -n task-app -- bash
```

## Important Note

The Kubernetes Secret in this learning project contains dummy credentials for local practice.

Never commit real production passwords, API keys, tokens, certificates, or other sensitive credentials to GitHub.

For production environments, use a proper secret-management solution.

## Learning Outcomes

This project provides practical experience with:

* Docker containerization
* Kubernetes Deployments
* Kubernetes Services
* NodePort and ClusterIP
* ConfigMaps
* Secrets
* PersistentVolumes
* PersistentVolumeClaims
* StorageClass
* Kubernetes Jobs
* Kubernetes CronJobs
* Health checks
* Resource requests and limits
* Kubernetes DNS
* Rolling Updates
* Database persistence
* Database backups
* Git and GitHub
