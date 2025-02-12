# **Demo: Running Docker Images on AWS ECS with Terraform**  
*Deploying a Node.js application on ECS with an ALB for high availability*  

## **Resources**  
- [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)  
- [AWS Elastic Load Balancing (ELB)](https://aws.amazon.com/elasticloadbalancing/)  
- [Terraform AWS ECS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_cluster)  
- [Amazon ECR (Elastic Container Registry)](https://aws.amazon.com/ecr/)  

---

## **1. Overview of the Demo**  

### **Objective**  
In this demo, we will:  
✅ Deploy a **Dockerized Node.js application** on **AWS ECS** using Terraform.  
✅ Use **ECR (Elastic Container Registry)** to store and pull the Docker image.  
✅ Set up an **ECS cluster** using EC2 instances with the **ECS agent installed**.  
✅ Configure an **Application Load Balancer (ALB)** to route traffic to the running containers.  
✅ Demonstrate **scaling ECS services** by modifying the Terraform configuration.  

### **Setup**  
All necessary Terraform files are stored in **Docker Demo 2**, and they define:  
- **ECR Repository (`ecr.tf`)** – Stores Docker images.  
- **ECS Cluster (`ecs.tf`)** – Manages containerized applications.  
- **IAM Policies (`iam.tf`)** – Grants ECS instances access to ECR and logs.  
- **Security Groups (`securitygroup.tf`)** – Controls access to the cluster and load balancer.  
- **Application Deployment (`myapp.tf`)** – Defines task definition, service, and load balancer.  
- **VPC & Subnet Configuration (`vpc.tf`)** – Sets up networking for ECS.  

---

## **2. Running Terraform to Deploy ECS**  

### **Step 1: Initialize Terraform**
```sh
terraform init
```
This initializes Terraform and downloads the required AWS provider plugins.

### **Step 2: Apply Terraform Configuration**
```sh
terraform apply -auto-approve
```
- This creates the **ECR repository, ECS cluster, IAM roles, security groups, ALB, and ECS service**.  
- Terraform outputs the **ELB DNS name**, which we will use to access the application.  

⏳ **Note:** Even though Terraform completes successfully, the ECS agent may take a few minutes to launch the container.

---

## **3. Verifying the Deployment**  

### **Step 1: Checking ECS Cluster Status**  
```sh
aws ecs list-clusters
aws ecs describe-clusters --cluster example-cluster
```
- Ensure that the **ECS agent** is running and that the cluster is active.  

### **Step 2: SSH into an ECS Instance**  
```sh
ssh -i mykey.pem ec2-user@<EC2_INSTANCE_IP>
```
- This logs into the ECS host instance.  
- Run the following to check running containers:  
  ```sh
  docker ps
  ```
  - The output should show the **ECS agent** and the **Node.js application container**.

### **Step 3: Testing Locally on the ECS Host**  
```sh
curl localhost:3000
```
- Expected Output: `Hello, World!`  
- This confirms that the **Node.js app is running inside the Docker container**.

---

## **4. Accessing the Application via Load Balancer**  

### **Step 1: Get the ELB DNS Name**  
Terraform outputs the **ELB DNS name**, which can also be retrieved using:  
```sh
aws elb describe-load-balancers --query "LoadBalancerDescriptions[*].DNSName"
```

### **Step 2: Test Application via Load Balancer**  
```sh
curl http://<ELB_DNS_NAME>
```
- Expected Output: `Hello, World!`  
- The **load balancer routes traffic** to the ECS container running the Node.js app.

---

## **5. Scaling the Application**  

### **Step 1: Modify Terraform Configuration**  
To scale the **ECS service**, update **`ecs.tf`**:
```hcl
resource "aws_ecs_service" "myapp-service" {
  name            = "myapp"
  cluster         = aws_ecs_cluster.example-cluster.id
  task_definition = aws_ecs_task_definition.myapp-task-definition.arn
  desired_count   = 2  # Change from 1 to 2 for scaling
}
```

### **Step 2: Apply Changes**  
```sh
terraform apply -auto-approve
```
- This scales the **application from 1 to 2 containers**.  
- The ALB will now distribute requests across both instances.

### **Step 3: Verify Scaling**  
```sh
aws ecs list-tasks --cluster example-cluster
```
- Should show **two running tasks** instead of one.

---

## **6. Terraform Code Breakdown**  

### **ECS Cluster Definition (`ecs.tf`)**
```hcl
resource "aws_ecs_cluster" "example-cluster" {
  name = "example-cluster"
}
```
- Creates an **ECS cluster** to host our containers.

### **Launch Template for EC2 Instances (`ecs.tf`)**
```hcl
resource "aws_launch_template" "ecs-example-launchconfig" {
  name          = "ecs-launchconfig"
  image_id      = var.ECS_AMIS[var.AWS_REGION]
  instance_type = var.ECS_INSTANCE_TYPE
  key_name      = aws_key_pair.mykeypair.key_name
  user_data = base64encode("#!/bin/bash\necho 'ECS_CLUSTER=example-cluster' > /etc/ecs/ecs.config\nstart ecs")
}
```
- Launches EC2 instances **pre-configured** to join ECS.

### **IAM Role for ECS Instances (`iam.tf`)**
```hcl
resource "aws_iam_role" "ecs-ec2-role" {
  name = "ecs-ec2-role"
  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole",
      Effect = "Allow",
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}
```
- Grants **ECS permissions** to EC2 instances.

### **ECS Task Definition (`myapp.tf`)**
```hcl
resource "aws_ecs_task_definition" "myapp-task-definition" {
  family                = "myapp"
  container_definitions = templatefile("templates/app.json.tpl", {
    REPOSITORY_URL = replace(aws_ecr_repository.myapp.repository_url, "https://", "")
  })
}
```
- Specifies **CPU, memory, ports, and the Docker image**.

### **Application Load Balancer (`myapp.tf`)**
```hcl
resource "aws_elb" "myapp-elb" {
  name               = "myapp-elb"
  listener {
    instance_port     = 3000
    instance_protocol = "http"
    lb_port           = 80
    lb_protocol       = "http"
  }
}
```
- Routes external requests to ECS containers.

---

## **7. Summary and Next Steps**  

### **Key Takeaways**  
✅ **ECS runs Docker containers** on EC2 instances.  
✅ **ECR stores Docker images** for easy deployment.  
✅ **Terraform automates infrastructure setup** for ECS and ALB.  
✅ **Load Balancing ensures high availability** for the application.  
✅ **Scaling is easy** with Terraform by modifying `desired_count`.  


