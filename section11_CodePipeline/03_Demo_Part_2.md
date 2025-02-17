# AWS CodePipeline Demo - Part 2

## Load Balancer and ECS Cluster Configuration

### ECS Cluster and Load Balancer Setup
- The **ECS cluster** in this setup is called **Demo**.
- A **Network Load Balancer (NLB)** is used for routing traffic.
  - Unlike an **Application Load Balancer (ALB)**, an **NLB** does not have security groups and is optimized for **low latency traffic forwarding**.
  - The NLB operates at the transport layer (Layer 4) and routes packets to the target without inspecting the contents.
  - It is placed in the **public subnet** for accessibility.
  - **Cross-zone load balancing** is enabled since we only have one task running.
  
### Listener and Target Groups
- The **NLB listener** is set up on **port 80**.
- The **default action** is to forward requests to the **Demo Blue target group**.
- During a deployment:
  - **CodeDeploy switches from Demo Blue to Demo Green**.
  - Lifecycle management is set to **ignore default action changes** so that AWS and CodeDeploy can modify it dynamically.
- Two target groups are configured:
  - **Demo Blue** (initial active group)
  - **Demo Green** (used during deployments)
  - Both groups use **port 3000**, **TCP protocol**, and **IP-based targets** within the **VPC**.
  - **Health checks** are enabled to ensure service availability.

## AWS Fargate and ECS Service Configuration

### Fargate vs. EC2
- Instead of traditional **EC2 instances**, **AWS Fargate** is used.
- **Fargate runs ECS tasks in a serverless fashion** without managing EC2 instances.
- Fargate uses **micro VMs**, which provide faster boot times compared to standard EC2 VMs.
- This ensures workload separation while allowing Amazon to run multiple customer workloads on a single physical machine securely.

### ECS Task Definition
- The **task definition** includes:
  - **CPU and memory settings** (required for Fargate).
  - **Network mode: AWS VPC** (Each task gets its own IP within the VPC).
  - **Container definition:**
    - **Container name:** `demo`
    - **Logging:** CloudWatch logs enabled.
    - **Health check:**
      - **Network Load Balancer only supports TCP health checks**, but additional **HTTP health checks** are performed inside the container.
    - **Port mapping:** Routes application traffic correctly.

### ECS Service Configuration
- The ECS **service type** is **Fargate**.
- To integrate with **CodePipeline and CodeDeploy**, the **deployment controller type** is set to **CodeDeploy**.
- **Network settings:**
  - **Subnets and security groups** configured.
  - **Public IP is assigned** (for demo purposes, but a **private subnet with NAT Gateway** is recommended for production).
  - **If a private subnet is used, a NAT Gateway is required to pull container images from ECR.**
  
## Security Considerations
- The **Network Load Balancer does not use security groups**, but **Fargate containers require proper security settings**.
- For demo purposes:
  - **Public IP is assigned**.
  - **All traffic is allowed** (not ideal for production, as it allows bypassing the load balancer).
- **For production:**
  - Use **private IPs** and place tasks in a **private subnet**.
  - Configure **NAT Gateway** to allow outbound traffic for pulling images.
  - Restrict **ingress rules** to prevent direct access to Fargate containers.

## VPC Configuration
- The **VPC setup** includes:
  - **Private subnets**
  - **Public subnets**
  - **No NAT Gateway enabled by default** (reducing costs).
- To enable **NAT Gateway**, modify the **VPC configuration** and be aware of **increased costs**.

## IAM Policies
- **IAM roles and policies** are defined for:
  - **IAM for ECS, CodeBuild, CodeDeploy, and CodePipeline.**
  - **IAM for KMS encryption (for securing artifacts stored in S3).**
  - **IAM policies ensure least privilege access for each service.**
- Detailed configurations can be found in:
  - `iam-codebuild.tf`
  - `iam-codedeploy.tf`
  - `iam-codepipeline.tf`
  - `iam-ecs.tf`
  - `kms.tf`

## Summary
- This setup uses **AWS Fargate**, **ECS**, and **Network Load Balancer**.
- CodeDeploy enables **Blue-Green Deployments** with **automatic rollback**.
- **Security best practices** should be applied in production by **limiting public access and enabling NAT Gateway**.
- All Terraform configurations can be found in the **Terraform codebase** (`terraform-course/codepipeline-demo/`).

## Next Steps
In the next section, we will:
- Deploy the Terraform configuration.
- Execute a full **CI/CD pipeline** run.
- Monitor logs and analyze deployments using AWS Console.

By the end of this demo, you will have a fully functional AWS CodePipeline setup that automates containerized application deployment using AWS Fargate.

