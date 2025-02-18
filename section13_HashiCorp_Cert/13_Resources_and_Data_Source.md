# **Terraform Resources and Data Sources**  

## **Official Resources**  
To gain a deeper understanding of Terraform resources, data sources, and dependencies, check out these official HashiCorp resources:  
- 📌 **[Terraform Resources](https://developer.hashicorp.com/terraform/language/resources)** – Learn how to create and manage infrastructure components.  
- 📌 **[Terraform Data Sources](https://developer.hashicorp.com/terraform/language/data-sources)** – Guide on retrieving existing infrastructure information.  
- 📌 **[Terraform Dependencies](https://developer.hashicorp.com/terraform/language/meta-arguments/depends_on)** – Explanation of how Terraform manages resource dependencies.  

---

## **1. Understanding Terraform Resources and Data Sources**  

Terraform **manages two types of objects**:  

| **Object Type** | **Definition** | **Purpose** |
|---------------|--------------|-----------|
| **Resources** (`resource`) | Defines infrastructure objects that Terraform creates and manages. | Used to create EC2 instances, VPCs, databases, security groups, etc. |
| **Data Sources** (`data`) | Retrieves existing infrastructure information. | Used to fetch AMI IDs, existing VPCs, or security groups without creating new resources. |

### **Key Differences**  
| **Feature**   | **Resource (`resource`)** | **Data Source (`data`)** |
|--------------|-----------------|----------------|
| **Creates infrastructure?** | ✅ Yes | ❌ No |
| **Modifies state?** | ✅ Yes | ❌ No |
| **Example usage** | `aws_instance`, `aws_vpc` | `aws_ami`, `aws_subnet` |

---

## **2. Using Data Sources in Terraform**  

### **Why Use Data Sources?**  
✅ **Retrieve existing infrastructure details** (e.g., VPC ID, latest AMI).  
✅ **Avoid hardcoding values** (e.g., dynamically fetch AMI IDs).  
✅ **Ensure infrastructure consistency** when using existing resources.

---

### **3. Example: Using a Data Source to Fetch an AMI ID**  

Instead of hardcoding an AMI ID, we use **a data source to fetch the latest Ubuntu AMI**.

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical (Ubuntu) account ID

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id  # Fetches AMI from data source
  instance_type = "t2.micro"
}
```
✅ **Automatically retrieves the latest Ubuntu AMI** without hardcoding it.  
✅ **Ensures consistency across deployments.**  

---

## **4. Comparing Resources and Data Sources: AWS Subnets**  

### **Using a Resource to Create a New Subnet**
```hcl
resource "aws_subnet" "new_subnet" {
  vpc_id     = aws_vpc.my_vpc.id
  cidr_block = "10.0.1.0/24"
}
```
✅ **Creates a new subnet in the VPC.**  
❌ **Not useful if you want to reference an existing subnet.**  

---

### **Using a Data Source to Fetch an Existing Subnet**
```hcl
data "aws_subnet" "existing_subnet" {
  filter {
    name   = "tag:Name"
    values = ["my-existing-subnet"]
  }
}

resource "aws_instance" "web" {
  subnet_id     = data.aws_subnet.existing_subnet.id
  instance_type = "t2.micro"
}
```
✅ **References an existing subnet instead of creating a new one.**  
✅ **Ensures consistency across deployments.**  

---

## **5. Terraform's Automatic Dependency Management**  

Terraform **automatically orders resource creation based on dependencies**.  
It determines **which resources must be created first** before others.

### **Example: Implicit Dependencies**  
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "subnet" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  subnet_id     = aws_subnet.subnet.id
  instance_type = "t2.micro"
}
```
✅ **Terraform ensures the VPC is created first, then the subnet, then the instance.**  
✅ **No need to manually specify dependencies—Terraform infers them automatically.**  

---

## **6. Using `depends_on` for Manual Dependencies**  

In rare cases, Terraform **does not detect dependencies automatically**.  
The `depends_on` argument **forces Terraform to create resources in a specific order**.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  depends_on    = [aws_security_group.firewall]
}
```
✅ Ensures that `aws_security_group.firewall` is created **before** launching the instance.  
✅ **Useful when Terraform cannot infer the dependency from attributes.**  

---

## **7. Terraform's Dependency Graph**  

Terraform **creates a dependency graph** based on resource references.  
Each resource must **wait** for its dependencies before being created.

### **Example: Dependency Graph for EC2 with an Elastic IP**  

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

resource "aws_eip" "web_eip" {
  instance = aws_instance.web.id
}
```
✅ **Terraform ensures the EC2 instance is created before the Elastic IP.**  
✅ **No need to use `depends_on` since Terraform detects the dependency automatically.**  

---

## **8. Best Practices for Resources, Data Sources, and Dependencies**  

✅ **Use resources to create new infrastructure** – Resources modify the Terraform state.  
✅ **Use data sources to fetch existing infrastructure** – Data sources do not modify Terraform state.  
✅ **Use implicit dependencies when possible** – Terraform automatically determines resource order.  
✅ **Use `depends_on` only when necessary** – Avoid adding unnecessary dependencies.  
✅ **Use data sources for dynamic lookups** – Fetch AMI IDs, VPCs, or other existing infrastructure dynamically.  

---

## **9. Summary of Key Terraform Commands**  

| **Command**                     | **Description** |
|----------------------------------|----------------|
| `terraform plan`                 | Previews infrastructure changes. |
| `terraform apply`                | Applies changes to infrastructure. |
| `terraform state list`           | Lists all resources tracked in state. |
| `terraform graph`                | Generates a dependency graph of Terraform resources. |

For further learning, check out:  
- **[Terraform Resources](https://developer.hashicorp.com/terraform/language/resources)**  
- **[Terraform Data Sources](https://developer.hashicorp.com/terraform/language/data-sources)**  
- **[Terraform Dependencies](https://developer.hashicorp.com/terraform/language/meta-arguments/depends_on)**  

---

## **10. Key Takeaways**  

✅ **Terraform manages both `resources` and `data sources`.**  
✅ **Resources create and modify infrastructure**, while **data sources fetch existing infrastructure information**.  
✅ **Terraform automatically detects dependencies based on resource references.**  
✅ **Use `depends_on` only when Terraform cannot infer dependencies automatically.**  
✅ **Terraform follows a dependency graph to ensure proper resource creation order.**  

