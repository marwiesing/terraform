# **Running Docker Images on ECS**  
*Scaling containerized applications with AWS ECS and Terraform*  

## **Resources**  
- [Amazon ECS Documentation](https://docs.aws.amazon.com/ecs/)  
- [Amazon EC2 Container Service (ECS) Overview](https://aws.amazon.com/ecs/)  
- [Amazon Elastic Load Balancer (ELB)](https://aws.amazon.com/elasticloadbalancing/)  
- [Terraform AWS ECS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_cluster)  
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)  

---

## **1. Introduction to Amazon ECS (Elastic Container Service)**  

Amazon **ECS (Elastic Container Service)** is a fully managed container orchestration service that enables you to run, manage, and scale Docker containers on AWS. It provides two launch types:  

1. **ECS-EC2 (Traditional ECS on EC2 instances)**  
   - Uses EC2 instances as container hosts.  
   - Requires a **custom Amazon Machine Image (AMI)** with the **ECS agent** installed.  
   - Allows fine-grained control over instances and networking.  

2. **AWS Fargate (Serverless Container Execution)**  
   - Runs containers **without managing EC2 instances**.  
   - AWS provisions and manages the infrastructure.  
   - Recommended for **serverless, auto-scaling workloads**.  

### **Why Use ECS?**  
✅ Simplifies container management with built-in orchestration.  
✅ Fully integrated with AWS services like ECR, ELB, IAM, and CloudWatch.  
✅ Supports **autoscaling** to adjust container instances dynamically.  
✅ Works with **Terraform** to automate infrastructure deployment.  

---

## **2. Setting Up an ECS Cluster with Terraform**  

An **ECS Cluster** is a logical grouping of EC2 instances that run Docker containers. The **ECS Agent** installed on each instance communicates with AWS to manage tasks and services.  

### **Step 1: Define the ECS Cluster in Terraform**  

#### **ecs-cluster.tf**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_ecs_cluster" "example_cluster" {
  name = "example-cluster"
}
```
- This creates an **ECS Cluster** named `example-cluster`.  

---

### **Step 2: Launch EC2 Instances with ECS Agent**  

ECS runs on **EC2 instances**, which must:  
- Use an **Amazon ECS-optimized AMI** (Amazon Machine Image).  
- Include **IAM permissions** to pull images from ECR and interact with ECS.  
- Join the ECS Cluster on startup.  

#### **launch-config.tf**
```hcl
resource "aws_launch_configuration" "ecs_launchconfig" {
  name          = "ecs-launchconfig"
  image_id      = "ami-xxxxxxxx"  # Replace with the latest ECS-optimized AMI
  instance_type = "t2.micro"
  key_name      = "my-keypair"
  iam_instance_profile = aws_iam_instance_profile.ecs_profile.name
  security_groups      = [aws_security_group.ecs_sg.id]

  user_data = <<EOF
#!/bin/bash
echo ECS_CLUSTER=example-cluster >> /etc/ecs/ecs.config
EOF

  lifecycle {
    create_before_destroy = true
  }
}
```
- **`image_id`**: Uses the latest ECS AMI (get from AWS documentation).  
- **`user_data`**: Configures the instance to join the **example-cluster**.  
- **`create_before_destroy = true`**: Ensures smooth updates.  

---

### **Step 3: Create an Auto Scaling Group for ECS**  

To ensure high availability, ECS instances should be **automatically replaced** if they fail.  

#### **autoscaling.tf**
```hcl
resource "aws_autoscaling_group" "ecs_asg" {
  name                = "ecs-asg"
  launch_configuration = aws_launch_configuration.ecs_launchconfig.id
  min_size            = 1
  max_size            = 3
  desired_capacity    = 2
  vpc_zone_identifier = [aws_subnet.public_subnet.id]
}
```
- **`desired_capacity = 2`**: Starts with **two ECS instances**.  
- **`max_size = 3`**: Can scale up to **three instances**.  

---

### **Step 4: IAM Role for ECS EC2 Instances**  

EC2 instances running ECS must have an **IAM Role** that allows them to:  
- **Register with ECS**.  
- **Pull images from ECR**.  
- **Write logs to CloudWatch**.  

#### **iam.tf**
```hcl
resource "aws_iam_role" "ecs_role" {
  name = "ecsRole"
  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole",
      Effect = "Allow",
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}

resource "aws_iam_policy_attachment" "ecs_policy" {
  name       = "ecsPolicyAttachment"
  roles      = [aws_iam_role.ecs_role.name]
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"
}
```

---

## **3. Defining an ECS Task and Service**  

A **task definition** specifies:  
- The **Docker image** (from ECR).  
- **Memory and CPU limits**.  
- **Environment variables**.  

#### **ecs-task-definition.tf**
```hcl
resource "aws_ecs_task_definition" "myapp_task" {
  family = "myapp"
  container_definitions = jsonencode([
    {
      name      = "myapp"
      image     = "${aws_ecr_repository.myapp.repository_url}:latest"
      memory    = 256
      cpu       = 256
      essential = true
      portMappings = [{
        containerPort = 3000
        hostPort      = 3000
      }]
    }
  ])
}
```

---

### **Step 2: Running a Service Based on the Task Definition**  

An **ECS service** ensures the application is always running.  

#### **ecs-service.tf**
```hcl
resource "aws_ecs_service" "myapp_service" {
  name            = "myapp-service"
  cluster         = aws_ecs_cluster.example_cluster.id
  task_definition = aws_ecs_task_definition.myapp_task.arn
  desired_count   = 2

  load_balancer {
    target_group_arn = aws_lb_target_group.myapp_tg.arn
    container_name   = "myapp"
    container_port   = 3000
  }

  depends_on = [aws_lb_listener.myapp_listener]
}
```
- **`desired_count = 2`**: Runs **two instances** of the container.  
- **`load_balancer`**: Connects to an **Elastic Load Balancer (ELB)** for **high availability**.  

---

## **4. Load Balancing and High Availability**  

To distribute traffic, we use an **Application Load Balancer (ALB)**.  

#### **ecs-loadbalancer.tf**
```hcl
resource "aws_lb" "myapp_lb" {
  name               = "myapp-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.lb_sg.id]
  subnets           = [aws_subnet.public_subnet.id]
}

resource "aws_lb_listener" "myapp_listener" {
  load_balancer_arn = aws_lb.myapp_lb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.myapp_tg.arn
  }
}
```
- The **ALB distributes traffic** across running containers.  
- If a container fails, the ALB stops routing traffic to it.  

---

## **5. Summary and Next Steps**  

### **Key Takeaways:**  
✅ **ECS runs Docker containers at scale** on EC2 instances.  
✅ **Task Definitions specify how containers run** (CPU, memory, ports).  
✅ **ECS Services ensure high availability** by restarting failed containers.  
✅ **ALB provides load balancing**, ensuring uninterrupted service.  
✅ **Terraform automates ECS deployments**, making scaling seamless.  

### **Next Steps:**  
- Deploy the ECS **service and test load balancing**.  
- Implement **AWS Fargate** for **serverless container execution**.  
- Set up **CI/CD pipelines** to **automate deployments**.  
