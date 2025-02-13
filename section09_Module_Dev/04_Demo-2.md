# **Terraform Module Development: ALB & ALB Rules**  

## **Resources**  
- [In4IT Terraform Modules Repository](https://github.com/in4it/terraform-modules) – Contains ALB and ECS Terraform modules  
- [AWS Application Load Balancer (ALB) Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) – Official AWS documentation for ALB  
- [Terraform AWS ALB Module](https://github.com/terraform-aws-modules/terraform-aws-alb) – Community-maintained Terraform module for ALB  
- [AWS ACM (Certificate Manager)](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html) – Used for SSL/TLS certificates in ALB  

---

## **1️⃣ Introduction to ALB in Terraform**  

In this section, we will deploy an **Application Load Balancer (ALB)** that serves as a traffic distribution layer for our ECS services.  

### **ALB Features in Our Setup**  
✅ Routes incoming traffic to ECS containers  
✅ Supports **HTTP and HTTPS listeners**  
✅ Uses **SSL/TLS encryption** via AWS Certificate Manager (ACM)  
✅ Allows **host-based and path-based routing**  

---

## **2️⃣ Defining the ALB Module in Terraform**  

The **ALB module** is referenced in `alb.tf`:  

```hcl
module "my-alb" {
  source  = "github.com/in4it/terraform-modules//alb?ref=1.0.0"

  vpc_id          = module.vpc.vpc_id
  name            = "ecs-alb"
  subnets         = join(",", module.vpc.public_subnets)
  default_target_arn = module.my-service.target_group_arn
  domain_name     = "ecs.newtech.academy"
  internal        = false
  security_group  = module.sg.alb_sg
}
```

### **Key Parameters**  
| Parameter | Description |
|-----------|-------------|
| **`vpc_id`** | Specifies which VPC the ALB should be deployed in. |
| **`name`** | The name of the ALB. |
| **`subnets`** | Specifies the subnets where the ALB should be launched. |
| **`default_target_arn`** | Defines the default backend service. |
| **`domain_name`** | The domain name used for SSL encryption. |
| **`internal`** | Boolean flag to specify whether this is an internal or external ALB. |
| **`security_group`** | Defines the security group attached to the ALB. |

---

## **3️⃣ ALB Listeners & SSL Certificate**  

### **HTTPS Listener with ACM Certificate**  

AWS ALB requires an **SSL certificate** for HTTPS traffic. **Terraform cannot create an ACM certificate**, so we must manually generate one in AWS ACM **before running `terraform apply`**.  

📌 **Steps to Create an ACM Certificate (Manual Process)**  
1️⃣ Go to the **AWS Certificate Manager (ACM)**.  
2️⃣ Request a **public certificate** for your domain (e.g., `ecs.newtech.academy`).  
3️⃣ Validate the certificate using **DNS or email verification**.  
4️⃣ Retrieve the **certificate ARN** and pass it to Terraform.  

```hcl
resource "aws_acm_certificate" "alb_cert" {
  domain_name       = "ecs.newtech.academy"
  validation_method = "DNS"
}
```

### **Defining HTTPS & HTTP Listeners**  

The ALB needs **two listeners**:  

1️⃣ **HTTPS Listener (Port 443, Secure)**  
```hcl
resource "aws_lb_listener" "https_listener" {
  load_balancer_arn = module.my-alb.alb_arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-2016-08"
  certificate_arn   = aws_acm_certificate.alb_cert.arn

  default_action {
    type             = "forward"
    target_group_arn = module.my-service.target_group_arn
  }
}
```
✅ Uses **SSL/TLS encryption** for secure traffic  
✅ Defaults to forwarding requests to the ECS service  

2️⃣ **HTTP Listener (Port 80, Redirects to HTTPS)**  
```hcl
resource "aws_lb_listener" "http_listener" {
  load_balancer_arn = module.my-alb.alb_arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type = "redirect"

    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "301"
    }
  }
}
```
✅ Redirects **HTTP traffic to HTTPS**  
✅ Ensures **secure communication** with clients  

---

## **4️⃣ ALB Security Group & Dynamic Port Handling**  

The ALB must be allowed to **communicate with ECS instances**. Since ECS containers use **random ports**, the security group must be configured to allow traffic in a specific range.  

### **ALB Security Group**  

```hcl
resource "aws_security_group" "alb_sg" {
  vpc_id = module.vpc.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
✅ **Allows public access on ports 80 & 443**  

### **ALB to ECS Security Group Rules**  

ECS assigns **random ports** to containers, so the ALB must allow **dynamic port ranges**:  

```hcl
resource "aws_security_group_rule" "alb_to_ecs" {
  security_group_id        = aws_security_group.alb_sg.id
  source_security_group_id = module.my-ecs.security_group_id
  from_port                = 32768
  to_port                  = 61000
  protocol                 = "tcp"
  type                     = "ingress"
}
```
✅ **Allows ALB to connect to ECS instances on random ports**  
✅ **Ensures traffic reaches the correct container**  

---

## **5️⃣ ALB Output Variables**  

Once Terraform provisions the ALB, it outputs important values:  

```hcl
output "alb_dns_name" {
  value = module.my-alb.dns_name
}

output "alb_zone_id" {
  value = module.my-alb.zone_id
}
```

📌 **These values can be used in AWS Route 53** to create a custom domain name.  

---

## **6️⃣ Optional: ALB Rules for Routing Traffic**  

If you have **multiple services** behind an ALB, you can use **path-based or host-based routing**.  

### **Defining an ALB Rule in Terraform**  

```hcl
resource "aws_lb_listener_rule" "api_rule" {
  listener_arn = aws_lb_listener.https_listener.arn
  priority     = 100

  conditions {
    field  = "host-header"
    values = ["api.newtech.academy"]
  }

  actions {
    type             = "forward"
    target_group_arn = module.my-api-service.target_group_arn
  }
}
```

📌 **Routing Logic:**  
- If a request **matches `api.newtech.academy`**, forward it to **API service**.  
- If no rule matches, **default target** (`my-service`) is used.  

---

## **7️⃣ Conclusion & Next Steps**  

In this section, we:  
✅ **Configured an ALB and associated listeners**  
✅ **Integrated ACM SSL for HTTPS security**  
✅ **Allowed ECS containers to communicate using dynamic ports**  
✅ **Created optional ALB rules for traffic routing**  

