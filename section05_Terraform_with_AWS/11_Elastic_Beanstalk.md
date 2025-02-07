**AWS Auto Scaling, Load Balancers, and Elastic Beanstalk**

## Introduction
This lecture covers **Auto Scaling in AWS**, **Load Balancers**, and now extends into **Elastic Beanstalk**, AWS’s **Platform as a Service (PaaS)** solution. Elastic Beanstalk simplifies deployment and management while integrating with **EC2, RDS, Load Balancers, and Auto Scaling Groups**.

### Resources
For reference, the Terraform code for this demo can be found in the `demo-17` folder.

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
AWS provides **Elastic Load Balancer (ELB)** as a managed service to distribute traffic across instances. ELB helps with:
- **Traffic Distribution:** Routes incoming traffic to healthy EC2 instances.
- **Scalability:** Automatically scales to handle more traffic.
- **SSL Termination:** Manages SSL certificates and offloads encryption tasks.
- **High Availability:** Supports multiple **Availability Zones (AZs)**.

### **2.2 Types of AWS Load Balancers**
- **Classic Load Balancer (CLB)**: Routes traffic at the transport and application layers.
- **Application Load Balancer (ALB)**: Routes traffic based on HTTP paths (e.g., `/api` to one instance, `/web` to another).

---

## **3. AWS Elastic Beanstalk Overview**

### **3.1 What is Elastic Beanstalk?**
Elastic Beanstalk is **AWS’s PaaS** solution that simplifies deployment, scaling, and monitoring of web applications. It abstracts infrastructure management by integrating:
- **EC2 instances** for compute power.
- **RDS** for managed databases.
- **Load Balancers** for traffic distribution.
- **Auto Scaling Groups** for dynamic instance scaling.

### **3.2 Key Features**
- **Managed Infrastructure:** No need to provision EC2 instances manually.
- **Automatic Load Balancing & Scaling:** Handles traffic distribution and scaling.
- **Support for Multiple Languages:** PHP, Java, Python, Ruby, Node.js, Go, .NET, and Docker.
- **Application Environments:** Supports **Dev, Staging, and Production** environments.
- **Integration with AWS Services:** Works seamlessly with **CloudWatch, IAM, and S3**.

---

## **4. Deploying Elastic Beanstalk with Terraform**

### **4.1 Define Elastic Beanstalk Application**
```hcl
resource "aws_elastic_beanstalk_application" "app" {
  name        = "app"
  description = "app"
}
```

### **4.2 Define an Elastic Beanstalk Environment**
```hcl
resource "aws_elastic_beanstalk_environment" "app-prod" {
  name                = "app-prod"
  application         = aws_elastic_beanstalk_application.app.name
  solution_stack_name = "64bit Amazon Linux 2 running PHP 8.0"
  setting {
    namespace = "aws:ec2:vpc"
    name      = "VPCId"
    value     = aws_vpc.main.id
  }
  setting {
    namespace = "aws:autoscaling:launchconfiguration"
    name      = "InstanceType"
    value     = "t2.micro"
  }
}
```

### **4.3 Deploying the Application**
1. **Initialize Terraform:**
   ```bash
   terraform init
   ```
2. **Apply the Terraform Configuration:**
   ```bash
   terraform apply
   ```
3. **Get Elastic Beanstalk URL:**
   ```bash
   terraform output eb
   ```
4. **Deploy an Application via Elastic Beanstalk CLI:**
   ```bash
   eb init
   eb deploy
   ```

---

## **5. Managing Environments and Scaling**

### **5.1 Updating Your Environment**
- **Rolling Updates:** Update a percentage of instances at a time (e.g., 30%).
- **Health Checks:** Ensures updates do not break the application.
- **Blue-Green Deployments:** Create a separate environment for testing before switching.

### **5.2 Auto Scaling and Load Balancing**
Elastic Beanstalk automatically manages Auto Scaling Groups and Load Balancers:
- **Cross-Zone Load Balancing:** Ensures traffic reaches all AZs.
- **Minimum Instance Count:** Defines baseline capacity.
- **Scaling Policies:** Adjust instance count based on CPU or memory utilization.

---

## **6. Summary**
- **AWS Elastic Beanstalk** simplifies application deployment and scaling.
- **Terraform automates setup** of Beanstalk applications and environments.
- **Elastic Beanstalk integrates** with EC2, RDS, and Auto Scaling Groups.
- **CLI tools like `eb`** help with deployment and management.

