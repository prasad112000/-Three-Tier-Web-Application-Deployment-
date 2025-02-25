# Three-Tier Web Application Deployment

## 📌 Project Overview
This repository contains all the necessary code and configurations to deploy a **Three-Tier Web Application** using AWS, Jenkins, Kubernetes, and Terraform. The project covers everything from **infrastructure provisioning** to **CI/CD automation** and **monitoring**.

## 📂 Repository Structure

- **Application-Code/** → Source code for frontend and backend of the application.
- **Jenkins-Pipeline-Code/** → Jenkins pipeline scripts for CI/CD automation.
- **Jenkins-Server-TF/** → Terraform scripts for provisioning a Jenkins Server on AWS.
- **Kubernetes-Manifests-Files/** → Kubernetes manifests to deploy the application on AWS EKS.

## 🛠️ Tools & Technologies Used

- **Infrastructure:** Terraform, AWS CLI
- **CI/CD:** Jenkins, Sonarqube, Terraform, Kubectl
- **Monitoring:** Helm, Prometheus, Grafana
- **GitOps:** ArgoCD
- **Containerization & Orchestration:** Docker, Kubernetes, AWS EKS

## 🚀 Deployment Steps

### **Step 1: IAM User Setup**
1. Create a user `eks-admin` with **AdministratorAccess**.
2. Generate Security Credentials: **Access Key & Secret Access Key**.

### **Step 2: EC2 Setup**
1. Launch an **Ubuntu EC2 instance** in **us-west-2**.
2. SSH into the instance.

### **Step 3: Install AWS CLI v2**
```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### **Step 4: Install Docker**
```sh
sudo apt-get update
sudo apt install docker.io
docker ps
sudo chown $USER /var/run/docker.sock
```

### **Step 5: Install kubectl**
```sh
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### **Step 6: Install eksctl**
```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### **Step 7: Setup EKS Cluster**
```sh
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

### **Step 8: Deploy Application Using Kubernetes Manifests**
```sh
kubectl create namespace workshop
kubectl apply -f Kubernetes-Manifests-Files/
kubectl delete -f Kubernetes-Manifests-Files/
```

### **Step 9: Install AWS Load Balancer**
```sh
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```

### **Step 10: Deploy AWS Load Balancer Controller**
```sh
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
```

## 🗑️ Cleanup Instructions
### **Delete EKS Cluster**
```sh
eksctl delete cluster --name three-tier-cluster --region us-west-2
```

### **Clean Up AWS Resources to Avoid Costs**
- **Stop/Terminate** the EC2 instance.
- **Delete Load Balancer** created in step 9 and 10.
- **Remove Security Groups** created during setup.

## 📌 Summary
This project demonstrates deploying a **Three-Tier Web Application** with AWS, Kubernetes, and CI/CD automation. It covers **infrastructure provisioning**, **CI/CD pipelines**, **monitoring**, and **GitOps best practices** using ArgoCD.

🔹 **By following this guide, you will:**
- Set up an AWS EKS cluster.
- Deploy applications using Kubernetes.
- Automate CI/CD pipelines with Jenkins.
- Implement GitOps using ArgoCD.
- Monitor using Prometheus & Grafana.
