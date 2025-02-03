## **Terraform Demo: Creating an EC2 Instance on AWS**

### **Introduction**
In this demo, I will show you how to:
- Create a Terraform configuration file to spin up an **EC2 instance** (either a `t2.micro` or `t3.micro`).
- Use `terraform apply` to deploy the infrastructure on AWS.
- Understand how Terraform interacts with AWS credentials.
- Modify and manage the instance, including changing instance types and destroying resources.

Both **t2.micro** and **t3.micro** instances are part of the AWS Free Tier. However, be cautious with **t3.micro**, as it has an **"unlimited mode"** enabled by default, which can lead to additional charges if CPU usage exceeds a threshold.

### **Prerequisites**
Before proceeding, ensure that:
1. You have an **AWS account** with programmatic access (AWS access keys).
2. Terraform is installed on your system (`terraform -v` should display the installed version).
3. AWS CLI is installed and configured (`aws configure` is used to set up credentials).
4. (Optional) You have **Visual Studio Code** or another code editor for editing Terraform files.

---

## **1. Cloning the Terraform Course Repository**
My Terraform configuration files are available in my GitHub repository. You can either:
- **Clone the repository**: This allows you to pull the latest updates when changes are made.
- **Download the ZIP**: This provides a standalone version without update capabilities.

### **Cloning the Repository**
To clone the repository, run:
```sh
git clone https://github.com/my-terraform-course.git
cd my-terraform-course
```
This will create a directory structure containing multiple Terraform configuration files. 

💡 *If you want to keep your local version updated with my changes, use `git pull` periodically.*

Inside the repository, you'll find a **README** file that maps different Terraform demos to the respective files, making navigation easier.

---

## **2. Setting Up the Terraform File**
Navigate to the `first-steps` directory, where you’ll find a Terraform file named `instance.tf`. 

```sh
cd first-steps
```

Open `instance.tf` in an editor:
```sh
code instance.tf   # VS Code
vim instance.tf    # Vim/VI (optional)
```

The file contains Terraform code for provisioning an EC2 instance.

### **Terraform Configuration for EC2 Instance**
Here’s what a minimal Terraform configuration might look like:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0abcdef1234567890"  # Update this with a valid Ubuntu AMI
  instance_type = "t2.micro"
  tags = {
    Name = "Terraform-Instance"
  }
}
```

This configuration:
- Uses the AWS provider.
- Creates an EC2 instance with an **Ubuntu AMI** (replace with the latest Ubuntu AMI from AWS).
- Specifies `t2.micro` as the instance type (you can change it to `t3.micro`).
- Assigns a tag (`Name = "Terraform-Instance"`) for easy identification in AWS.

---

## **3. Configuring AWS Credentials**
Terraform requires AWS credentials to create resources. There are two ways to provide them:

### **Option 1: Hardcoding Credentials (Not Recommended)**
You can specify credentials directly in the Terraform provider block:
```hcl
provider "aws" {
  access_key = "YOUR_ACCESS_KEY"
  secret_key = "YOUR_SECRET_KEY"
  region     = "us-east-1"
}
```
⚠️ *Hardcoding credentials is a security risk. If you accidentally push this file to a public repository, your credentials may be exposed!*

### **Option 2: Using AWS CLI Configuration (Recommended)**
A safer method is to use the AWS CLI to store credentials securely:
```sh
aws configure
```
This will prompt you for:
- AWS Access Key ID
- AWS Secret Access Key
- Default region (e.g., `us-east-1`)
- Output format (`json`, `yaml`, or `table`)

Once configured, Terraform will automatically use these credentials.

---

Terraform automatically detects and uses **AWS environment variables** if they are set in your shell session. It does not require `aws configure` unless you want to use **named profiles**. The AWS provider in Terraform checks the environment variables in the following priority order:

### **3.1. Terraform's Default Environment Variables**
Terraform **automatically picks up** these environment variables:

| Environment Variable        | Description                                   |
|-----------------------------|-----------------------------------------------|
| `AWS_ACCESS_KEY_ID`         | AWS access key ID                             |
| `AWS_SECRET_ACCESS_KEY`     | AWS secret access key                         |
| `AWS_SESSION_TOKEN` (optional) | Temporary session token for IAM roles (if using STS) |
| `AWS_DEFAULT_REGION`        | Default AWS region (alternative to setting `region` in the provider block) |

Since you have already set:

```sh
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_DEFAULT_REGION
```

Terraform will **automatically** use these values when running commands like `terraform plan` or `terraform apply`.

---

### **3.2. How Terraform Uses Environment Variables**
Terraform's **AWS provider** follows this process when authenticating:
1. **Checks explicit credentials** in the `provider "aws"` block (hardcoded credentials – **not recommended**).
2. **Checks environment variables** (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`).
3. **Checks AWS shared credentials file** (`~/.aws/credentials` created by `aws configure`).
4. **Checks AWS profiles** if `profile` is set in the provider block.
5. **Uses EC2 instance metadata** if running Terraform inside an AWS EC2 instance with an attached IAM role.

