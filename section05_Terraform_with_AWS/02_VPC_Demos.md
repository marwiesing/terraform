## **Terraform with AWS: Setting Up a VPC and Launching EC2 Instances**
### **Resources**
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- [Terraform AWS Provider: VPC](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)
- [AWS CIDR Block Guide](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#VPC_Sizing)
- [AWS Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [AWS NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Terraform AWS EC2 Instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)
- [AWS Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [AWS Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

---

## **Part 1: Creating a VPC with Terraform**
### **Overview**
In this Terraform setup, we will configure a **Virtual Private Cloud (VPC)** with:
- **Public & Private Subnets** across multiple Availability Zones.
- **An Internet Gateway** for public instance connectivity.
- **A NAT Gateway** for private instance outbound access.
- **Route Tables** to control traffic flow.

The **Terraform files** are organized as follows:
```bash
$ ls terraform-course/demo-7/*.tf
terraform-course/demo-7/nat.tf       
terraform-course/demo-7/vars.tf      
terraform-course/demo-7/vpc.tf
terraform-course/demo-7/provider.tf  
terraform-course/demo-7/versions.tf
```
---

### **Step 1: Defining the VPC**
```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "main"
  }
}
```
✅ **CIDR Block**: `10.0.0.0/16` (Allows 65,536 private IPs).  
✅ **DNS Support**: Enables internal hostname resolution.  

---

### **Step 2: Creating Public and Private Subnets**
Each **Availability Zone (AZ)** has a **public and private subnet**.

#### **Public Subnets**
```hcl
resource "aws_subnet" "main-public-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1a"
  tags = { Name = "main-public-1" }
}
```
✅ **Public IP on Launch**: `true` ensures instances can reach the internet.

#### **Private Subnets**
```hcl
resource "aws_subnet" "main-private-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.4.0/24"
  map_public_ip_on_launch = false
  availability_zone       = "eu-west-1a"
  tags = { Name = "main-private-1" }
}
```
✅ **No Public IP**: Private instances must use **NAT Gateway** for outbound traffic.

---

### **Step 3: Configuring Routing & Internet Connectivity**
#### **Internet Gateway for Public Subnets**
```hcl
resource "aws_internet_gateway" "main-gw" {
  vpc_id = aws_vpc.main.id
  tags = { Name = "main-igw" }
}
```
#### **Route Table for Public Subnets**
```hcl
resource "aws_route_table" "main-public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main-gw.id
  }
}
```
✅ **Associating Public Subnets with the Route Table**
```hcl
resource "aws_route_table_association" "main-public-1-a" {
  subnet_id      = aws_subnet.main-public-1.id
  route_table_id = aws_route_table.main-public.id
}
```
---

### **Step 4: Configuring NAT Gateway for Private Subnets**
Private instances need a **NAT Gateway** for outbound internet access.

#### **Create an Elastic IP**
```hcl
resource "aws_eip" "nat" {
  domain = "vpc"
}
```
#### **Create the NAT Gateway**
```hcl
resource "aws_nat_gateway" "nat-gw" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.main-public-1.id
}
```
#### **Route Table for Private Subnets**
```hcl
resource "aws_route_table" "main-private" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat-gw.id
  }
}
```

---

## **Part 2: Launching an EC2 Instance in a VPC**
### **Step 1: Define the EC2 Instance**
```hcl
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id = aws_subnet.main-public-1.id
  vpc_security_group_ids = [aws_security_group.allow-ssh.id]
  key_name = aws_key_pair.mykeypair.key_name
}
```
✅ **Instance is in a Public Subnet**  
✅ **Security Group Controls Access**  

### **Step 2: Configure SSH Key Pair**
```hcl
resource "aws_key_pair" "mykeypair" {
  key_name   = "mykeypair"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}
```
**⚠️ Important:** Never upload your **private key** to AWS.

---

### **Step 3: Security Group for SSH Access**
```hcl
resource "aws_security_group" "allow-ssh" {
  vpc_id = aws_vpc.main.id
  name        = "allow-ssh"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Change to your IP for security
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
✅ **Best Practice**: Restrict SSH to your **own IP address** (`/32`).

---

## **Part 3: Deploying Terraform Configuration**
```bash
terraform init
terraform plan
terraform apply
```
✅ **Terraform automatically provisions all resources**.

---

## **Part 4: Connecting to the Instance**
```bash
ssh -i mykey ubuntu@<instance-ip>
```
✅ **Verify Network Configuration**
```bash
ifconfig
route -n
```
✅ **Test Internet Connectivity**
```bash
ping google.com
```

---

## **Final Thoughts**
✅ **VPC with Public & Private Subnets**  
✅ **NAT Gateway for Private Instances**  
✅ **EC2 Instance with Secure SSH Access**  
✅ **Terraform Automation for Infrastructure as Code**  

---
## **Next Steps**
- Deploy **Load Balancer** & **Auto Scaling Groups**.
- Implement **Terraform modules** for modular deployments.

