# Hotel Booking Platform - Infrastructure Repository

This repository contains the Kubernetes infrastructure and GitOps deployment configuration for the Hotel Booking Platform.

The infrastructure is responsible for deploying, managing, and monitoring the complete application stack, including:

* Frontend application
* Backend API
* MongoDB database
* Nexus Docker Registry
* ArgoCD GitOps deployment platform
* Grafana monitoring stack

All application deployments are managed using Kubernetes and synchronized automatically using ArgoCD.

---

# Architecture Overview

The deployment architecture:

```
Developer
   |
   v
GitHub Repositories
   |
   |
   +----------------+
   |                |
frontend-repo   backend-repo
   |                |
   v                v
GitHub Actions CI/CD Pipelines
   |
   v
Nexus Docker Registry
   |
   v
Kubernetes Cluster
   |
   +-------------------+
   |                   |
Frontend Pods      Backend Pods
                        |
                        v
                    MongoDB
```

Monitoring:

```
Kubernetes Cluster
        |
        v
    Promtail
        |
        v
       Loki
        |
        v
     Grafana
        |
        v
   Dashboards
```

---

# Technology Stack

| Component          | Technology                  |
| ------------------ | --------------------------- |
| Container Runtime  | Docker Desktop Kubernetes   |
| Orchestration      | Kubernetes                  |
| GitOps             | ArgoCD                      |
| CI/CD              | GitHub Actions              |
| Container Registry | Nexus Repository            |
| Frontend           | React + Nginx               |
| Backend            | Node.js + Express           |
| Database           | MongoDB                     |
| Monitoring         | Prometheus + Loki + Grafana |

---

# Repository Structure

```
infra-repo/
│
├── argocd/
│   ├── applications/
│   │   ├── frontend-app.yaml
│   │   |── backend-app.yaml
|   |   └── nexus-app.yaml
│
├── frontend/
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
|   ├── frontend-config.yaml
|   ├── frontend-namespace.yaml
│
├── backend/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── backend-config.yaml
│   ├── backend-namespace.yaml
│   └── backend-secret.yaml
│
├── mongodb/
│   ├── mongodb-deployment.yaml
│   ├── mongodb-service.yaml
│   ├── mongodb-config.yaml
│   ├── mongodb-secret.yaml
│   ├── mongodb-pvc.yaml
│
├── nexus/
│   ├── nexus-deployment.yaml
│   ├── nexus-namespace.yaml
│   ├── nexus-pvc.yaml
│   └── nexus-service.yaml
│
└── monitoring/
    ├── loki-values.yaml
    └── namespace.yaml
```

---

# Kubernetes Namespaces

The cluster is organized into dedicated namespaces:

| Namespace      | Purpose                      |
| -------------- | ---------------------------- |
| hotel-frontend | Frontend application         |
| hotel-backend  | Backend API and database     |
| argocd         | GitOps deployment controller |
| nexus          | Docker registry              |
| monitoring     | Prometheus, Loki, Grafana    |

---

# Application Deployment

## Frontend Deployment

The frontend application is deployed using:

* Kubernetes Deployment
* Kubernetes Service
* Nginx container

The frontend image is pulled from Nexus:

```
frontend:<version>
```

Example:

```
172.18.0.4:30082/frontend:v3
```

---

## Backend Deployment

The backend application is deployed using:

* Kubernetes Deployment
* Kubernetes Service
* ConfigMap
* Secret

Backend configuration:

### Container Image Management

Backend container images are referenced using immutable SHA-based tags.

Example:

```yaml
image: host.docker.internal:8082/backend:f5c6a361dcaedbff19318653b497dfa098fb8cd8


```
PORT=3000
MONGO_URL=mongodb://mongodb.hotel-backend.svc.cluster.local:27017/hotelapp
```

---

## MongoDB Deployment

MongoDB provides persistent storage using:

* PersistentVolumeClaim
* PersistentVolume
* Kubernetes Service

MongoDB stores reservation data:

```
Database:
hotelapp

