# Daily DevOps-to-Prod Practice

## Daily flow (high-level checklist)

Aim: ~2 hours initially, later ~40 minutes.

1. Pick a stack for today (Python/Flask or Node/Express).
2. Create or tweak a tiny app (can copy from AI if stuck).
3. Write a Dockerfile and build the image.
4. Write docker-compose.yml and run it locally.
5. Write Kubernetes deployment.yaml and service.yaml and deploy to a local cluster.
6. Write/update Terraform main.tf for a simple AWS EC2-based deployment.
7. Write/update Jenkinsfile to simulate CI/CD (build, push, deploy).
8. Clean up and commit to GitHub.

***

## Folder structure for this repo

```bash
daily-devops-practice/
├─ app/                # Python or Node app of the day
│  ├─ python/          # (optional) keep past Python variants
│  └─ node/            # (optional) keep past Node variants
├─ docker/             # Dockerfiles
├─ compose/            # docker-compose files
├─ k8s/                # Kubernetes manifests
├─ infra/              # Terraform for AWS
├─ ci/                 # Jenkinsfile and CI-related scripts
└─ README.md           # This file
```

You can simplify this and just keep one active version per day if you prefer.

***

## Step 0 – Prerequisites

Before using this routine:

- Git installed and GitHub repo created.
- Docker and docker-compose installed.
- A local Kubernetes cluster (minikube/kind) and kubectl configured.
- Terraform installed and AWS credentials configured (via environment variables or profile).
- A Jenkins instance (local Docker, VM, or Jenkins on k8s) with access to Docker and kubectl.

***

## Step 1 – Create a tiny app (Python or Node)

Pick **one** per day: Python/Flask or Node/Express.

### Option A – Python Flask app

`app/app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Flask DevOps daily practice!"

if __name__ == "__main__":
    # Listen on all interfaces in container
    app.run(host="0.0.0.0", port=5000)
```

`app/requirements.txt`:

```txt
flask==3.0.0
```

Run locally (optional sanity check):

```bash
cd app
pip install -r requirements.txt
python app.py  # Visit http://localhost:5000
```

### Option B – Node.js Express app

`app/index.js`:

