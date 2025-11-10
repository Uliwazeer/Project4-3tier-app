# Project4-3Tier-App: Three-Tier DevSecOps Kubernetes Deployment on AWS EKS

**Author:** Ali wazeer  
**Tech Stack:** AWS EKS, ArgoCD, Prometheus, Grafana, Jenkins, Terraform, Docker, Kubernetes, React, Django, PostgreSQL, Trivy, SonarQube, OWASP Dependency-Check 
![DevSecOps_Project](https://github.com/user-attachments/assets/c64357f7-af80-45ce-a422-c1dcf146556c)

## Jenkins Pipeline Architecture
<img width="1344" height="768" alt="Gemini_Generated_Image_hmqx61hmqx61hmqx" src="https://github.com/user-attachments/assets/57fafb3a-49cf-4b32-a5be-2901c73f40df" />

---

## 🚀 Project Overview

This project demonstrates a **secure, scalable, and fully automated three-tier web application** deployment on **AWS EKS** using modern **DevSecOps practices**.  

The three-tier architecture consists of:  
- **Frontend:** React  
- **Backend:** Django Rest Framework (Python)  
- **Database:** PostgreSQL  

It leverages **CI/CD automation, GitOps, containerization, monitoring, and infrastructure as code** to deliver a production-ready system.

---

## 🔗 Resources

- **GitHub Repository:** [project4-3tier-app]([https://github.com/Uliwazeer/project4-3tier-app]  


**Contact:**  
- Email: ali.wazeer2000@gmail.com  
- LinkedIn: /aliwazeer  
- GitHub: [aliwazeer](https://github.com/Uliwazeer)

---

## 📑 Table of Contents

1. [Introduction](#introduction)  
2. [IAM User Setup](#iam-user-setup)  
3. [Terraform & AWS CLI Installation](#terraform--aws-cli-installation)  
4. [S3 Bucket & DynamoDB Table Setup](#s3-bucket--dynamodb-table-setup)  
5. [Jenkins EC2 Server Setup](#jenkins-ec2-server-setup)  
6. [Installing Jenkins Plugins](#installing-jenkins-plugins)  
7. [SonarQube Integration](#sonarqube-integration)  
8. [Amazon ECR Repositories](#amazon-ecr-repositories)  
9. [EKS Cluster Deployment](#eks-cluster-deployment)  
10. [Prometheus & Grafana Installation](#prometheus--grafana-installation)  
11. [Jenkins CI/CD Pipelines](#jenkins-cicd-pipelines)  
12. [ArgoCD Installation & Application Deployment](#argocd-installation--application-deployment)  
13. [DNS Configuration](#dns-configuration)  
14. [Data Persistence](#data-persistence)  
15. [Conclusion & Cleanup](#conclusion--cleanup)

---

## Introduction

Deploy a **fully automated DevSecOps pipeline** for a three-tier web application on AWS EKS. This project includes **infrastructure as code, CI/CD pipelines, security scanning, GitOps deployment, and cluster monitoring**, providing a blueprint for **production-ready deployments**.

---

## IAM User Setup

1. Create an IAM user in AWS Console with permissions for:  
   - EKS  
   - S3  
   - DynamoDB  
   - ECR  
   - EC2  

2. Generate **Access Key** and **Secret Key** for CLI configuration.  

```bash
aws configure
````

---

## Terraform & AWS CLI Installation

* Install **Terraform**: [Terraform Official Website](https://www.terraform.io/)
* Install **AWS CLI**: [AWS CLI Installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Verify installation:

```bash
terraform --version
aws --version
```

---

## S3 Bucket & DynamoDB Table Setup

* **S3 Bucket**: Store Terraform state files
* **DynamoDB Table**: Used for Terraform state locking
* Follow the video guide for naming conventions and creation.

---

## Jenkins EC2 Server Setup

1. Clone the repository:

```bash
git clone https://github.com/Uliwazeer/project4-3tier-app.git
cd jenkins-server-terraform
```

2. Update `backend.tf` with your S3 bucket and DynamoDB table names.
3. Create EC2 key pair:

```bash
aws ec2 create-key-pair --key-name project4-3tier-app-key --query "KeyMaterial" --output text > project4-3tier-app.pem
chmod 400 project4-3tier-app.pem
```

4. Deploy Jenkins server:

```bash
terraform init
terraform validate
terraform apply
```

5. SSH to Jenkins server:

```bash
ssh -i "project4-3tier-app.pem" ubuntu@<EC2_PUBLIC_IP>
aws configure
```

---

## Installing Jenkins Plugins

Install the following plugins:

* AWS Credentials
* AWS Steps
* Docker
* Eclipse Temurin Installer
* NodeJS
* OWASP Dependency-Check
* SonarQube Scanner

---

## SonarQube Integration

1. Access SonarQube at:

```
http://<JENKINS_SERVER_IP>:9090
```

* Default credentials: `admin/admin`
* Setup projects for frontend and backend code analysis
* Generate **sonar-token** and configure webhook in Jenkins.

---

## Amazon ECR Repositories

* Create ECR repositories for frontend & backend Docker images.
* Add credentials to Jenkins: AWS keys, GitHub, sonar-token, ECR repos, and AWS Account ID.

---

## EKS Cluster Deployment

1. Deploy cluster using `eksctl`:

```bash
eksctl create cluster --name project4-3tier-eks --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
```

2. Configure **Load Balancer** with AWS Load Balancer Controller using Helm.
3. Validate pods and nodes:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```
<img width="1366" height="343" alt="Screenshot (647)" src="https://github.com/user-attachments/assets/2f5d0171-a742-498e-9e52-c20b8a4567ba" />

---

## Prometheus & Grafana Installation

1. Add Helm repos:

```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

2. Create monitoring namespace:

```bash
kubectl create namespace monitoring
```

3. Install kube-prometheus-stack:

```bash
helm install stable prometheus-community/kube-prometheus-stack -n monitoring
```

4. Expose Prometheus & Grafana via LoadBalancer for external access.

* Grafana default login: `admin/prom-operator`

---

## Jenkins CI/CD Pipelines

* Build, test, scan, and deploy **frontend and backend** applications.
* Integrates **Trivy** for security scans and **SonarQube** for code quality.


---

## ArgoCD Installation & Application Deployment

1. Create namespaces:

```bash
kubectl create namespace argocd
kubectl create namespace three-tier
```

2. Install ArgoCD:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

3. Retrieve ArgoCD credentials:

```bash
sudo apt install jq -y
export ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o json | jq -r '.status.loadBalancer.ingress[0].hostname')
export ARGO_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
```

4. Deploy applications via ArgoCD:

* `database`
* `backend`
* `frontend`
* `frontend-ingress`
* `backend-ingress`

---

## DNS Configuration

* Configure DNS to point to EKS Load Balancer or Ingress endpoints for frontend and backend applications.
  *(Detailed steps in the video.)*

---

## Data Persistence

* Configure **Persistent Volume (PV)** and **Persistent Volume Claim (PVC)** for database pods.
* Ensures data remains intact even if pods are deleted.

---

## Conclusion & Cleanup

* Test end-to-end deployment, verify monitoring, security, and scaling.
* To avoid AWS costs after testing, delete the EKS cluster:

```bash
eksctl delete cluster --name project4-3tier-eks --region us-west-2 --wait --timeout 30m
```


---

This **README** provides a complete blueprint for deploying a **production-grade three-tier DevSecOps application** on AWS EKS with **automation, security, and scalability**.

<img width="1366" height="721" alt="Screenshot (645)" src="https://github.com/user-attachments/assets/1d059f3f-7aab-4e77-aa73-5074a0a162fa" />
<img width="1366" height="717" alt="Screenshot (644)" src="https://github.com/user-attachments/assets/1bc77589-5a7b-4b69-aac1-ed4fac959f54" />
<img width="1366" height="717" alt="Screenshot (643)" src="https://github.com/user-attachments/assets/2ff7f331-8b74-450a-97da-cf2d1cce0a40" />
<img width="1366" height="689" alt="Screenshot (609)" src="https://github.com/user-attachments/assets/0b7296fc-ce56-451f-b70c-549176112c8a" />
<img width="1366" height="768" alt="Screenshot (607)" src="https://github.com/user-attachments/assets/c0e9bda0-c737-40bc-985b-d458d1d46957" />
<img width="1366" height="685" alt="Screenshot (600)" src="https://github.com/user-attachments/assets/6606bbca-a4cd-4429-bb79-247b3278387d" />
<img width="1366" height="685" alt="Screenshot (598)" src="https://github.com/user-attachments/assets/21aa61e1-d6ba-4601-861f-6333318d8140" />
<img width="1366" height="697" alt="Screenshot (578)" src="https://github.com/user-attachments/assets/c93bf192-c327-4441-95a9-7c6055f0ae95" />
<img width="1366" height="685" alt="Screenshot (577)" src="https://github.com/user-attachments/assets/e1f86708-fe93-47ab-a20b-148c61a249dc" />
<img width="1366" height="689" alt="Screenshot (561)" src="https://github.com/user-attachments/assets/acc8ab24-3612-42fd-92ab-f21feb991eee" />
<img width="1366" height="685" alt="Screenshot (560)" src="https://github.com/user-attachments/assets/8cb2ffd1-9a80-4668-a35f-0ead2fb827fa" />
<img width="1366" height="343" alt="Screenshot (647)" src="https://github.com/user-attachments/assets/e5967dd4-5d4f-4253-879b-a159d1c8002e" />
<img width="1366" height="685" alt="Screenshot (617)" src="https://github.com/user-attachments/assets/85cf00a9-6f5e-42e2-929b-84c81e375b75" />
<img width="1366" height="693" alt="Screenshot (606)" src="https://github.com/user-attachments/assets/fe762f84-9bd1-4733-a14a-057887aeb360" />
<img width="1366" height="685" alt="Screenshot (597)" src="https://github.com/user-attachments/assets/fcfb7eaf-471d-4f1c-9cd7-a947ab0a1ab6" />
<img width="1366" height="693" alt="Screenshot (574)" src="https://github.com/user-attachments/assets/4484d5fb-664d-4d6f-9086-5a3d8e0e04da" />
<img width="1366" height="689" alt="Screenshot (638)" src="https://github.com/user-attachments/assets/65d365c6-0eac-4d1c-a881-149f5b4257ec" />
<img width="1366" height="685" alt="Screenshot (621)" src="https://github.com/user-attachments/assets/891b5991-49d0-4905-b997-f6894202ba46" />

