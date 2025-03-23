# 📝 ToDo Reminder - Flask App on AWS EKS

This is a simple Flask-based ToDo Reminder app deployed on **AWS Elastic Kubernetes Service (EKS)**.  
It supports task creation, completion tracking, and priority assignment.

---

## 🚀 Project Features

- Built using Flask, HTML, and CSS
- Deployed via Kubernetes on AWS EKS
- Persistent storage with EBS volume for MongoDB
- Health monitoring, rolling updates, and service exposure
- Separate YAML files for deployments, services, and storage

---

## 📂 Project Structure

AWS_EKS_Kubernetes_flask-to-do-app/ │ ├── app.py # Flask backend logic ├── Dockerfile # Docker image for the app ├── docker-compose.yml ├── requirements.txt │ ├── static/ # CSS and image assets ├── templates/ # HTML templates │ ├── flask-deployment.yaml ├── health-monitoring.yaml ├── mongodb-deployment.yaml ├── replication-controller.yaml ├── rolling-update.yaml ├── service.yaml │ └── Screenshots/ # 📸 Screenshots of running app

yaml
Copy
Edit

---

## 📸 Deployed App (EKS)

**Live Application**(non functional cluster deleted)  
🔗 [ToDo App on AWS](http://aaa64e290e604414b88a8a986b4a58d8-0c9f3e865c5777f3.elb.us-east-1.amazonaws.com/list)

---

## 🧰 Tools & Technologies

- Python (Flask)
- MongoDB
- Docker
- Kubernetes (YAML)
- AWS EKS
- Amazon EBS (via CSI driver)

---

## 🛠️ Deployment Highlights

```bash
# Create EKS cluster
eksctl create cluster \
  --name flask-cluster \
  --region us-east-1 \
  --version 1.31 \
  --nodegroup-name flask-node-group \
  --node-type t2.small \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed

# Scale node group to fix volume affinity issue
eksctl scale nodegroup \
  --cluster flask-cluster \
  --name flask-node-group \
  --nodes=4 \
  --nodes-min=3 \
  --nodes-max=6 \
  --region us-east-1

# Enable IAM OIDC and EBS CSI driver
eksctl utils associate-iam-oidc-provider --cluster flask-cluster --region us-east-1
eksctl create addon --name aws-ebs-csi-driver --cluster flask-cluster --region us-east-1

# Deploy app and services
kubectl apply -f mongodb-deployment.yaml
kubectl apply -f flask-deployment.yaml
kubectl apply -f health-monitoring.yaml
kubectl apply -f replication-controller.yaml
kubectl apply -f rolling-update.yaml
kubectl apply -f service.yaml
⚙️ Notes
StorageClass changed from gp1 ➝ gp2 for dynamic provisioning on AWS

MongoDB nodeSelector set to zone: us-east-1c to match EBS volume

Docker image: arr8154/flask-todo used during deployment