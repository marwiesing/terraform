# **Terraform Module Development: ECS & ALB Demo**  

## **Resources**  
- [In4IT Terraform Modules Repository](https://github.com/in4it/terraform-modules) – Contains ECS and ALB Terraform modules  
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) – Official Terraform AWS Provider documentation  
- [Terraform Modules Guide](https://developer.hashicorp.com/terraform/language/modules) – HashiCorp’s guide on creating and using modules  

---

## **Introduction to the Demo**  

In this demo, we will **clone, explore, and deploy** Terraform modules for **Amazon ECS and ALB**. These modules are stored in the **In4IT Terraform Modules Repository** and are structured to support modular and reusable infrastructure provisioning.  

We will cover:  
✅ **Cloning the repository and exploring the modules**  
✅ **Understanding module structure and dependencies**  
✅ **Deploying an ECS Cluster and configuring an ALB**  
✅ **Managing IAM, Security Groups, and Logging with CloudWatch**  
✅ **How ALB target groups route traffic to ECS services**  

---

## **1️⃣ Cloning the Repository & Exploring the Modules**  

First, we **clone the repository** that contains the Terraform modules:  

```sh
git clone https://github.com/in4it/terraform-modules.git
cd terraform-modules
```

This repository includes multiple modules:  

```
/modules
   ├── alb
   ├── alb-rule
   ├── ecs-cluster
   ├── ecs-service
/demo
   ├── ecs.tf  # Calls the modules
```

In the **demo folder**, we have a `ecs.tf` file that **orchestrates the modules**.  

The **four Terraform modules** we will be using:  
1️⃣ **ECS Cluster Module** – Creates an ECS cluster with EC2 instances  
2️⃣ **ECS Service Module** – Deploys a service with Docker containers  
3️⃣ **ALB Module** – Sets up the Application Load Balancer  
4️⃣ **ALB Rule Module** – Defines path-based and host-based routing  

---

## **2️⃣ Understanding the ECS Cluster Module**  

### **Module Declaration (`ecs.tf`)**  

The **ECS Cluster Module** is referenced in the `ecs.tf` file like this:  

```hcl
module "my-ecs" {
  source  = "github.com/in4it/terraform-modules//ecs-cluster?ref=1.0.0"

  vpc_id            = module.vpc.vpc_id
  cluster_name      = "my-ecs-cluster"
  instance_type     = "t2.small"
  key_name          = "my-key"
  vpc_subnets       = module.vpc.public_subnets
  enable_ssh        = true
  security_group    = module.sg.ecs_sg
  log_group         = "ecs-log-group"
  aws_account_id    = "123456789012"
  region            = "us-east-1"
}
```

### **Key Parameters**  
| Parameter | Description |
|-----------|-------------|
| **`vpc_id`** | Specifies which VPC the ECS cluster should be deployed in. |
| **`cluster_name`** | Defines the name of the ECS cluster. |
| **`instance_type`** | The EC2 instance type used for ECS. |
| **`key_name`** | The SSH key pair for accessing ECS instances. |
| **`security_group`** | The security group attached to the ECS cluster. |
| **`log_group`** | The CloudWatch log group for ECS logs. |
| **`region`** | AWS region for deployment. |

### **How the ECS Cluster is Created**  

The **ECS cluster module (`ecs.tf`)** does the following:  
1. Retrieves the **Amazon Machine Image (AMI)** for ECS-optimized instances.  
2. Configures an **Auto Scaling Group** that manages EC2 instances.  
3. Defines a **CloudWatch Log Group** for container logging.  
4. Creates **IAM roles** to grant ECS access to AWS services.  

📌 **Dependency Handling:**  
- The **ECS Cluster depends on the VPC** (`module.vpc.vpc_id`).  
- The **ECS Service depends on the ECS Cluster** (`module.my-ecs.cluster_arn`).  

---

## **3️⃣ IAM & Security Group Configuration**  

The ECS cluster needs **IAM roles and security groups** to function correctly.  

### **IAM Roles (`iam.tf`)**  

IAM roles grant permissions to ECS instances and tasks:  
```hcl
resource "aws_iam_role" "ecs_role" {
  name = "ecsRole"
  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "ecs.amazonaws.com"
      },
      "Effect": "Allow"
    }
  ]
}
EOF
}
```

📌 **Permissions Included:**  
- Allows ECS instances to **register tasks** and **submit logs** to CloudWatch.  
- Grants access to **start ECS services** on EC2.  

### **Security Group (`security_group.tf`)**  

The **security group** controls inbound/outbound traffic:  
```hcl
resource "aws_security_group" "ecs_sg" {
  name_prefix = "ecs-sg"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
✅ **Allows SSH traffic if enabled**  
✅ **Ensures controlled access to ECS instances**  

---

## **4️⃣ ALB & Target Group Configuration**  

### **ALB Module (`alb.tf`)**  

The **ALB module** sets up:  
✅ **Application Load Balancer**  
✅ **Security Group for ALB**  
✅ **ALB Target Group** (connects ECS services)  
✅ **Listeners for HTTP & HTTPS**  

```hcl
module "alb" {
  source  = "github.com/in4it/terraform-modules//alb?ref=1.0.0"

  vpc_id     = module.vpc.vpc_id
  name       = "ecs-alb"
  subnets    = module.vpc.public_subnets
  security_group = module.sg.alb_sg
}
```

📌 **Key Features:**  
- ALB **routes traffic** to ECS containers.  
- **Health checks** ensure only healthy containers receive traffic.  
- ALB Target Group registers running ECS tasks dynamically.  

---

## **5️⃣ ECS Service & Task Definition**  

### **ECS Service (`ecs-service.tf`)**  

The **ECS Service module** configures the Docker container:  
```hcl
module "my-service" {
  source  = "github.com/in4it/terraform-modules//ecs-service?ref=1.0.0"

  vpc_id        = module.vpc.vpc_id
  app_name      = "my-app"
  app_port      = 80
  app_version   = "latest"
  cluster_arn   = module.my-ecs.cluster_arn
  service_role  = module.my-ecs.service_role_arn
  desired_count = 2
  alb_arn       = module.alb.alb_arn
}
```

📌 **Features:**  
- Uses **random port assignment** (no manual mapping needed).  
- Connects the ECS service to the **ALB Target Group**.  
- Ensures logging to **CloudWatch**.  

---

## **6️⃣ Output Variables**  

Once deployed, Terraform outputs important values:  
```hcl
output "ecs_cluster_arn" {
  value = module.my-ecs.cluster_arn
}

output "alb_dns_name" {
  value = module.alb.dns_name
}
```
📌 **Cluster ARN** – Used to reference the ECS cluster in future configurations.  
📌 **ALB DNS Name** – Used to access the service via the load balancer.  

---

## **Conclusion & Next Steps**  

In this demo, we:  
✅ **Cloned and explored the Terraform modules**  
✅ **Configured ECS Cluster, IAM, and Security Groups**  
✅ **Deployed an ALB and ECS Service**  
✅ **Managed dependencies between resources**  

