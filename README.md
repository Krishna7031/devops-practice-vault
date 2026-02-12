# 🚀 Daily DevOps Practice

**Wake up → Practice → Commit → Repeat → Land 18 LPA Job**

This repository powers your daily morning DevOps routine: Dockerfiles → Docker Compose → Kubernetes manifests → Cloud deployment. Complete one section per day, commit changes, build interview confidence.

[![Daily Practice](https://img.shields.io/badge/Daily-Practice-brightgreen)](https://github.com/yourusername/daily-devops-practice)

## 🎯 Quickstart Routine (5-15 mins daily)

```bash
# 1. Build & test containers
docker build -t practice-app .
docker-compose up

# 2. Start K8s cluster
minikube start  # or kind/k3s

# 3. Deploy to K8s
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# 4. Test
minikube service my-app

# 5. Commit progress
git add . && git commit -m "Day X: [Python|Node|Compose|K8s]"
```

## 📦 1. Python Dockerfile

**Basic Flask app** - Perfect for web service practice.

**`python-app/Dockerfile`:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

**`python-app/requirements.txt`:**
```
Flask==2.3.2
```

**`python-app/app.py`:**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "✅ Python Docker Practice Success!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

**Test:** `docker build -t py-app ./python-app && docker run -p 5000:5000 py-app`

## 🟦 2. Node.js Dockerfile (Multi-stage)

**Express server** - Production-grade build practices.

**`node-app/Dockerfile`:**
```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build

# Production stage (smaller image)
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json .
RUN npm ci --only=production && npm cache clean --force
EXPOSE 3000
CMD ["node", "dist/app.js"]
```

**`node-app/package.json`:**
```json
{
  "name": "node-practice",
  "version": "1.0.0",
  "scripts": {
    "build": "echo 'Built for production'"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

**`node-app/app.js`:**
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('✅ Node Docker Practice Success!');
});

app.listen(3000, () => {
    console.log('🚀 Server running on port 3000');
});
```

**Test:** `docker build -t node-app ./node-app && docker run -p 3000:3000 node-app`

## 🐳 3. Docker Compose (Multi-container)

**Python + Redis stack** - Real-world microservices practice.

**`docker-compose.yml`:**
```yaml
version: '3.8'
services:
  web:
    build: ./python-app
    ports:
      - "5000:5000"
    depends_on:
      - redis
    environment:
      - REDIS_HOST=redis
  
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
```

**Test:** `docker-compose up --build`

## ☸️ 4. Kubernetes Deployment

**3 replicas with health checks** - Interview-ready manifests.

**`k8s/deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: py-app:latest  # Tag your built image
        ports:
        - containerPort: 5000
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
```

## 🌐 5. Kubernetes Service

**NodePort exposure** - Easy minikube testing.

**`k8s/service.yaml`:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 5000
    targetPort: 5000
    nodePort: 30000
    protocol: TCP
```

## ✅ Daily Practice Checklist

### Day 1-7: Master Basics
- [ ] Python Dockerfile + build + run
- [ ] Node.js Dockerfile (multi-stage) + build + run
- [ ] Docker Compose (Python + Redis) + test connectivity
- [ ] Minikube start + deploy YAMLs
- [ ] Verify: `kubectl get pods,svc` + `minikube service`

### Day 8-14: Add Complexity
- [ ] Scale: `kubectl scale deployment my-app --replicas=5`
- [ ] Rolling update: Change image tag, `kubectl apply`
- [ ] Debug: `kubectl logs`, `kubectl describe pod`
- [ ] Port-forward: `kubectl port-forward svc/my-app-service 5000:5000`

### Day 15+: Production Patterns
- [ ] ConfigMaps/Secrets for environment vars
- [ ] HorizontalPodAutoscaler
- [ ] Deploy to cloud (GKE/EKS)

## 📊 Progress Tracker

```
Week 1: [Dockerfiles] ✅
Week 2: [Compose + K8s] ✅
Week 3: [Scaling + Debug] ⏳
Week 4: [Cloud Deploy] ⏳
```

## 💼 Interview Prep Commands

```bash
# Status checks (interview favorites)
kubectl get pods -o wide
kubectl get svc
kubectl describe pod <pod-name>
kubectl logs <pod-name>

# Debugging
kubectl exec -it <pod-name> -- /bin/sh
kubectl port-forward pod/<pod-name> 8080:80

# Scale & update
kubectl scale deployment/my-app --replicas=5
kubectl rollout status deployment/my-app
```
