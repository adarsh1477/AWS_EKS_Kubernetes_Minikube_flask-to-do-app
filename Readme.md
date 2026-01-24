ToDo Reminder App — Flask + MongoDB on AWS EKS

A production-style deployment of a Flask To-Do application on AWS Elastic Kubernetes Service (EKS) with AWS ALB Ingress, persistent MongoDB storage via EBS, and secure IAM Roles for Service Accounts (IRSA).

✅ Application Features

Create, view, and complete reminder tasks

Persistent storage using MongoDB

Responsive minimal UI built with HTML + CSS + JS

Internal service communication inside K8s cluster

🏗️ Infrastructure & Deployment Features
Component	Choice
Container Runtime	Docker
Orchestration	Kubernetes on AWS EKS
Ingress	AWS Load Balancer Controller (ALB)
Storage	Amazon EBS via EBS CSI Driver
Database	MongoDB Stateful container
Access	IAM Roles for Service Accounts (IRSA)
CI/CD	Manual (docker push → apply manifests)
Exposure	Public Ingress (internet-facing ALB)
📂 Repository Structure
AWS_EKS_Kubernetes_flask-to-do-app/
│
├── app.py                           # Flask backend logic
├── Dockerfile                       # Flask image build
├── requirements.txt                 # Python dependencies
│
├── static/                          # CSS & assets
├── templates/                       # HTML templates
│
├── docker-compose.yml               # Local testing (optional)
│
└── Manifests/                       # Kubernetes deployment files
    ├── flask-deployment.yaml        # Flask Deployment
    ├── flask-service.yaml           # Flask Service (ClusterIP)
    ├── ingress.yaml                 # ALB Ingress
    ├── mongodb-deployment.yaml      # MongoDB Deployment
    ├── mongodb-service.yaml         # MongoDB Service (ClusterIP)
    ├── mongopvc.yaml                # PersistentVolumeClaim
    ├── ebs-iam-policy.json          # IAM policy for EBS CSI
    ├── trust.json                   # OIDC trust relationship

🧱 Architecture Overview

The final working architecture looks like this:

User → ALB Ingress → Flask Service (ClusterIP) → Flask Pods
                                       │
                                       ↓
                              MongoDB Service → MongoDB Pod → EBS Volume

Key Implementation Details

Flask Deployment

Uses image stored in Amazon ECR

Scaled via replicas: 3

MongoDB Deployment

Uses mongo:5.0

Connected via internal ClusterIP service

Persistent storage via EBS Volume

Persistent Storage

PVC ReadWriteOnce

StorageClass: gp2

Bound via EBS CSI driver

Ingress (ALB)

internet-facing

target-type: ip

Uses ingressClassName: alb

📦 Containerization & Image Push Flow
docker build -t flask-app .
docker tag flask-app:latest <AWS_ACCOUNT>.dkr.ecr.<REGION>.amazonaws.com/flask-app:latest
docker push <AWS_ECR_URL>/flask-app:latest


Flask deployment then pulls image from ECR.

🔐 IAM & Permissions
1. ALB Controller IAM

Created via:

Downloading controller policy from AWS GitHub

Creating IAM policy

Creating ServiceAccount in kube-system

Attaching IAM role via IRSA

2. EBS CSI IAM

Required because PVC creation hit:

UnauthorizedOperation: ec2:DescribeAvailabilityZones


Fixed via:

Creating AmazonEKS_EBS_CSI_Driver_Policy

Creating ebs-csi-controller-sa with IAM role via IRSA

Updating EKS Add-On to use this IAM role

🗄️ Data Persistence Behavior

MongoDB writes → Stored on EBS → Survives:

Pod restarts

MongoDB container restarts

Flask restarts

Does not survive if:

EBS volume deleted

PVC deleted

Cluster deleted

🌐 Networking Setup
Component	Type
Flask Service	ClusterIP
MongoDB Service	ClusterIP
Ingress	ALB (public)

Ingress annotations used:

alb.ingress.kubernetes.io/scheme: internet-facing
alb.ingress.kubernetes.io/target-type: ip

🔍 Observability

Tools enabled:

kubectl logs

kubectl describe

metrics-server for HPA (optional)

ALB health checks

🧪 Validation Steps

After apply:

kubectl get pods -o wide
kubectl get svc
kubectl get ingress
kubectl get pvc


Validation criteria:

All Flask pods Running

MongoDB Running

PVC Bound

Ingress has ADDRESS

Data persists after refresh/restart

🏁 Current Status

✔ Application reachable via ALB
✔ MongoDB data persists using EBS
✔ EKS cluster healthy
✔ IRSA configured for ALB & EBS
✔ ECR image pull successful

Cluster later deleted to avoid AWS billing.

🧰 Tech Used

Python Flask

MongoDB

Docker

AWS ECR

AWS EKS

AWS ALB Controller

AWS EBS CSI Driver

Kubernetes

IRSA
