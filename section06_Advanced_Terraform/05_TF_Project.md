# Terraform Project Structure: Best Practices and Implementation

## References
- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices Guide](https://www.terraform.io/docs/cloud/guides/recommended-practices.html)
- [AWS Organizations Setup](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts.html)
- [Terraform Module Structure](https://developer.hashicorp.com/terraform/language/modules)

---

## Introduction
When implementing Terraform in a production environment, it is crucial to adopt a well-organized project structure. A properly structured Terraform project facilitates:

- Environment isolation for development and production.
- Improved security by using separate AWS accounts for development, production, and billing.
- Efficient Terraform execution by reducing the scope of `terraform apply` runs.
- Reusability through modules, reducing duplication and maintenance effort.

This guide outlines a recommended Terraform project structure and demonstrates its practical implementation.

---

## Environment Isolation
To maintain isolation and security, Terraform configurations should be structured to separate development and production environments:

- **Development Environment**: Used for testing infrastructure changes before applying them to production.
- **Production Environment**: The live system where only verified changes should be deployed.
- **Billing Account**: Centralized AWS billing management.

AWS Organizations can be used to create and manage multiple accounts efficiently.

---

## Terraform Project Structure
A recommended Terraform project structure consists of the following:

```
terraform-course/
│── modules/
│   ├── vpc/
│   │   ├── vpc.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   ├── instances/
│   │   ├── instance.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│── envs/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── provider.tf
│   │   ├── variables.tf
│   ├── prod/
│   │   ├── main.tf
│   │   ├── provider.tf
│   │   ├── variables.tf
│── terraform.tfvars
│── backend.tf
│── versions.tf
```

### Explanation of Structure
- **Modules (`modules/`)**
  - **`vpc/`**: Manages the Virtual Private Cloud (VPC) resources.
  - **`instances/`**: Defines EC2 instances and their configurations.
- **Environments (`envs/`)**
  - **`dev/` and `prod/`**: Separate directories for development and production, each containing its own Terraform configuration.
- **`terraform.tfvars`**: Defines variable values.
- **`backend.tf`**: Configures the remote backend (e.g., S3, Terraform Cloud) for state management.
- **`versions.tf`**: Specifies required Terraform and provider versions.

---

## Implementing the Project Structure
### 1. Define Modules
#### VPC Module (`modules/vpc/vpc.tf`)
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.11.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
}
```

#### Instances Module (`modules/instances/instance.tf`)
```hcl
resource "aws_instance" "example" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = var.subnet_id
  key_name      = aws_key_pair.mykeypair.key_name
}
```

### 2. Configure Environments
#### Development (`envs/dev/main.tf`)
```hcl
module "vpc" {
  source      = "../../modules/vpc"
  aws_region  = "eu-west-1"
}

module "instances" {
  source    = "../../modules/instances"
  subnet_id = module.vpc.public_subnets[0]
}
```

#### Production (`envs/prod/main.tf`)
```hcl
module "vpc" {
  source      = "../../modules/vpc"
  aws_region  = "us-east-1"
}

module "instances" {
  source    = "../../modules/instances"
  subnet_id = module.vpc.public_subnets[0]
}
```

### 3. Configure Terraform State
#### Backend Configuration (`backend.tf`)
```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}
```

---

## Deployment Workflow
1. **Initialize Terraform**
   ```sh
   terraform init
   ```

2. **Plan the Deployment**
   ```sh
   terraform plan -var-file=terraform.tfvars
   ```

3. **Apply the Configuration**
   ```sh
   terraform apply -var-file=terraform.tfvars
   ```

4. **Check Deployed Resources**
   ```sh
   terraform show
   ```

5. **Destroy the Infrastructure (if needed)**
   ```sh
   terraform destroy -var-file=terraform.tfvars
   ```

---

## Conclusion
Adopting a structured Terraform project organization is crucial for managing complex infrastructure efficiently. By leveraging modules and separating environments, teams can maintain best practices, improve security, and optimize Terraform execution. Additionally, using remote backends for state management enhances collaboration and reliability.

Following this structure ensures:
- Improved maintainability.
- Reduced risk of production errors.
- Faster execution times.

This guide provides a solid foundation for structuring Terraform projects effectively in real-world scenarios.

