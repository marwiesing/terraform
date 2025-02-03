## **Terraform: Using Data Sources to Dynamically Retrieve AWS AMI IDs (Ubuntu 20.04 Focal Fossa)**

In this lecture, we are continuing to refine our Terraform configuration by making it more **dynamic** and **flexible**. A key improvement is to **avoid hardcoding the AMI ID** and instead use **Terraform data sources** to fetch the latest Ubuntu 20.04 AMI dynamically.

### **Why Avoid Hardcoding AMI IDs?**
When deploying EC2 instances, it’s important to use the latest security updates and system improvements. However, **hardcoding AMI IDs** can lead to:
- **Security risks**: Outdated AMIs may lack the latest patches.
- **Operational inefficiencies**: You would need to manually update Terraform files whenever a new AMI is released.
- **Increased maintenance overhead**: Each environment (development, staging, production) may require different AMI versions.

Instead, we use **Terraform data sources** to dynamically query AWS and fetch the **latest official Ubuntu 20.04 AMI**.

---

### **Terraform Resources vs. Data Sources**

Terraform distinguishes between:
1. **Resources** – Create and manage new infrastructure components (e.g., EC2 instances, VPCs).
2. **Data Sources** – Retrieve existing information from AWS (e.g., fetching the latest AMI).

We will **use a data source to fetch the latest Ubuntu 20.04 AMI** instead of hardcoding it.

---

### **Using Data Sources to Fetch the Latest Ubuntu 20.04 AMI**

#### **Step 1: Define a Data Source for Ubuntu 20.04**
We modify our Terraform configuration to retrieve the latest **Ubuntu 20.04 (Focal Fossa) AMI**.

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical (Official Ubuntu AMIs)
}
```

#### **Explanation of Parameters:**
| Parameter              | Description |
|------------------------|-------------|
| `most_recent = true`   | Retrieves the latest available Ubuntu 20.04 AMI. |
| `name = "ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"` | Ensures that we only retrieve AMIs matching Ubuntu 20.04. |
| `virtualization-type = "hvm"` | Ensures that the AMI supports **hardware virtualization** (modern EC2 instances). |
| `owners = ["099720109477"]` | Uses **Canonical's AWS account ID** to ensure AMI authenticity. |

🔗 **Source:** [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/connect_instance)  
🔗 **Ubuntu AWS Documentation:** [Ubuntu on AWS](https://documentation.ubuntu.com/aws/en/latest/)

---

#### **Step 2: Use the AMI in an EC2 Instance**
Now that we have a **dynamic** AMI, let’s use it in our EC2 instance:

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id  # Dynamically retrieve AMI ID
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleInstance"
    Environment = "Development"
    OS = "Ubuntu 20.04"
  }
}
```

#### **What’s Happening Here?**
- The **AMI ID is automatically assigned** from the data source.
- The instance type is set to **t2.micro** (part of the free tier).
- **Tags** are used to categorize the instance.

---

### **Finding Ubuntu AMI IDs Manually**
If you want to verify which AMI Terraform will use, you can **manually list AMI IDs** via AWS CLI:

```bash
aws ec2 describe-images --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*" \
  --query 'Images[*].[ImageId,Name]' --output table
```

This will output:
```
-----------------------------------------------
| ImageId       | Name                        |
-----------------------------------------------
| ami-12345678  | ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20240101 |
-----------------------------------------------
```

Terraform will pick the **latest** one available.

---

### **Expanding Our Infrastructure: Adding Networking**
Right now, our instance is created **without** networking. To make it accessible:
1. **Attach a Security Group** to allow SSH.
2. **Assign a Public IP Address**.
3. **Ensure it’s in a VPC with Internet Access**.

#### **Updated EC2 Configuration with Security Group**
```hcl
resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "Allow SSH inbound traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Allow from anywhere (Change for security)
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]

  tags = {
    Name = "Ubuntu20Instance"
  }
}
```

This allows us to **SSH into our instance** once we add an SSH key.

---

### **Terraform Workflow**
To deploy this configuration, follow these steps:

1. **Initialize Terraform** (Downloads AWS Provider)
   ```bash
   terraform init
   ```

2. **Plan the Changes** (Shows what will be created)
   ```bash
   terraform plan
   ```

3. **Apply the Configuration** (Creates the instance)
   ```bash
   terraform apply
   ```

4. **Connect via SSH**
   Once the instance is running, connect via:
   ```bash
   ssh -i my-key.pem ubuntu@<public-ip>
   ```

---

### **Troubleshooting Terraform Errors**

| Error  | Cause | Solution |
|--------|-------|----------|
| `Error: No AMI found matching criteria` | The AMI filter is incorrect. | Verify the AMI using AWS CLI. |
| `Error: Unable to locate credentials` | AWS credentials are missing. | Run `aws configure`. |
| `Error: Instance failed to launch` | Security group or VPC misconfiguration. | Ensure a VPC and SG exist. |

---

### **Key Takeaways**
✅ **Use Data Sources** to dynamically retrieve Ubuntu 20.04 AMIs.  
✅ **Use Filters** to refine the search for AMIs.  
✅ **Attach a Security Group** to allow SSH access.  
✅ **Use Terraform Best Practices** (avoid hardcoding, modularize resources).  
✅ **Always Destroy Unused Resources**:
```bash
terraform destroy
```

---

### **Next Steps**
In the next lectures, we will:
- **Attach an SSH key pair** for secure login.
- **Deploy into a VPC for better networking control**.
- **Use Terraform variables** for better reusability.


