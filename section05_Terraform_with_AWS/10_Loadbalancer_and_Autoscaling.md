**AWS Auto Scaling and Load Balancers**

## Introduction
This lecture covers **Auto Scaling in AWS**, and now extends into **Load Balancing** to efficiently distribute traffic across multiple instances. Load Balancers in AWS enhance **availability**, **scalability**, and **security**.

### Resources
For reference, the Terraform code for this demo can be found in the `demo-16` folder.

---

## **1. Understanding AWS Auto Scaling**
AWS Auto Scaling enables you to automatically **add or remove EC2 instances** based on defined policies. This helps to: 
- Maintain **optimal performance** during peak periods.
- **Reduce costs** by de-provisioning instances when they are not needed.
- **Improve availability** by ensuring instances are automatically replaced in case of failure.

### **1.1 Key Components**
To configure Auto Scaling, you need to set up **two main resources**:

#### **1.1.1 AWS Launch Configuration**
Defines the instance specifications:
- **AMI ID**: The image that will be used to launch instances.
- **Instance Type**: Specifies compute resources (e.g., `t2.micro`).
- **Security Groups**: Defines network rules.
- **Key Pair**: For SSH access.

#### **1.1.2 Auto Scaling Group (ASG)**
Manages the **actual scaling** of instances:
- **Minimum and Maximum Instances**: Defines the range of instances in the group.
- **VPC Subnets**: Specifies which subnets to launch instances in.
- **Health Checks**: Ensures instances remain operational.

---

## **2. Understanding Load Balancers in AWS**

### **2.1 Purpose of Load Balancers**
Once your Auto Scaling Group provisions instances, you need a **Load Balancer** to distribute incoming traffic across multiple EC2 instances efficiently. AWS provides **Elastic Load Balancer (ELB)** as a managed service to handle this.

**Benefits of ELB:**
- **Traffic Distribution:** Automatically routes incoming traffic to healthy EC2 instances.
- **Scalability:** ELB scales automatically with incoming traffic.
- **Health Checks:** It continuously monitors instances and avoids sending traffic to failed instances.
- **SSL Termination:** Offloads SSL/TLS encryption from EC2 instances, reducing CPU load.
- **High Availability:** Supports multiple Availability Zones (AZs) to enhance reliability.

### **2.2 Types of AWS Load Balancers**
AWS offers two primary types of Load Balancers:

#### **2.2.1 Classic Load Balancer (CLB)**
- Operates at the **transport (Layer 4) and application (Layer 7) levels**.
- Routes traffic based on basic network information (IP, ports).
- Example: Routes **HTTP (port 80)** traffic to **EC2 instances running on port 8080**.

#### **2.2.2 Application Load Balancer (ALB)**
- Operates at the **application layer (Layer 7)**.
- Can route traffic based on HTTP paths.
- Example: Requests to `example.com/api` go to one EC2 instance, while `example.com/website` goes to another.
- Provides advanced routing rules that are not possible with CLB.

---

## **3. Setting Up Load Balancers in Terraform**

### **3.1 Creating an Application Load Balancer (ALB)**
```hcl
resource "aws_lb" "example-alb" {
  name               = "example-load-balancer"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = [aws_subnet.main-public-1.id, aws_subnet.main-public-2.id]
}
```

### **3.2 Defining a Target Group**
```hcl
resource "aws_lb_target_group" "example-tg" {
  name     = "example-target-group"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
}
```

### **3.3 Creating a Listener Rule for HTTP Traffic**
```hcl
resource "aws_lb_listener" "http_listener" {
  load_balancer_arn = aws_lb.example-alb.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.example-tg.arn
  }
}
```

### **3.4 Registering Instances to the Target Group**
```hcl
resource "aws_autoscaling_attachment" "example-attachment" {
  autoscaling_group_name = aws_autoscaling_group.example-autoscaling.name
  lb_target_group_arn    = aws_lb_target_group.example-tg.arn
}
```

---

## **4. Deploying Auto Scaling with Load Balancers**

1. **Initialize Terraform:**
   ```bash
   terraform init
   ```
2. **Apply the Terraform Configuration:**
   ```bash
   terraform apply
   ```
3. **Verify in AWS Console:**
   - Check **Load Balancers**.
   - View **Auto Scaling Groups**.
   - Validate **Target Group Registration**.

4. **Test Load Balancer by Sending Requests:**
   ```bash
   curl http://<load_balancer_dns>
   ```

5. **Destroy Resources After Testing:**
   ```bash
   terraform destroy
   ```

---

## **5. Summary**
- **AWS Load Balancer (ELB)** ensures efficient traffic distribution.
- **Application Load Balancer (ALB)** offers advanced routing based on HTTP paths.
- **Load Balancers enhance fault tolerance and performance.**
- **Terraform automates ALB setup and integrates with Auto Scaling Groups.**



