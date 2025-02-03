Here's an enhanced and more detailed version of the transcript with additional explanations, examples, and references to relevant resources:

---

# **Expanding Our Terraform Code – VPC, Subnets, and Public IPs**
#### **Resources**
- 🔗 [Terraform AWS VPC Module](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest)
- 🔗 [Terraform AWS Instance Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)
- 🔗 [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
- 🔗 [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/index.html)

---

## **Introduction**
In this lecture, we will enhance our existing Terraform configuration by introducing:
✅ **A VPC** (Virtual Private Cloud)  
✅ **A security group** (to define network rules)  
✅ **A public IP address** (to allow remote access)  

Instead of defining a VPC manually (which requires multiple resources), we will leverage the **Terraform AWS VPC module**. This will simplify the process while keeping our configuration readable.

---

## **Using the Terraform AWS VPC Module**
Creating a **VPC from scratch** requires defining multiple resources:
- **VPC**
- **Subnets** (public & private)
- **Internet Gateway**
- **Route Tables**
- **Network ACLs & Security Groups**
- **NAT Gateway (optional, but costs money)**

Using the **Terraform AWS VPC module**, we can create a VPC in a few lines of code.

### **Step 1: Adding the VPC Module**
Create a new file called **`vpc.tf`** and add the following:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway     = false  # Disabling NAT to avoid extra costs
  enable_vpn_gateway     = false  # We don’t need a VPN Gateway

  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```
✅ This creates:
- A **VPC** named `my-vpc`.
- **Three public and three private subnets** in different availability zones.
- **No NAT Gateway** (to avoid AWS costs).

---

## **Step 2: Defining the AWS Region as a Variable**
Create a new file **`variables.tf`**:

```hcl
variable "aws_region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-east-1"
}
```

Now, we can reference this variable throughout our configuration using **`var.aws_region`**.

---

## **Step 3: Initializing and Applying the VPC Module**
### **1. Initialize Terraform**
Since we are using a module, we need to initialize Terraform:
```sh
terraform init
```
This will **download** the VPC module from the Terraform registry.

### **2. Apply Only the VPC Module**
If we only want to test the VPC, we can use **resource targeting**:
```sh
terraform apply -target=module.vpc
```
✅ **Note:** Terraform warns that `-target` should only be used in exceptional cases.

### **3. Apply All Changes**
Once satisfied, apply everything:
```sh
terraform apply
```

Terraform will create **22 resources**, including subnets, route tables, and an internet gateway.

---

## **Step 4: Outputting Public Subnet Information**
We need to find out which **subnet IDs** were created so we can launch our EC2 instance inside them.

### **Check Available Outputs**
The **VPC module** exports useful outputs, such as:
- `module.vpc.public_subnets`
- `module.vpc.private_subnets`
- `module.vpc.vpc_id`

To view them:
```sh
terraform output
```

### **Explicitly Define an Output for Public Subnets**
Create **`outputs.tf`**:

```hcl
output "public_subnets" {
  description = "List of public subnet IDs"
  value       = module.vpc.public_subnets
}
```
After applying Terraform, the public subnet IDs will be displayed:
```sh
public_subnets = [
  "subnet-0123abc456def7890",
  "subnet-0987fed654cba3210",
  "subnet-0a1b2c3d4e5f67890"
]
```
Now, we can use one of these subnets to launch our EC2 instance.

---

## **Step 5: Launching an EC2 Instance in the VPC**
Modify your **`main.tf`** to launch an instance inside one of the **public subnets**:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"  # Ubuntu AMI ID (change as needed)
  instance_type = "t2.micro"
  subnet_id     = module.vpc.public_subnets[0]  # Launch in the first public subnet

  associate_public_ip_address = true  # Ensure the instance gets a public IP

  tags = {
    Name = "WebServer"
  }
}
```

---

## **Step 6: Ensuring Public IP Assignment**
If your instance doesn’t get a public IP:
1. Check if the **subnet has public IP auto-assignment enabled**:
   ```hcl
   map_public_ip_on_launch = true
   ```
   Add this inside **`module "vpc"`** in `vpc.tf`:
   ```hcl
   enable_dns_hostnames    = true
   enable_dns_support      = true
   map_public_ip_on_launch = true
   ```

2. If the **VPC doesn’t automatically assign public IPs**, set it manually in the **EC2 instance**:
   ```hcl
   associate_public_ip_address = true
   ```

### **Re-apply Terraform to Fix Public IP Issue**
```sh
terraform apply
```
This will:
✅ Update the VPC subnet settings.  
✅ Ensure new EC2 instances receive a **public IP**.  
✅ Recreate the EC2 instance.

---

## **Step 7: Handling EC2 Instance Recreation**
Since **Terraform doesn’t detect subnet changes as an EC2 attribute**, we need to manually **taint** the instance:

```sh
terraform taint aws_instance.web
terraform apply
```
This forces Terraform to:
✅ **Destroy and recreate** the EC2 instance.  
✅ Assign a **new public IP**.  

---

## **Key Takeaways**
✅ **VPC modules simplify networking setups** in Terraform.  
✅ **Public subnets** must have `map_public_ip_on_launch = true` to assign IPs automatically.  
✅ **EC2 instances can manually request a public IP** with `associate_public_ip_address = true`.  
✅ **Tainting an instance forces recreation** when needed.  
✅ **Outputs allow us to retrieve subnet IDs dynamically** for launching instances.  

---

## **Next Steps**
In the next lecture, we will:
- Configure **security groups** to allow SSH access.
- Assign a **key pair** to log into our EC2 instance.
- Secure our infrastructure properly.

🚀 **Stay tuned!** 🚀