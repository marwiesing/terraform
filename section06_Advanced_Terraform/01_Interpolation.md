# Advanced Terraform Usage

## References:
- [Terraform Interpolation Syntax](https://developer.hashicorp.com/terraform/language/expressions)
- [Terraform Functions](https://developer.hashicorp.com/terraform/language/functions)
- [Terraform Variables](https://developer.hashicorp.com/terraform/language/values/variables)
- [Terraform Modules](https://developer.hashicorp.com/terraform/language/modules)

---

## Interpolation in Terraform
Interpolation is the process of embedding expressions within string values in Terraform. Terraform provides an interpolation syntax that allows users to reference variables, perform simple operations, and conditionally set values within configuration files.

### Syntax
Interpolation expressions in Terraform are enclosed within `${}`. The basic syntax is:
```hcl
${expression}
```

Expressions inside the interpolation can include:![variables](image.png)
![various](image-1.png)
- **Variables**: `${var.name}`
- **Resource attributes**: `${aws_instance.example.id}`
- **Data sources**: `${data.aws_ami.ubuntu.id}`
- **Functions**: `${lookup(var.amis, var.aws_region)}`
- **Math operations**: `${2 + 3 * 4}`
- **Conditionals**: `${var.env == "prod" ? "high-performance" : "standard"}`

### Referencing Resources
In Terraform, resources can be referenced using the pattern:
```hcl
${resource_type.resource_name.attribute}
```
For example, referencing the ID of an AWS instance:
```hcl
${aws_instance.my_instance.id}
```
Additional attributes include:
- `${aws_instance.my_instance.private_ip}`
- `${aws_instance.my_instance.public_ip}`

To see all available attributes for a resource, refer to the official Terraform provider documentation.

### Using Data Sources
Data sources allow Terraform to fetch information from existing resources. The syntax for referencing a data source is:
```hcl
${data.data_source_type.resource_name.attribute}
```
Example:
```hcl
${data.aws_ami.ubuntu.id}
```
where `data.aws_ami.ubuntu.id` fetches the ID of an Ubuntu AMI from AWS.

## Terraform Variables
Terraform supports different types of variables that can be interpolated into configurations.

### String Variables
String variables are the most basic type and can be referenced using:
```hcl
${var.name}
```
Example:
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

resource "aws_instance" "example" {
  instance_type = var.instance_type
}
```

### Map Variables
Maps store key-value pairs and are useful for defining configurations based on region or environment.
```hcl
variable "amis" {
  type = map(string)
  default = {
    us-east-1 = "ami-12345678"
    eu-west-1 = "ami-87654321"
  }
}
```
Referencing a map value:
```hcl
${var.amis["us-east-1"]}
```
Alternatively, using the `lookup` function:
```hcl
${lookup(var.amis, var.aws_region)}
```

### List Variables
Lists store multiple values that can be accessed via an index.
```hcl
variable "subnets" {
  type    = list(string)
  default = ["subnet-abc123", "subnet-def456"]
}
```
Accessing list values:
```hcl
${var.subnets[0]}  # First element
${var.subnets[1]}  # Second element
```
To create a comma-separated list of all elements:
```hcl
${join(",", var.subnets)}
```

### Choosing Between Lists and Maps
- **Use a List** when values are ordered and referenced by position.
- **Use a Map** when values need meaningful keys (e.g., AMIs per region).

## Module Outputs
Modules allow reusable Terraform configurations. Outputs from modules can be accessed as:
```hcl
module.module_name.output_name
```
Example:
```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
}

output "vpc_id" {
  value = module.vpc.vpc_id
}
```

## Using Count for Resource Replication
Terraform allows creating multiple instances of a resource using the `count` parameter.
```hcl
resource "aws_instance" "example" {
  count = 3
  ami   = "ami-12345678"
  instance_type = "t2.micro"
  tags = {
    Name = "example-${count.index}"
  }
}
```
The `count.index` variable helps create unique names for each instance.

## Path Information
Terraform provides built-in path variables:
```hcl
path.cwd    # Current working directory
path.module # Path of the module where the file is located
path.root   # Path of the root module
```
These are useful for working with file paths in templates or scripts.

## Meta Information
Terraform provides metadata functions:
```hcl
terraform.workspace  # Current workspace
```
Terraform Workspaces allow environment isolation. The default workspace is `default`, but you can create and switch between workspaces.
```sh
tf workspace new dev
tf workspace select dev
```
Using workspace in configuration:
```hcl
${terraform.workspace == "prod" ? "high-performance" : "standard"}
```

## Math Operations in Terraform
Terraform supports basic arithmetic operations:
```hcl
2 + 3 * 4    # Results in 14
10 / 2       # Results in 5
15 % 4       # Results in 3
```
You can also use mathematical functions such as:
```hcl
max(10, 20)  # Returns 20
min(5, 8)    # Returns 5
```

## Conclusion
Terraform's interpolation features allow for powerful and flexible configurations. By understanding variables, resource references, functions, and meta-information, you can write more dynamic and reusable Terraform scripts.

