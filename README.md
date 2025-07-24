
# EyegoApp - CI/CD with GitHub Actions, EKS and ECR

This repository contains the infrastructure and deployment pipeline for EyegoApp, a containerized Node.js API deployed on Amazon EKS using GitHub Actions and Amazon ECR.

## Project Overview

The application follows a modern DevOps pipeline that automates:
- Docker image build and push to Amazon ECR
- Kubernetes deployment to Amazon EKS
- Infrastructure automation using GitHub Actions

## Technologies Used

| Tool            | Purpose                           |
|-----------------|-----------------------------------|
| AWS EKS         | Kubernetes Cluster Management     |
| AWS ECR         | Docker Image Registry             |
| GitHub Actions  | CI/CD Pipeline Automation         |
| Docker          | Application Containerization      |
| kubectl         | Kubernetes CLI                    |

## Folder Structure

```
eyegoapp/
├── k8s/
│   ├── deployment.yaml
|   |__aws-auth-patch.yaml    
│   └── service.yaml
├── .github/
│   └── workflows/
│       └── deploy.yml
├── Dockerfile
└── README.md
```

## ECR and EKS Configuration

- Amazon ECR repository created:  
  982248023155.dkr.ecr.us-east-1.amazonaws.com/eyegoapi

- EKS Cluster:  
  eyego-cluster deployed in us-east-1 with public access enabled for kubectl.

## CI/CD Pipeline (.github/workflows/deploy.yml)

This pipeline is triggered on every push to the main branch and performs the following:

1. Checkout repository code
2. Configure AWS credentials using GitHub secrets
3. Authenticate Docker with Amazon ECR
4. Build and push Docker image to ECR
5. Update kubeconfig to connect to EKS
6. Apply Kubernetes manifests using kubectl

## GitHub Secrets

| Secret Name             | Description                      |
|-------------------------|----------------------------------|
| AWS_ACCESS_KEY_ID       | IAM user with EKS and ECR access |
| AWS_SECRET_ACCESS_KEY   | Secret access key for IAM user   |

## Deployment

The application is deployed using standard Kubernetes resources:

- deployment.yaml: Defines the app containers and replicas
- service.yaml: Exposes the app to the internet using LoadBalancer service

Once deployed, the app is publicly accessible via the AWS ELB URL provided by the service.

## Post-Deployment Checks

Run the following after deployment:

```
kubectl get pods
kubectl get svc
```

To access the app in your browser:

```
http://a0b12c34d567e890f1234.us-east-1.elb.amazonaws.com
```

## Future Improvements
- Add Helm chart support for better packaging
- Add Prometheus and Grafana for observability
##  Deployment & Migration Guide
This section explains how to migrate the current AWS EKS-based deployment to either **Google Kubernetes Engine (GKE)** or **Alibaba Cloud Container Service for Kubernetes (ACK)** with minimal changes.

---

### 1. Container Registry Migration

* **From**: AWS Elastic Container Registry (ECR)
* **To**:

  * **GKE**: Google Container Registry (GCR) or Artifact Registry  
    Example: `gcr.io/your-project-id/eyegoapi:latest`

  * **Alibaba Cloud**: Alibaba Container Registry  
    Example: `registry.cn-hangzhou.aliyuncs.com/your-namespace/eyegoapi:latest`

**Steps**:

```bash
# Tag image for new registry
docker tag eyegoapi:latest gcr.io/your-project-id/eyegoapi:latest

# Authenticate and push to GCR
gcloud auth configure-docker
docker push gcr.io/your-project-id/eyegoapi:latest
```

---

### 2. Kubernetes Cluster Migration

* **To GKE**:

```bash
gcloud container clusters create eyego-cluster   --num-nodes=2   --region=us-central1   --enable-ip-alias

gcloud container clusters get-credentials eyego-cluster --region us-central1
```

* **To Alibaba ACK**:

```bash
aliyun cs DescribeClusterUserKubeconfig --ClusterId <YourClusterID>
```

---

### 3. Deploy to New Cluster

```bash
kubectl apply -f k8s/
```

---

### 4. CI/CD Pipeline Updates

* Replace AWS credentials with:
  - `google-github-actions/auth` for GKE
  - Alibaba Cloud CLI auth for ACK

* Update image repository and `kubeconfig` commands
---
## Author

Built by Mohamed Elmahdy Cloud&Devops Engineer
