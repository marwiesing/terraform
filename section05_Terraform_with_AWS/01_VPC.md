## **Terraform with AWS: Virtual Private Cloud (VPC)**
  
### **Resources**
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Terraform AWS Provider: VPC](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)
- [AWS CIDR Block Guide](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#VPC_Sizing)
- [AWS Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

---

## **Introduction to Terraform and AWS Networking**
  
### **Instructor's Overview**
In this section, we will explore how Terraform is used to provision infrastructure on AWS, starting with one of the most fundamental components: **Virtual Private Cloud (VPC)**. The goal is to help you understand:
- What a VPC is and why it is essential.
- How to create a VPC using Terraform.
- Best practices for structuring a network in AWS.
  
By the end of this module, you will be able to define a VPC and configure its core components, such as **subnets, internet gateways, and route tables**.

---

## **What is a VPC?**
### **Definition & Default Behavior**
A **Virtual Private Cloud (VPC)** is a logically isolated network in AWS where you can launch and manage your cloud resources securely. By default, AWS provides a **default VPC** in each region, which is automatically configured to allow easy deployment of EC2 instances.

- **Key Characteristics:**
  - **Network Isolation:** A VPC provides an isolated environment where instances can communicate securely.
  - **Customizable Networking:** You define IP address ranges, subnets, and routing rules.
  - **Public & Private Subnets:** Control which instances have internet access.
  - **Security:** Integration with **security groups** and **network ACLs**.

### **AWS Default VPC vs. Custom VPC**
- **Default VPC:**  
  - Automatically set up in each AWS region.
  - Comes with a **single public subnet**, an **internet gateway**, and **default route tables**.
  - All instances get a **public IP** automatically.
  
- **Custom VPC (Recommended for Production Use):**  
  - Manually created and controlled.
  - Allows for **public & private subnets**.
  - More secure and optimized for different workloads.

### **Comparison to Other Cloud Providers**
Unlike AWS, some providers like **DigitalOcean** do not have a VPC concept but instead use a **flat networking model** (like EC2-Classic). In such cases, strict firewall configurations are required to prevent unauthorized access.

---

## **Subnetting in AWS VPC**
### **CIDR Blocks and IP Addressing**![subnet masks](image.png)
A VPC is assigned an **IP range** using **CIDR notation** (Classless Inter-Domain Routing). AWS supports three major **private IP ranges**:
- **10.0.0.0/8** (Large networks, 16.7 million addresses)
- **172.16.0.0/12** (Medium networks, ~1 million addresses)
- **192.168.0.0/16** (Small networks, 65,536 addresses)

For example, defining a VPC with **CIDR block `10.0.0.0/16`** gives:
- **Subnet mask:** `255.255.0.0`
- **Available IPs:** `65,536`

👉 **Best Practice:** Use **/16 for a VPC** and divide it into smaller **/24 subnets**.

### **Subnet Types**
Each VPC consists of **subnets**, which are portions of the VPC's IP range assigned to different **Availability Zones**.

1. **Public Subnets**
   - Directly connected to the **internet gateway**.
   - Instances get a **public IP**.
   - Used for applications requiring internet access (e.g., web servers, load balancers).
  
2. **Private Subnets**
   - **No direct internet access.**
   - Typically used for **databases, backend servers, and internal applications**.
   - Can access the internet via a **NAT Gateway**.

### **Example AWS Region with Multiple Availability Zones**
AWS regions are divided into **Availability Zones (AZs)**:
- Example: **EU-West-1 (Ireland)**
  - `eu-west-1a`
  - `eu-west-1b`
  - `eu-west-1c`

A common **highly available** network setup:
- **Public Subnets:** One in each AZ (`10.0.1.0/24`, `10.0.2.0/24`, `10.0.3.0/24`).
- **Private Subnets:** One in each AZ (`10.0.4.0/24`, `10.0.5.0/24`, `10.0.6.0/24`).

Each subnet has a **fixed range of 256 IPs** (`/24`), which is typically enough for most workloads.

---

## **Connecting Subnets: Internet Gateway & Route Tables**
### **Internet Gateway (IGW)**
- A **public subnet** needs an **Internet Gateway (IGW)** to allow external traffic.
- An IGW is **attached to the VPC** and enables outbound internet access.

### **Route Tables**
- **Define how traffic is routed** within a VPC.
- Each subnet is associated with **a route table**.
- **Public subnet route table:**
  - Routes local traffic within the VPC.
  - Routes internet-bound traffic (`0.0.0.0/0`) to **Internet Gateway (IGW)**.
- **Private subnet route table:**
  - No direct internet access.
  - Can use a **NAT Gateway** to allow outgoing connections.

#### **Example Route Table:**
| Destination | Target         | Notes                      |
|-------------|--------------|----------------------------|
| `10.0.0.0/16` | local      | Internal VPC communication |
| `0.0.0.0/0`  | IGW ID      | Internet access (public)   |

---

## **VPC Peering & Private Communication**
### **VPC Peering**
- Used when **two VPCs** need private communication.
- Avoids the need for **public IPs**.
- Example use case: Connecting **backend services** hosted in different VPCs.

### **Private Communication in a VPC**
- **Instances in the same VPC can communicate privately**.
- If instances are in different subnets but **same VPC**, they can use **private IPs**.
- Security groups define **rules for allowed communication**.

---

## **Best Practices for AWS VPC**
✅ **Use Terraform for Infrastructure as Code (IaC).**  
✅ **Separate public and private subnets** to isolate workloads.  
✅ **Deploy applications in multiple AZs** for high availability.  
✅ **Restrict access using Security Groups and Network ACLs.**  
✅ **Avoid using the default VPC in production.**  
✅ **Use VPC peering or AWS Transit Gateway** for secure VPC interconnections.  
✅ **Leverage NAT Gateways** to allow private instances to access the internet securely.  

---

## **Next Steps**
Now that we have covered **VPC fundamentals**, the next step is to implement these concepts using **Terraform**. In the next lecture, we will:
- Write a Terraform script to create a **custom VPC**.
- Define **subnets, route tables, and an internet gateway**.
- Launch an EC2 instance inside a public subnet.

---

This enhanced version of the transcript includes:
✅ **Additional explanations** for better understanding.  
✅ **Best practices** to follow in real-world scenarios.  
✅ **Expanded examples** of CIDR blocks, route tables, and subnet designs.  
✅ **References to AWS official documentation** for further reading.  