Since your **environment variables are already set**, Terraform **skips** the need for `aws configure` and directly reads the credentials.

---

### **3.3. Verifying That Terraform Uses Your Environment Variables**
To confirm Terraform is correctly picking up your credentials, run:

```sh
env | grep AWS
```

This will list all AWS-related environment variables Terraform can use.

Alternatively, run:

```sh
terraform plan
```

If Terraform is correctly using your credentials, it will not prompt for authentication and will show a plan for creating resources.

---

## **4. Initializing Terraform**
Before applying any changes, run:

```sh
terraform init
```

This does the following:
✅ Downloads the necessary **Terraform provider plugins** (e.g., AWS provider).  
✅ Creates a `.terraform/` directory to store provider data.  
✅ Generates a `.terraform.lock.hcl` file to lock provider versions.  

After initialization, check the directory:

```sh
ls -la
```

You'll see the `.terraform/` folder and the lock file.

---

## **5. Running Terraform Plan**
Before applying the configuration, review what Terraform will do:

```sh
terraform plan
```

This command:
- Shows a **preview** of the resources Terraform will create.
- Helps detect issues before making changes.
- Displays the exact **AWS resources** to be provisioned.

Look for:
- `"Plan: 1 to add, 0 to change, 0 to destroy"` → This means Terraform will create **one new resource**.

---

## **6. Applying the Terraform Configuration**
Once you're satisfied with the plan, apply the changes:

```sh
terraform apply
```

Terraform will prompt:
```
Do you want to perform these actions?
Enter a value: yes
```

Type **`yes`** and press **Enter**.

Terraform will now create the EC2 instance. This process may take a few minutes.

### **Verifying in AWS Console**
1. Open the **AWS Management Console**.
2. Navigate to **EC2 Dashboard** → **Instances**.
3. Ensure the correct **AWS region** (`us-east-1`) is selected.
4. Look for the instance with the tag `"Terraform-Instance"`.

---

## **7. Modifying the Instance Type**
To modify the instance type (e.g., from `t2.micro` to `t3.micro`):
1. Update `instance.tf`:
```hcl
instance_type = "t3.micro"
```
2. Run:
```sh
terraform apply
```
Terraform will detect the change and update the instance **in place** (without destroying it).

---

## **8. Destroying the Instance**
If you no longer need the instance, run:

```sh
terraform destroy
```
Terraform will prompt for confirmation:
```
Do you really want to destroy all resources?
Enter a value: yes
```
After entering `yes`, Terraform will delete the instance.

---

## **9. Best Practices & Additional Notes**
### **Avoiding Unintentional Costs**
- **Monitor your AWS billing**: AWS Free Tier limits apply for `t2.micro` and `t3.micro` but exceeding usage may incur charges.
- **Disable "unlimited mode"**: `t3.micro` instances support burstable performance, which can exceed free tier limits.
- **Shut down unused instances**: Use `terraform destroy` to remove resources.

### **Using Variables for Flexibility**
Instead of hardcoding values, use variables:
```hcl
variable "instance_type" {
  default = "t2.micro"
}

resource "aws_instance" "example" {
  ami           = "ami-0abcdef1234567890"
  instance_type = var.instance_type
}
```
This allows setting instance type dynamically:
```sh
terraform apply -var="instance_type=t3.micro"
```

### **State Management**
Terraform maintains a **state file** (`terraform.tfstate`) that tracks resources. Be cautious when editing it manually.

### **Using Remote State**
For collaboration, use **remote backends** (S3, Terraform Cloud) instead of local state files.

---

## **Conclusion**
In this demo, we:
✅ Set up Terraform to provision an **EC2 instance**.  
✅ Used **Terraform commands** (`init`, `plan`, `apply`, `destroy`).  
✅ Managed AWS credentials securely.  
✅ Modified instance parameters dynamically.  
✅ Applied **best practices** for cost management and state handling.  

This is just a starting point! You can extend this setup by:
- **Adding security groups** for SSH access.
- **Attaching IAM roles** for secure access to AWS services.
- **Automating Terraform** using **GitHub Actions** or **GitLab CI/CD**.

Happy Terraforming! 🚀