```js
const express = require("express");
const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.send("Hello from Express DevOps daily practice!");
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

`app/package.json`:

```json
{
  "name": "devops-daily-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

Install and run locally (optional):

```bash
cd app
npm install
npm start  # Visit http://localhost:3000
```

***

## Step 2 – Write Dockerfile

Create one Dockerfile per day depending on which stack you chose.

### Dockerfile for Python/Flask

`docker/Dockerfile.python` (or just `Dockerfile` if today is Python day):

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

EXPOSE 5000

CMD ["python", "app.py"]
```

Build image:

```bash
docker build -f docker/Dockerfile.python -t devops-daily-python:latest .
```

Run container:

```bash
docker run --rm -p 5000:5000 devops-daily-python:latest
```

### Dockerfile for Node/Express

`docker/Dockerfile.node` (or just `Dockerfile` if today is Node day):

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY app/package*.json ./
RUN npm install --only=production

COPY app/ .

EXPOSE 3000

CMD ["npm", "start"]
```

Build image:

```bash
docker build -f docker/Dockerfile.node -t devops-daily-node:latest .
```

Run container:

```bash
docker run --rm -p 3000:3000 devops-daily-node:latest
```

***

## Step 3 – Write docker-compose.yml

Create a compose file that runs your app container, optionally with a fake “db” to simulate multi-service setups.

`compose/docker-compose.yml`:

```yaml
version: "3.9"

services:
  web:
    image: devops-daily-python:latest  # or devops-daily-node:latest
    build:
      context: ..
      dockerfile: docker/Dockerfile.python  # or docker/Dockerfile.node
    ports:
      - "5000:5000"  # Python
      # - "3000:3000"  # Use this instead for Node
    environment:
      - APP_ENV=dev
      - APP_NAME=daily-devops

  # Example extra service for practice (comment in/out as needed)
  # redis:
  #   image: redis:7-alpine
  #   ports:
  #     - "6379:6379"
```

Commands:

```bash
cd compose
docker-compose up --build
# Test in browser: http://localhost:5000 or http://localhost:3000
docker-compose down
```

***

## Step 4 – Kubernetes Deployment and Service

Target: deploy the same image to a local cluster (minikube or kind).

`k8s/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-daily-app
  labels:
    app: devops-daily-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-daily-app
  template:
    metadata:
      labels:
        app: devops-daily-app
    spec:
      containers:
        - name: web
          image: devops-daily-python:latest  # Or devops-daily-node:latest (if using local registry, adjust)
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 5000  # 3000 for Node
          env:
            - name: APP_ENV
              value: "k8s"
            - name: APP_NAME
              value: "daily-devops"
```

`k8s/service.yaml` (NodePort for easy local access):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devops-daily-service
spec:
  type: NodePort
  selector:
    app: devops-daily-app
  ports:
    - name: http
      port: 5000        # Service port (or 3000)
      targetPort: 5000  # Container port (or 3000)
      nodePort: 30080   # NodePort (range 30000–32767)
```

Commands (example with minikube):

```bash
minikube start

kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

kubectl get pods
kubectl get svc devops-daily-service

# Access via minikube:
minikube service devops-daily-service
# or Node IP + nodePort: http://<node-ip>:30080
```

Daily variations:

- Change replicas (e.g., 1, 3, 5) and run `kubectl apply -f k8s/deployment.yaml`.
- Change container resources (later) and observe scheduling.
- Switch ports depending on Python vs Node.

***

## Step 5 – Terraform: simple AWS EC2 deployment

Goal: use Terraform to create **one EC2 instance** that installs Docker and runs your image from Docker Hub.

> For **real deployments**, add VPC, subnets, remote state, variables, etc.  
> For **daily practice**, keep it minimal and focus on the IaC flow.

`infra/main.tf`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  required_version = ">= 1.5.0"
}

provider "aws" {
  region = "ap-south-1"
  # Credentials picked from environment or shared config profile
}

resource "aws_security_group" "devops_daily_sg" {
  name        = "devops-daily-sg"
  description = "Allow SSH and app traffic"

  # Default VPC is used for simplicity
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 5000
    to_port     = 5000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "devops_daily_ec2" {
  ami           = "ami-0c2af51e265bd5e0e" # Example: Amazon Linux 2 in ap-south-1 (update if needed)
  instance_type = "t2.micro"

  vpc_security_group_ids = [aws_security_group.devops_daily_sg.id]

  tags = {
    Name = "devops-daily-ec2"
  }

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              amazon-linux-extras install docker -y
              systemctl enable docker
              systemctl start docker

              docker pull <your-dockerhub-username>/devops-daily:latest
              docker run -d -p 80:5000 --name devops-daily-app <your-dockerhub-username>/devops-daily:latest
              EOF
}
```

Usage:

```bash
cd infra

terraform init
terraform plan
terraform apply  # Type 'yes' when prompted
```

After apply completes:

- Note the public IP of `aws_instance.devops_daily_ec2`.
- Visit `http://<public-ip>` in your browser.

Clean up after practice:

```bash
terraform destroy
```

Daily variations:

- Change region or instance type.
- Change exposed port or image name.
- Later, add variables and outputs.

***

## Step 6 – Jenkinsfile: simple CI/CD pipeline

Goal: simulate a CI pipeline that:

1. Checks out your code.
2. Builds and tags a Docker image.
3. Pushes the image to Docker Hub.
4. Deploys to Kubernetes (or triggers Terraform).

`ci/Jenkinsfile` (Declarative):

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "<your-dockerhub-username>/devops-daily"
        KUBECONFIG   = credentials('kubeconfig-cred-id') // Optional: if using Jenkins credentials
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -f docker/Dockerfile.python -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/devops-daily-app
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
```

Daily variations:

- Swap Dockerfile for Node version in the build step.
- Add a `Test` stage that runs unit tests inside a container.
- Replace “Deploy to Kubernetes” with a stage that runs `terraform apply` from `infra/`.

***

## Step 7 – Using AI when you get stuck

When blocked on:

- Small app logic (new endpoint, different response).
- Requirements (`requirements.txt`, `package.json`).
- Minor YAML details (ports, apiVersion, etc.).

You can:

1. Ask an AI assistant (e.g., “Give me a minimal Flask app that returns JSON with /health endpoint.”).
2. Copy the relevant snippet into `app/` or the respective config file.
3. Read through and **understand** it before running, so the concept sticks.

Repeat the same whenever you hit a wall, but always try to reason about the change afterward.

***

## How to run the full flow in one sitting

This can be your **default morning script**:

```bash
# 1) Pull latest and create today's branch
git pull origin main
git checkout -b practice-$(date +%Y-%m-%d)

# 2) Update app/ (Python or Node)
#    - edit app.py/index.js + requirements/package.json

# 3) Docker build + run
docker build -f docker/Dockerfile.python -t devops-daily-python:latest .
docker run --rm -p 5000:5000 devops-daily-python:latest

# 4) docker-compose
cd compose
docker-compose up --build -d
docker-compose ps
docker-compose down
cd ..

# 5) Kubernetes
minikube start
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get pods
kubectl get svc devops-daily-service

# 6) (Optional on chosen days) Terraform
cd infra
terraform plan
terraform apply -auto-approve
# test with public IP
terraform destroy -auto-approve
cd ..

# 7) (Optional) Trigger Jenkins pipeline via UI or webhooks

# 8) Commit and push
git status
git add .
git commit -m "Daily devops practice $(date +%Y-%m-%d)"
git push origin HEAD
```

***