Collection:
reservations
```

---

# Docker Registry - Nexus

Nexus is used as the private Docker registry.

Image flow:

```
GitHub Actions
       |
       v
Docker Build
       |
       v
Nexus Registry
       |
       v
Kubernetes Pull
```

Example image:

```
172.18.0.4:30082/backend:v3
```

---

# CI/CD Pipeline

Each application repository contains a GitHub Actions workflow.

Pipeline flow:

```
Code Commit
     |
     v
GitHub Actions
     |
     +--> Build Docker Image
     |
     +--> Tag Image
     |
     +--> Push Image to Nexus
     |
     v
Update Kubernetes Manifest
     |
     v
ArgoCD Sync
```

---

# ArgoCD GitOps Deployment

ArgoCD continuously monitors Kubernetes manifests stored in this repository.

Deployment flow:

```
Developer
    |
    v
Pull Request
    |
    v
MAIN Branch
    |
    v
ArgoCD Detection
    |
    v
Kubernetes Synchronization
    |
    v
New Pods Running
```

---

# ArgoCD Applications

Configured applications:

## Frontend Application

Source:

```
frontend Kubernetes manifests
```

Destination:

```
Namespace: hotel-frontend
```

---

## Backend Application

Source:

```
backend Kubernetes manifests
```

Destination:

```
Namespace: hotel-backend
```

---

# Secrets Management

Sensitive information is stored using Kubernetes Secrets.

Examples:

* Docker registry credentials
* Database credentials

Example:

```
kubectl get secrets -A
```

Secrets are not stored directly in application repositories.

---

# Persistent Storage

Persistent storage is configured for:

* MongoDB
* Monitoring components
* Nexus

Storage resources:

```
PersistentVolume
PersistentVolumeClaim
```

Verify:

```bash
kubectl get pv
kubectl get pvc -A
```

---

# Monitoring Platform

The monitoring stack is deployed inside:

```
namespace: monitoring
```

Components:

* Prometheus
* Grafana
* Loki
* Promtail

---

## Prometheus

Collects Kubernetes metrics:

* Pod status
* CPU usage
* Memory usage
* Cluster metrics

---

## Loki

Stores application logs.

Collected logs:

* Frontend
* Backend
* MongoDB
* ArgoCD
* Nexus

---

## Grafana

Provides dashboards for:

* Logs
* Metrics
* Application monitoring

Example Loki query:

```
{namespace="hotel-backend"}
```

---

# Useful Kubernetes Commands

## Check all namespaces

```bash
kubectl get namespaces
```

---

## Check pods

```bash
kubectl get pods -A
```

---

## Check deployments

```bash
kubectl get deployments -A
```

---

## ArgoCD status

```bash
kubectl get applications -n argocd
```

---

## View application logs

Backend:

```bash
kubectl logs deployment/backend -n hotel-backend
```

Frontend:

```bash
kubectl logs deployment/frontend -n hotel-frontend
```

---

# Deployment Validation Checklist

Before submission:

## Application

* [x] Frontend deployed successfully
* [x] Backend deployed successfully
* [x] MongoDB stores reservations
* [x] UI communicates with backend

## GitOps

* [x] ArgoCD installed
* [x] Applications configured
* [x] MAIN branch synchronization enabled

## Registry

* [x] Nexus Docker registry deployed
* [x] Images pushed successfully
* [x] Kubernetes pulls images

## Monitoring

* [x] Prometheus running
* [x] Loki running
* [x] Grafana running
* [x] Logs collected from Kubernetes workloads

## OpenBao Secret Management
OpenBao is deployed inside Kubernetes using Helm.

Namespace:
openbao

Components:
- OpenBao server
- Persistent storage
- Web UI

Verification:
kubectl get pods -n openbao
kubectl get svc -n openbao

---

# Author

Gabriel Swack
