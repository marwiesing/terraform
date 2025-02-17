Here's an enhanced and structured version of the **AWS EKS Demo**, including additional context, explanations, and references.  

---

# **AWS EKS Demo: Deploying a Kubernetes Cluster with Terraform**  

## **Overview**  

In this demo, we will create an **Amazon Elastic Kubernetes Service (EKS) cluster** using **Terraform**, AWS's infrastructure as code (IaC) tool. We will walk through the necessary steps to configure AWS credentials, set up networking components, deploy worker nodes, and authenticate with the cluster using `kubectl`.  

📌 **References:**  
- [Terraform AWS EKS Module](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)  
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)  
- [Kubernetes Official Docs](https://kubernetes.io/docs/home/)  

---

## **1️⃣ Prerequisites**  

### **AWS Credentials Setup**  
Before deploying the EKS cluster, you need to configure your **AWS CLI credentials**:  

```bash
aws configure
```
Ensure that the IAM user has **admin privileges** or sufficient permissions to create EKS and related AWS resources.  

🔹 **IAM Policies Required:**  
- `AmazonEKSClusterPolicy`  
- `AmazonEKSServicePolicy`  
- `AmazonEKSWorkerNodePolicy`  
- `AmazonEC2ContainerRegistryReadOnly` (if pulling from ECR)  

---

## **2️⃣ Running Terraform to Create EKS Cluster**  

### **Step 1: Navigate to the Terraform Project Directory**  

```bash
cd ~/udemy/terraform/terraform-course/eks-demo
```

### **Step 2: Initialize Terraform**  

```bash
terraform init
```

### **Step 3: Apply the Terraform Configuration**  

```bash
terraform apply
```
📌 **⚠️ Important:** Running an EKS cluster costs **$0.20/hour**, plus **EC2 instance pricing** for worker nodes. **Make sure to destroy the cluster when not in use.**  

Terraform will create **36 resources**, including:  
✔ **VPC & Subnets** (for cluster networking)  
✔ **IAM Roles & Security Groups** (for authentication and authorization)  
✔ **EKS Control Plane** (fully managed by AWS)  
✔ **Worker Nodes (EC2 instances)** (for running containerized applications)  

---

## **3️⃣ Reviewing Terraform Configuration**  

### **Networking & VPC Setup (`vpc.tf`)**  

The `vpc.tf` file defines a VPC using the Terraform AWS VPC module. It also provisions **public and private subnets** across three Availability Zones:  

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  name    = "terraform-eks-vpc"
  cidr    = "10.0.0.0/16"
  azs     = slice(data.aws_availability_zones.available.names, 0, 3)
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
}
```

🔹 **Key Notes:**  
- A **multi-AZ setup** improves availability.  
- Uses **public subnets** for worker nodes.  
- No **NAT gateway** is provisioned to save costs.  

---

### **EKS Cluster Configuration (`eks-cluster.tf`)**  

Defines the **EKS cluster**, specifying its IAM role and VPC networking details:  

```hcl
resource "aws_eks_cluster" "demo" {
  name     = var.cluster-name
  role_arn = aws_iam_role.demo-cluster.arn

  vpc_config {
    security_group_ids = [aws_security_group.demo-cluster.id]
    subnet_ids         = module.vpc.public_subnets
  }

  depends_on = [
    aws_iam_role_policy_attachment.demo-cluster-AmazonEKSClusterPolicy,
    aws_iam_role_policy_attachment.demo-cluster-AmazonEKSServicePolicy,
  ]
}
```

🔹 **Key Notes:**  
- Uses the **Terraform EKS module** for ease of setup.  
- The **IAM role** grants the cluster permissions to interact with AWS services.  

---

### **Worker Nodes Configuration (`eks-workers.tf`)**  

Worker nodes are deployed using an **Auto Scaling Group** (ASG) and an **EC2 Launch Configuration**:  

```hcl
resource "aws_launch_configuration" "demo" {
  iam_instance_profile = aws_iam_instance_profile.demo-node.name
  image_id             = data.aws_ami.eks-worker.id
  instance_type        = "t2.medium"
  security_groups      = [aws_security_group.demo-node.id]
  user_data_base64     = base64encode(local.demo-node-userdata)
}
```

🔹 **Key Notes:**  
- Uses **Amazon EKS-optimized AMI** for worker nodes.  
- Uses an **Auto Scaling Group** to dynamically scale instances.  
- Worker nodes are deployed in **public subnets**.  

---

### **IAM Configuration (`iam.tf`, `iam-workers.tf`)**  

Worker nodes need IAM roles to interact with the EKS cluster:  

```hcl
resource "aws_iam_role_policy_attachment" "demo-node-AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.demo-node.name
}
```

🔹 **IAM Policies Assigned:**  
- **EKS Worker Node Policy** (allows communication with control plane)  
- **EKS CNI Policy** (for networking)  
- **EC2 Container Registry Read-Only** (if pulling images from ECR)  

---

## **4️⃣ Connecting to the EKS Cluster**  

Once Terraform has successfully applied the configuration, obtain cluster credentials:  

### **Step 1: Configure `kubectl`**  

```bash
aws eks --region us-east-1 update-kubeconfig --name terraform-eks-demo
```

This writes the cluster information to `~/.kube/config` so `kubectl` can interact with it.

### **Step 2: Test the Cluster Connection**  

```bash
kubectl get nodes
```
✔ If everything is working, you should see worker nodes in **Ready** state.  

---

## **5️⃣ Deploying an Application on EKS**  

### **Step 1: Deploy a Sample Application**  

```bash
kubectl run hello-world --image=gcr.io/google-samples/hello-app:1.0 --port=8080
```

### **Step 2: Expose the Application with a Load Balancer**  

```bash
kubectl expose deployment hello-world --type=LoadBalancer --port=8080
```
✔ This creates an **ELB (Elastic Load Balancer)** that routes external traffic to your app.  

### **Step 3: Get the Load Balancer URL**  

```bash
kubectl get services hello-world
```
Once the external IP is available, test the deployment:  

```bash
curl http://<LOAD_BALANCER_URL>:8080
```

✔ You should receive a response from the Hello World application.  

---

## **6️⃣ Cleaning Up the Resources**  

**⚠️ Don't forget to destroy resources to avoid unexpected AWS charges!**  

```bash
terraform destroy
```
✔ This **removes** all AWS resources, including **VPC, IAM roles, EKS cluster, and worker nodes**.  

---

## **Conclusion**  

✅ AWS EKS provides a **fully managed Kubernetes control plane**, reducing operational overhead.  
✅ Using **Terraform** automates EKS provisioning and infrastructure management.  
✅ The demo showed how to **deploy a Kubernetes application** on EKS using `kubectl`.  

📌 **Further Learning:**  
- [AWS EKS Workshop](https://www.eksworkshop.com/)  
- [Terraform AWS EKS Module](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)  
- [Kubernetes Official Documentation](https://kubernetes.io/docs/home/)  

---

This enhanced version **clarifies steps, adds explanations, references best practices, and modernizes authentication methods**. Let me know if you need additional refinements! 🚀