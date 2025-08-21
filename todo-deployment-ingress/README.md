# Todo Application with NGINX Ingress

This repository contains Kubernetes manifests for deploying a Todo application with NGINX Ingress path-based routing in the `todo-ingress` namespace.

## Architecture

```
                    ┌─────────────────┐
                    │   NGINX Ingress │
                    │  todo-app.local │
                    └─────────┬───────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
    ┌─────────────────┐ ┌─────────────┐ ┌─────────────┐
    │    Frontend     │ │ User Service│ │Task Service │
    │     (React)     │ │   (Django)  │ │    (Go)     │
    │    Port: 80     │ │  Port: 8000 │ │ Port: 8001  │
    └─────────────────┘ └─────────────┘ └─────────────┘
                              │               │
                              └───────┬───────┘
                                      │
                    ┌─────────────────┼───────────────┐
                    ▼                 ▼               ▼
              ┌───────────┐   ┌─────────────┐  ┌──────────────┐
              │PostgreSQL │   │  RabbitMQ   │  │    Volume    │
              │  Port:5432│   │Port:5672/   │  │   Storage    │
              │           │   │    15672    │  │              │
              └───────────┘   └─────────────┘  └──────────────┘
```

## File Structure

```
kubernetes/
├── 00-namespace.yaml          # Namespace definition
├── 01-secrets.yaml           # Database and RabbitMQ secrets
├── 02-configmaps.yaml        # Application and RabbitMQ configuration
├── 03-postgres.yaml          # PostgreSQL StatefulSet and Service
├── 04-rabbitmq.yaml          # RabbitMQ Deployment, Service, and PVC
├── 05-user-service.yaml      # User Service Deployment and Service
├── 06-task-service.yaml      # Task Service Deployment and Service
├── 07-frontend.yaml          # Frontend Deployment and Service
├── 08-ingress.yaml           # NGINX Ingress configuration
├── 09-ingress-ssl.yaml       # SSL/TLS enabled Ingress (optional)
```

## Quick Start

### 1. Prerequisites


nfs-subdir extenral provisionar

implementaitons:

install ingress controller with helm :

helm upgrade ingress-nginx ingress-nginx/ingress-nginx   --namespace ingress-nginx   --set controller.service.type=NodePort   --set controller.service.nodePorts.http=30090   --set controller.service.nodePorts.https=30443


Ensure you have:
- Kubernetes cluster running
- `kubectl` configured
- NGINX Ingress Controller installed



### 3. Manual Deployment (Alternative)

```bash
# Deploy in order
kubectl apply -f 00-namespace.yaml
kubectl apply -f 01-secrets.yaml
kubectl apply -f 02-configmaps.yaml
kubectl apply -f 03-postgres.yaml
kubectl apply -f 04-rabbitmq.yaml

# Wait for infrastructure
kubectl wait --for=condition=ready pod -l app=postgres -n todo-ingress --timeout=100s
kubectl wait --for=condition=ready pod -l app=rabbitmq -n todo-ingress --timeout=100s

# Deploy services
kubectl apply -f 05-user-service.yaml
kubectl apply -f 06-tasks-service.yaml
kubectl apply -f 07-frontend.yaml



# Deploy ingress
kubectl apply -f 08-ingress.yaml
```

## DNS Configuration

### For Local Development

Add to your `/etc/hosts` file in your local machine
```
<node-ip> todo-app.local
```




## Access URLs

| Service | URL | Description |
|---------|-----|-------------|
| Frontend | http://todo-app.local | Main React application |
| User API | http://todo-app.local/api/users | User management endpoints |
| Task API | http://todo-app.local/api/tasks | Task management endpoints |
| RabbitMQ | http://todo-app.local/rabbitmq | Management interface (admin/admin123) |

## API Endpoints

### User Service (`/api/users`)
- `GET /api/users/` - List users
- `POST /api/users/` - Create user
- `GET /api/users/{id}/` - Get user details
- `PUT /api/users/{id}/` - Update user
- `DELETE /api/users/{id}/` - Delete user

### Task Service (`/api/tasks`)
- `GET /api/tasks/` - List tasks
- `POST /api/tasks/` - Create task
- `GET /api/tasks/{id}/` - Get task details
- `PUT /api/tasks/{id}/` - Update task
- `DELETE /api/tasks/{id}/` - Delete task

## Testing

### Health Checks
```bash
# Test frontend
curl -H "Host: todo-app.local" http://<INGRESS_IP>/

# Test user service
curl -H "Host: todo-app.local" http://<INGRESS_IP>/api/users/

# Test task service
curl -H "Host: todo-app.local" http://<INGRESS_IP>/api/tasks/
```

### Browser Testing
1. Add DNS entry to `/etc/hosts`
2. Navigate to http://todo-app.local
3. The React frontend should load and make API calls to the backend services

