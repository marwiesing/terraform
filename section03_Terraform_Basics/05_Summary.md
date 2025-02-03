## **Recap: Setting Up Terraform and Managing Infrastructure**

In this lecture, we completed the foundational steps of using Terraform to provision infrastructure on AWS. Let's review the key takeaways.

---

### **1. Setting Up Cloud Access: AWS, Azure, and DigitalOcean**

Before using Terraform, we first **set up cloud provider access**. While we used **AWS**, the same principles apply to other cloud providers.

#### **Steps for AWS Setup**
- **Opened an AWS Account** (required to use AWS services).
- **Created an IAM Admin User** to follow security best practices:
  - Instead of using the root account, we created a dedicated IAM user.
  - Assigned **AdministratorAccess** permissions to manage AWS resources via Terraform.
  - Created and downloaded **Access Keys** for Terraform to authenticate.

#### **Using Other Cloud Providers**
Terraform supports multiple cloud providers, including:
- **Azure**: You need to create a **Service Principal** with necessary permissions.
- **DigitalOcean**: Instead of an IAM user, you generate an **API token** with permissions to create and manage droplets (instances).
- **Google Cloud Platform (GCP)**: You must create and download a **service account key**.

🔗 **More Information**:  
- [AWS IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)  
- [Terraform Cloud Provider Documentation](https://registry.terraform.io/browse/providers)

---

### **2. Writing a Basic Terraform Configuration**

We wrote a simple **Terraform configuration file** (`main.tf`) to provision an AWS **t2.micro EC2 instance**.

#### **Example Terraform File**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id  # Ubuntu 20.04 AMI
  instance_type = "t2.micro"

  tags = {
    Name = "TerraformInstance"
    Environment = "Dev"
  }
}
```

#### **What This Configuration Does**
- Specifies AWS as the cloud provider.
- Defines an EC2 instance using a dynamically retrieved **Ubuntu 20.04 AMI**.
- Assigns the **t2.micro** instance type (part of AWS free tier).
- Adds **tags** for easy identification.

---

### **3. Running Terraform Commands**
Once the Terraform configuration was written, we used the following commands:

#### **Step 1: Initialize Terraform**
```bash
terraform init
```
✅ **Purpose**: Downloads required providers and sets up Terraform.

---

#### **Step 2: Preview Changes with `terraform plan`**
```bash
terraform plan
```
✅ **Purpose**:  
- Shows **what resources will be created, modified, or destroyed**.  
- Does **not** apply any changes—just a preview.

🔹 **Example Output:**
```
Terraform will perform the following actions:

  + aws_instance.web will be created
    - ami: "ami-xxxxxxxxx"
    - instance_type: "t2.micro"
```

---

#### **Step 3: Apply Changes with `terraform apply`**
```bash
terraform apply
```
✅ **Purpose**: Actually provisions the resources in AWS.

🚀 **Best Practice: Use `terraform plan` before applying changes to review them.**

---

### **4. Using Terraform Plan with an Out File**
For more controlled deployments, you can **save the plan** and apply it later.

```bash
terraform plan -out=tfplan
```
✅ **Purpose**: Saves the proposed changes into a file (`tfplan`).  

To apply only the saved changes:
```bash
terraform apply tfplan
```
✅ **Why is this useful?**
- Ensures **no unexpected changes** occur.
- Can be used in **CI/CD pipelines** for automated infrastructure deployment.

---

### **5. Destroying Infrastructure**
If you created infrastructure for testing purposes, it's best to **destroy it to avoid costs**.

```bash
terraform destroy
```
✅ **Purpose**: Removes all resources Terraform created.

⚠️ **Important Warning:**  
- Never run `terraform destroy` in a **production environment** unless you are sure!
- This command will **delete everything Terraform manages**, potentially causing **downtime or data loss**.

### **Example Safe Approach**
To prevent accidental deletion:
```bash
terraform destroy -target aws_instance.web
```
This will **only destroy the specific resource** (`aws_instance.web`) rather than everything.

---

### **Key Takeaways**
✅ **Use IAM Users & API Keys**: Set up cloud provider access securely.  
✅ **Use `terraform plan` Before `apply`**: Always check what Terraform will do.  
✅ **Save Terraform Plans (`-out=tfplan`)**: Ensures predictable changes.  
✅ **Destroy Lab Resources**: Use `terraform destroy` to remove test instances and avoid AWS charges.  
✅ **Be Cautious in Production**: Never run `terraform destroy` unless intentional.  

---

#### **Next Steps**
In the next lecture, we will:
- Introduce **Terraform Variables** for more flexible configurations.
- Add **Security Groups** to allow SSH access to instances.
- Discuss **State Management** and best practices for collaboration.

🚀 **Terraform simplifies infrastructure management, making it repeatable, scalable, and automated! Keep experimenting and learning!** 🔥

---

#### **Resources**
- 🔗 [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/connect_instance)
- 🔗 [Ubuntu AWS Guide](https://documentation.ubuntu.com/aws/en/latest/)
- 🔗 [AWS Free Tier Pricing](https://aws.amazon.com/free/)
- 🔗 [Terraform CLI Documentation](https://developer.hashicorp.com/terraform/cli)

