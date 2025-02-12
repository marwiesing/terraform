# Terraform Conditionals and Interpolations

## Resources
- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Modules - AWS VPC](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest)
- [Terraform Conditionals](https://developer.hashicorp.com/terraform/language/expressions/conditionals)

---

## Understanding Conditionals in Terraform

### Conditional Syntax
Interpolations in Terraform can include conditionals using the following syntax:

```hcl
condition ? true_value : false_value
```

This structure evaluates `condition`, returning `true_value` if the condition is `true`, and `false_value` otherwise. This is commonly used to dynamically configure resource attributes based on environment variables or other input conditions.

### Example: Conditional `count` Attribute
A common use case is defining the number of instances to create based on an environment variable:

```hcl
resource "aws_instance" "myinstance" {
  count = var.env == "production" ? 2 : 1
  ami = "ami-123456"
  instance_type = "t2.micro"
}
```

- If `var.env` is set to `production`, Terraform creates **two instances**.
- Otherwise, only one instance is created.

### Conditional Interpolation Anywhere in Terraform
Conditionals are not limited to the `count` attribute; they can be used for any value:

```hcl
subnet_id = var.env == "prod" ? module.vpc-prod.public_subnets[0] : module.vpc-dev.public_subnets[0]
```

This assigns a subnet based on whether the environment is production (`prod`) or development (`dev`).

### Supported Operators
Terraform supports standard comparison and logical operators:

#### Equality Operators:
- `==` (equals)
- `!=` (not equals)

#### Numerical Comparison:
- `>` (greater than)
- `<` (less than)
- `>=` (greater than or equal to)
- `<=` (less than or equal to)

#### Boolean Logic:
- `&&` (AND)
- `||` (OR)
- `!` (NOT)

---

## Demo: Using Conditionals with AWS VPC Module

This demo showcases conditional usage within an AWS Virtual Private Cloud (VPC) module.

### Setting Up the Environment
Terraform provides an AWS VPC module to simplify network provisioning. Instead of defining a VPC manually, we use this module to create separate VPCs for production and development environments.

### Defining the VPCs

```hcl
module "vpc-prod" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.59.0"
  
  name = "vpc-prod"
  cidr = "10.0.0.0/16"
  
  azs             = ["${var.AWS_REGION}a", "${var.AWS_REGION}b", "${var.AWS_REGION}c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = false
  enable_vpn_gateway = false
  
  tags = {
    Terraform   = "true"
    Environment = "prod"
  }
}

module "vpc-dev" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.59.0"
  
  name = "vpc-dev"
  cidr = "10.0.0.0/16"
  
  azs             = ["${var.AWS_REGION}a", "${var.AWS_REGION}b", "${var.AWS_REGION}c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = false
  enable_vpn_gateway = false
  
  tags = {
    Terraform   = "true"
    Environment = "dev"
  }
}
```

### Provisioning AWS Instances Based on Environment

```hcl
resource "aws_instance" "example" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  # Conditional subnet selection
  subnet_id = var.env == "prod" ? module.vpc-prod.public_subnets[0] : module.vpc-dev.public_subnets[0]

  # Conditional security group selection
  vpc_security_group_ids = [var.env == "prod" ? aws_security_group.allow-ssh-prod.id : aws_security_group.allow-ssh-dev.id]

  key_name = aws_key_pair.mykeypair.key_name
}
```

### Defining Security Groups

```hcl
resource "aws_security_group" "allow-ssh-prod" {
  vpc_id      = module.vpc-prod.vpc_id
  name        = "allow-ssh"
  description = "Security group allowing SSH"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "allow-ssh-dev" {
  vpc_id      = module.vpc-dev.vpc_id
  name        = "allow-ssh"
  description = "Security group allowing SSH"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## Running the Terraform Code

### Initialize Terraform
Run the following command to download necessary modules and provider plugins:

```sh
terraform init
```

### Generate an SSH Key
To securely access the EC2 instance, generate an SSH key:

```sh
ssh-keygen -f mykey
```

### Apply the Configuration
To apply the Terraform plan and deploy the resources, run:

```sh
terraform apply
```

By default, this deploys the **production** environment. To deploy the **development** environment, specify the `env` variable:

```sh
terraform apply -var="env=dev"
```

### Managing Multiple Environments
To ensure that production and development states do not interfere with each other, maintain separate state files:

- **Option 1:** Use different directories for each environment.
- **Option 2:** Use Terraform workspaces.
- **Option 3:** Leverage modules to maintain separate infrastructure components.

---

## Summary
- Terraform conditionals allow dynamic configuration of resources.
- The `? :` syntax enables conditional evaluations.
- Conditionals can be used for `count`, `subnet_id`, `security_groups`, and more.
- The AWS VPC module simplifies networking infrastructure creation.
- Environment-based provisioning helps segregate development and production deployments efficiently.

