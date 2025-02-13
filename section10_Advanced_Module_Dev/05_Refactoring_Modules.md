# Advanced Module Development: Refactoring Modules

## References
- [Terraform Moved Blocks](https://developer.hashicorp.com/terraform/language/modules/moved)
- [Terraform for_each and count](https://developer.hashicorp.com/terraform/language/meta-arguments/for_each)
- [Terraform State Manipulation](https://developer.hashicorp.com/terraform/cli/state/mv)

## Introduction
Refactoring modules is an essential part of maintaining Terraform infrastructure as requirements evolve. When modifying an existing module, renaming resources, or changing how resources are created, **moved blocks** help prevent unnecessary resource destruction and recreation. Terraform 1.1 introduced **moved blocks** to streamline renaming and restructuring resources and modules.

## When to Use Moved Blocks
Moved blocks ensure that changes in resource/module names or organization **do not force Terraform to destroy and recreate resources**. Use moved blocks when:
- Renaming a resource or module.
- Adding `count` or `for_each` to an existing resource or module.
- Moving a resource into a module or restructuring modules.
- Ensuring backward compatibility for existing Terraform state.

## Renaming a Resource
If a resource name changes (e.g., renaming an AWS instance from `web` to `web-new`), Terraform treats it as a new resource, leading to deletion and recreation. Instead, use a **moved block**:

```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.web-new
}
```
This ensures the state file updates automatically without affecting the actual infrastructure.

## Adding `count` or `for_each`
When converting a **static resource** into a **count-based or for_each-based** resource, Terraform must understand where the original resource maps to within the new structure.

### Example: Adding `count`
Before refactoring:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```
After refactoring:
```hcl
resource "aws_instance" "web" {
  count         = 2
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```
To maintain continuity, use a **moved block**:
```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.web[0]
}
```
This ensures the original instance is mapped to the first instance (`web[0]`), avoiding unnecessary recreation.

### Example: Switching to `for_each`
Instead of a count-based approach, `for_each` can be used for more control.
```hcl
variable "architectures" {
  type = map(string)
  default = {
    x86_64 = "ami-x86_64"
    arm64  = "ami-arm64"
  }
}

resource "aws_instance" "web" {
  for_each      = var.architectures
  ami           = each.value
  instance_type = "t2.micro"
}
```
To migrate an existing `aws_instance.web` to a `for_each` structure:
```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.web["x86_64"]
}
```
Terraform will recognize the change and preserve the resource, creating additional instances as needed.

## Moving a Resource into a Module
If refactoring involves moving a resource into a module, Terraform must know where the resource has been relocated.

### Before Moving Into a Module
```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

### After Moving to a Module
```hcl
module "instances" {
  source = "./modules/instances"
}
```
Within `./modules/instances/main.tf`:
```hcl
resource "aws_instance" "web_instance" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```
A **moved block** prevents Terraform from destroying and recreating the resource:
```hcl
moved {
  from = aws_instance.web
  to   = module.instances.aws_instance.web_instance
}
```

## Moving a Module
Modules can also be renamed or restructured similarly.

### Example: Renaming a Module
Before refactoring:
```hcl
module "vpc" {
  source = "./modules/vpc"
}
```
After renaming:
```hcl
module "networking" {
  source = "./modules/vpc"
}
```
A **moved block** ensures continuity:
```hcl
moved {
  from = module.vpc
  to   = module.networking
}
```

### Example: Adding `count` to a Module
Before using `count`:
```hcl
module "vpc" {
  source = "./modules/vpc"
}
```
After adding `count`:
```hcl
module "vpc" {
  count  = 2
  source = "./modules/vpc"
}
```
To avoid resource recreation:
```hcl
moved {
  from = module.vpc
  to   = module.vpc[0]
}
```

## Handling Legacy Move Blocks
Over time, modules may accumulate multiple move blocks. Keeping them ensures backward compatibility, preventing unintended deletions for users who have not updated their module state. However, move blocks **can be removed** once all users have upgraded and no longer need them.

## Limitations of Move Blocks
- **Move blocks only apply within Terraform modules**.
- **External modules (e.g., Terraform Registry modules) do not support move blocks**.
- **Use `terraform state mv`** for renaming or restructuring root module resources.

### Using `terraform state mv`
For manual state manipulation:
```sh
terraform state mv aws_instance.web module.instances.aws_instance.web_instance
```
Use this method when working outside of a module, or for complex migrations not supported by move blocks.

## Conclusion
- **Move blocks provide a structured way to refactor modules without destroying resources**.
- **They support renaming resources, adding `count` or `for_each`, and moving resources into modules**.
- **State migrations outside modules still require `terraform state mv`**.
- **Removing move blocks too soon can break state for users who haven’t updated yet**.
- **Terraform 1.1+ is required to use move blocks**.

By using move blocks effectively, Terraform developers can refactor modules safely while preserving existing infrastructure and ensuring smooth transitions.