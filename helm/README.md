# 🚀 Full Stack Application - Helm Chart

A production-ready Helm chart for deploying a complete full-stack application with React frontend, Node.js backend, and MongoDB database.

## 📋 Table of Contents

- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Deployment Environments](#deployment-environments)
- [Management](#management)
- [Troubleshooting](#troubleshooting)

## 🏗️ Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    Frontend     │    │     Backend     │    │     MongoDB     │
│   (React/Nginx) │───▶│   (Node.js)     │───▶│   (Database)    │
│     Port: 80    │    │   Port: 5001    │    │   Port: 27017   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ frontend-service│    │ backend-service │    │ mongodb-service │
│   (ClusterIP)   │    │   (ClusterIP)   │    │   (ClusterIP)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                                                       ▼
                                             ┌─────────────────┐
                                             │ Persistent Vol. │
                                             │   (5Gi/20Gi)   │
                                             └─────────────────┘
```

## 📋 Prerequisites

- **Kubernetes cluster** (Minikube, EKS, GKE, AKS, etc.)
- **Helm 3.x** installed
- **kubectl** configured to access your cluster
- **Docker images** available in registry

## 🚀 Quick Start

### 1. Deploy the Application

```bash
# Clone or navigate to the chart directory
cd k8s-helm

# Deploy with default settings (development-like)
helm install full-stack-app .

# Check deployment status
helm status full-stack-app
kubectl get pods -n full-stack-app
```

### 2. Access the Application

```bash
# Access frontend (React app)
kubectl port-forward service/frontend-service 8080:80 -n full-stack-app
# Open http://localhost:8080

# Access backend API
kubectl port-forward service/backend-service 5001:5001 -n full-stack-app
# API available at http://localhost:5001
```

## ⚙️ Configuration

### 📁 File Organization

```
k8s-helm/
├── Chart.yaml                    # Chart metadata
├── values.yaml                   # Default configuration
├── values-dev.yaml              # Development overrides
├── values-prod.yaml             # Production overrides
├── README.md                    # This file
└── templates/
    ├── config/
    │   ├── namespace.yaml       # Namespace definition
    │   └── secrets.yaml         # Application secrets
    ├── deployments/
    │   ├── backend-deployment.yaml
    │   ├── frontend-deployment.yaml
    │   └── mongodb-deployment.yaml
    ├── services/
    │   ├── backend-service.yaml
    │   ├── frontend-service.yaml
    │   └── mongodb-service.yaml
    └── storage/
        ├── mongodb-pv.yaml      # Persistent Volume
        └── mongodb-pvc.yaml     # Persistent Volume Claim
```

### 🔧 Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `images.backend.repository` | Backend Docker image | `mayurj023/full-backend` |
| `images.backend.tag` | Backend image tag | `latest` |
| `images.frontend.repository` | Frontend Docker image | `mayurj023/full-stack_frontend` |
| `replicas.backend` | Backend pod replicas | `1` |
| `replicas.frontend` | Frontend pod replicas | `1` |
| `resources.mongodb.limits.memory` | MongoDB memory limit | `1Gi` |
| `mongodb.storage.size` | Database storage size | `5Gi` |
| `mongodb.auth.username` | MongoDB admin username | `mongodbadmin` |

## 🌍 Deployment Environments

### 🔧 Development Environment

**Optimized for**: Local development, resource efficiency, fast iteration

```bash
helm install full-stack-app . -f values-dev.yaml
```

**Features**:
- Single replica for all services
- Reduced resource requirements
- 1Gi storage for database
- Faster startup times

### 🏭 Production Environment

**Optimized for**: High availability, performance, scalability

```bash
helm install full-stack-app . -f values-prod.yaml
```

**Features**:
- Multiple replicas (Backend: 3, Frontend: 2)
- Enhanced resource limits
- 20Gi storage for database
- Production-grade configuration

### 🎯 Custom Configuration

Create your own values file:

```yaml
# my-custom-values.yaml
replicas:
  backend: 2

images:
  backend:
    tag: "v2.1.0"
  frontend:
    tag: "v1.5.0"

mongodb:
  storage:
    size: 10Gi
```

Deploy with custom values:
```bash
helm install full-stack-app . -f my-custom-values.yaml
```

## 🔄 Management

### Upgrade Application

```bash
# Upgrade with new configuration
helm upgrade full-stack-app . -f values-prod.yaml

# Upgrade with new image versions
helm upgrade full-stack-app . \
  --set images.backend.tag=v2.0.0 \
  --set images.frontend.tag=v1.1.0

# Check upgrade status
helm status full-stack-app
```

### Rollback

```bash
# View release history
helm history full-stack-app

# Rollback to previous version
helm rollback full-stack-app

# Rollback to specific revision
helm rollback full-stack-app 2
```

### Scale Application

```bash
# Scale backend replicas
helm upgrade full-stack-app . --set replicas.backend=5

# Scale frontend replicas
helm upgrade full-stack-app . --set replicas.frontend=3
```

### Uninstall

```bash
# Remove the application
helm uninstall full-stack-app

# Clean up persistent volumes (if needed)
kubectl delete pv mongodb-pv
```

## 🔍 Troubleshooting

### Check Application Status

```bash
# Check Helm release
helm status full-stack-app

# Check all resources
kubectl get all -n full-stack-app

# Check persistent volumes
kubectl get pv,pvc -n full-stack-app
```

### View Logs

```bash
# Backend logs
kubectl logs -l app=backend -n full-stack-app --tail=100

# Frontend logs
kubectl logs -l app=frontend -n full-stack-app --tail=100

# MongoDB logs
kubectl logs -l app=mongodb -n full-stack-app --tail=100

# Follow logs in real-time
kubectl logs -l app=backend -n full-stack-app -f
```

### Debug Deployments

```bash
# Dry run to see generated manifests
helm install full-stack-app . --dry-run --debug

# Template specific files
helm template full-stack-app . -s templates/deployments/backend-deployment.yaml

# Describe problematic pods
kubectl describe pod <pod-name> -n full-stack-app
```

### Common Issues

#### 1. Pods Stuck in Pending
```bash
# Check node resources
kubectl describe nodes

# Check pod events
kubectl describe pod <pod-name> -n full-stack-app
```

#### 2. Database Connection Issues
```bash
# Check MongoDB service
kubectl get svc mongodb-service -n full-stack-app

# Test connectivity from backend pod
kubectl exec -it <backend-pod> -n full-stack-app -- nslookup mongodb-service
```

#### 3. Persistent Volume Issues
```bash
# Check PV/PVC status
kubectl get pv,pvc -n full-stack-app

# Check PVC events
kubectl describe pvc mongodb-pvc -n full-stack-app
```

## 🔒 Security Notes

- MongoDB credentials are stored in Kubernetes secrets
- JWT secrets are base64 encoded
- For production, consider external secret management (AWS Secrets Manager, HashiCorp Vault)
- Review and update default passwords before production deployment

## 🤝 Contributing

1. Make changes to templates or values
2. Test with `helm template` and `helm install --dry-run`
3. Update version in `Chart.yaml`
4. Update this README with any new features

## 📄 License

This Helm chart is licensed under the MIT License.
