# **Terraform with AWS: Deploying RDS for Managed Databases**

## **Resources**
- [AWS RDS Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
- [Terraform AWS Provider: RDS](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance)
- [Terraform AWS Provider: DB Subnet Group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_subnet_group)
- [AWS Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

---

## **Part 1: Introduction to AWS RDS**
### **What is RDS?**
Amazon **Relational Database Service (RDS)** is a **fully managed database service** that simplifies deployment, scaling, backup, and maintenance. RDS supports:
- **MySQL**
- **MariaDB**
- **PostgreSQL**
- **Microsoft SQL Server**
- **Oracle**
- **Amazon Aurora (AWS's own optimized database)**

### **Why Use RDS Instead of Running a Database on EC2?**
✅ **Automated Backups & Snapshots** – RDS provides automated daily backups.  
✅ **High Availability** – Multi-AZ deployments ensure **failover to a standby** in case of failure.  
✅ **Scalability** – You can **resize storage and compute capacity** on demand.  
✅ **Automatic Updates & Maintenance** – AWS manages security patches.  

---

## **Part 2: Key Components of RDS in Terraform**
To deploy an **RDS instance**, we need:
1. **A VPC**: The database must be in a **private subnet**.
2. **DB Subnet Group**: Specifies which subnets the RDS can be placed in.
3. **DB Parameter Group**: Configures database settings.
4. **Security Group**: Controls **which instances** can access the RDS.
5. **RDS Instance**: The actual **managed database**.

---

### **Step 1: Creating a DB Subnet Group**
An RDS instance **must be deployed in a VPC** with a **DB Subnet Group**, ensuring redundancy.

```hcl
resource "aws_db_subnet_group" "mariadb-subnet" {
  name        = "mariadb-subnet"
  description = "RDS subnet group"
  subnet_ids  = [aws_subnet.main-private-1.id, aws_subnet.main-private-2.id]
}
```
✅ **Ensures RDS is launched in private subnets (`main-private-1` & `main-private-2`).**  
✅ **Prepares for a Multi-AZ deployment (if enabled).**  

---

### **Step 2: Configuring a DB Parameter Group**
RDS does not provide **shell access**, so **database settings must be configured through a parameter group**.

```hcl
resource "aws_db_parameter_group" "mariadb-parameters" {
  name        = "mariadb-parameters"
  family      = "mariadb10.4"
  description = "MariaDB parameter group"

  parameter {
    name  = "max_allowed_packet"
    value = "16777216"
  }
}
```
✅ **Allows fine-tuning of database configurations.**  
✅ **Example: `max_allowed_packet` increases the allowed packet size for large queries.**  

---

### **Step 3: Creating a Security Group for RDS**
RDS **should not be publicly accessible**. Instead, allow connections **only from specific EC2 instances**.

```hcl
resource "aws_security_group" "allow-mariadb" {
  vpc_id      = aws_vpc.main.id
  name        = "allow-mariadb"
  description = "Allow MariaDB access from example instance"

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.example-instance.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "allow-mariadb"
  }
}
```
✅ **Allows only EC2 instances in `example-instance` security group to access the database.**  
✅ **Prevents external internet access.**  

---

### **Step 4: Deploying the RDS Instance**
Finally, deploy the **RDS instance**.

```hcl
resource "aws_db_instance" "mariadb" {
  allocated_storage       = 100 # Minimum recommended storage for performance
  engine                  = "mariadb"
  engine_version          = "10.4"
  instance_class          = "db.t2.small"  # Use "db.t2.micro" for free tier
  identifier              = "mariadb"
  db_name                 = "mariadb"
  username                = "root"
  password                = var.RDS_PASSWORD
  db_subnet_group_name    = aws_db_subnet_group.mariadb-subnet.name
  parameter_group_name    = aws_db_parameter_group.mariadb-parameters.name
  multi_az                = false  # Change to true for high availability
  vpc_security_group_ids  = [aws_security_group.allow-mariadb.id]
  storage_type            = "gp2"
  backup_retention_period = 30
  availability_zone       = aws_subnet.main-private-1.availability_zone
  skip_final_snapshot     = true  # Prevents Terraform from taking a final snapshot on destroy

  tags = {
    Name = "mariadb-instance"
  }
}
```
✅ **Launches an RDS instance in a private subnet.**  
✅ **Enables daily backups (`backup_retention_period = 30`).**  
✅ **Prevents unwanted deletion (`skip_final_snapshot = false` is recommended for production).**  

---

## **Part 3: Deploying an EC2 Instance to Connect to RDS**
Since RDS is **not publicly accessible**, an **EC2 instance inside the VPC** must be used to connect.

```hcl
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main-public-1.id
  key_name      = aws_key_pair.mykeypair.key_name

  vpc_security_group_ids = [aws_security_group.example-instance.id]
}
```
✅ **This instance will be used to SSH into and connect to RDS.**  
✅ **Security group ensures this EC2 can reach RDS.**  

---

## **Part 4: Running Terraform & Connecting to RDS**
### **Step 1: Apply the Terraform Configuration**
```bash
terraform init
terraform apply -var "RDS_PASSWORD=mysecurepassword"
```
✅ **Creates the VPC, security groups, RDS instance, and EC2 instance.**  

---

### **Step 2: Retrieve the RDS Endpoint**
After applying, find the **RDS endpoint** using:
```bash
terraform output rds
```
Expected output:
```
rds = mariadb.c9abcdefghijk.eu-west-1.rds.amazonaws.com
```

---

### **Step 3: Connect to the EC2 Instance**
SSH into the instance:
```bash
ssh -i mykey ubuntu@<ec2-instance-public-ip>
```

---

### **Step 4: Install the MySQL Client and Connect**
Inside the EC2 instance:
```bash
sudo apt update && sudo apt install -y mysql-client
```
Now connect to the RDS instance:
```bash
mysql -h mariadb.c9abcdefghijk.eu-west-1.rds.amazonaws.com -u root -p
```
✅ **Enter the RDS password provided to Terraform.**  
✅ **You are now connected to the RDS database.**  

---

## **Part 5: Best Practices**
✅ **Use `multi_az = true` for high availability in production.**  
✅ **Set `skip_final_snapshot = false` to avoid losing data when deleting RDS.**  
✅ **Store passwords securely using AWS Secrets Manager instead of plain variables.**  
✅ **Restrict security groups to only required instances.**  

---

## **Part 6: Cleaning Up**
To **destroy all resources** and avoid unnecessary charges:
```bash
terraform destroy
```
✅ **Ensure backups are taken before deleting the RDS instance.**  

---

## **Final Thoughts**
- **RDS simplifies database management** with automated backups, high availability, and security.
- **Terraform allows you to automate the entire setup** of **RDS, networking, security, and access**.
- **Security groups must be properly configured** to allow access only from trusted sources.

---

## **Next Steps**
- **Creating Read Replicas for Performance Scaling.**
- **Automating RDS Backups & Snapshots with Terraform.**
- **Monitoring RDS Metrics with AWS CloudWatch.**

---
