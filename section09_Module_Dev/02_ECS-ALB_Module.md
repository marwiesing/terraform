# **Terraform Module Development: ECS Architecture Overview**  

## **Resources**  
- [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/latest/developerguide/Welcome.html) – Official AWS documentation on ECS  
- [Terraform AWS ECS Module](https://github.com/terraform-aws-modules/terraform-aws-ecs) – Community-maintained Terraform module for ECS  
- [AWS ALB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) – Introduction to AWS Application Load Balancer (ALB)  
- [Terraform AWS ALB Module](https://github.com/terraform-aws-modules/terraform-aws-alb) – Terraform module to provision an ALB  

---

## **ECS Architecture Overview**  ![ECS & ALB module](image.png)

To better understand the components we will be creating, let’s analyze the **ECS Cluster architecture** using the diagram provided.  

### **1️⃣ ECS Cluster (Compute Layer)**  

The **ECS Cluster** is the foundation of the architecture. It provides the compute resources required to run containerized applications.  

#### **Components of an ECS Cluster**  
| Component | Description |
|-----------|-------------|
| **ECS Resource** | Defines the ECS Cluster, enabling task execution. |
| **CloudWatch Logs** | Captures logs from running containers to facilitate monitoring and troubleshooting. |
| **IAM Roles** | Manages permissions for EC2 instances and ECS tasks (containers) to securely access AWS services. |
| **Security Groups** | Controls inbound and outbound traffic to ECS instances. |
| **Auto Scaling Group** | Dynamically adjusts the number of EC2 instances based on demand and health checks. |

> **Example Use Case**:  
> - The Auto Scaling Group ensures high availability by restarting failed EC2 instances or adding new instances when demand increases.  
> - IAM Roles grant permission for ECS tasks to access AWS resources like S3, DynamoDB, or RDS.  

---

### **2️⃣ ECS Service (Container Layer)**  

An **ECS Service** manages and runs containerized applications within an ECS Cluster. It ensures the desired number of running instances (tasks) for a given service.  

#### **Components of ECS Service**  
| Component | Description |
|-----------|-------------|
| **ECS Service Resource** | Defines the ECS Service that runs Docker containers within the ECS Cluster. |
| **ECS Task Definition** | A JSON template that specifies container settings, including CPU/memory limits, logging, and network configurations. |
| **CloudWatch Log Group** | Stores application logs from running containers for monitoring and debugging. |
| **ALB Target Group** | Registers running ECS tasks (containers) to allow traffic routing via the ALB. |

> **Example Use Case**:  
> - If an ECS task stops unexpectedly, the ECS Service will automatically restart it.  
> - The ECS Service template ensures logs are forwarded to CloudWatch for centralized monitoring.  

---

### **3️⃣ Application Load Balancer (ALB) (Traffic Management Layer)**  

The **Application Load Balancer (ALB)** is used to distribute traffic to the appropriate ECS services. It dynamically routes requests to healthy backend services.  

#### **Components of ALB**  
| Component | Description |
|-----------|-------------|
| **ALB Resource** | Creates the actual load balancer to handle traffic. |
| **ALB Security Group** | Controls traffic access to the load balancer. |
| **ALB Target Group** | Registers ECS tasks as targets for incoming requests. |
| **HTTP/HTTPS Listeners** | Listens on **Port 80 (HTTP)** and **Port 443 (HTTPS)** for incoming requests. |
| **ACM Certificate** | Manages SSL/TLS certificates to secure HTTPS connections. |

> **Example Use Case**:  
> - The ALB distributes traffic among multiple ECS tasks for load balancing and high availability.  
> - The ALB Security Group allows traffic only from trusted sources, ensuring network security.  

---

### **4️⃣ ALB Rules (Routing Layer)**  

The ALB uses **routing rules** to determine how incoming traffic is forwarded to ECS services.  

#### **Types of ALB Rules**  
| Rule Type | Description |
|-----------|-------------|
| **Path-Based Routing** | Routes traffic based on request paths (e.g., `/api` → ECS Service A, `/admin` → ECS Service B). |
| **Host-Based Routing** | Routes traffic based on domain names (e.g., `api.example.com` → ECS Service A, `admin.example.com` → ECS Service B). |
| **Default Target** | If no rule matches, traffic is sent to a default ECS Service. |

> **Example Use Case**:  
> - API traffic (`api.example.com`) is routed to a backend ECS service handling API requests.  
> - Web traffic (`example.com`) is routed to the frontend ECS service.  

---

## **Terraform Implementation Plan**  

To achieve this setup, we will **modularize** the infrastructure by creating **four Terraform modules**:  

1️⃣ **ECS Cluster Module**  
   - Defines the ECS Cluster and Auto Scaling configuration.  

2️⃣ **ECS Service Module**  
   - Specifies the Docker container, networking, and IAM policies.  

3️⃣ **ALB Module**  
   - Configures the Application Load Balancer, target groups, and security rules.  

4️⃣ **ALB Rule Module**  
   - Implements path-based or host-based routing rules.  

By structuring our Terraform code into **reusable modules**, we ensure:  
✅ **Scalability** – Easily replicate services across environments.  
✅ **Maintainability** – Simplified updates and management.  
✅ **Security** – Consistent application of best practices.  

