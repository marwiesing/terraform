Here's a detailed and enhanced version of the transcript with additional examples, explanations, and references to resources:

---

## **Terraform Input Variables – A Deep Dive**
#### **Resources**
- 🔗 [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/connect_instance)
- 🔗 [Ubuntu AWS Guide](https://documentation.ubuntu.com/aws/en/latest/)
- 🔗 [AWS Free Tier Pricing](https://aws.amazon.com/free/)
- 🔗 [Terraform CLI Documentation](https://developer.hashicorp.com/terraform/cli)

---

### **Introduction to Input Variables**
Terraform input variables provide a flexible way to define values that can be reused across different environments. Rather than hardcoding values, we can declare variables and set their values dynamically.

#### **Basic Syntax of Input Variables**
Variables in Terraform are declared using the `variable` keyword, followed by the variable name. They can have optional attributes such as `type` and `default`.

Example:
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```

- The `type` attribute enforces a specific data type (e.g., `string`, `number`, `list`, `map`).
- The `default` attribute sets a default value if no explicit value is provided.

---

### **Using Variables in Terraform Configurations**
Once declared, we can reference the variable inside our Terraform configuration files using the `var.<variable_name>` syntax.

Example usage in an AWS EC2 instance:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"  # Example AMI ID
  instance_type = var.instance_type

  tags = {
    Name = "WebServer"
  }
}
```
Here, `var.instance_type` retrieves the value of the `instance_type` variable.

---

### **Structuring Terraform Files for Clarity**
While Terraform does not enforce specific file names, it is common practice to structure configuration files in a modular way:
- **`main.tf`** → Contains the main Terraform resources.
- **`variables.tf`** → Stores variable declarations.
- **`terraform.tfvars`** → Contains variable values.
- **`provider.tf`** → Defines provider configurations.
- **`datasource.tf`** → Stores data sources.

Example:
```
.
├── main.tf
├── variables.tf
├── terraform.tfvars
├── provider.tf
├── datasource.tf
```

This structure improves maintainability, especially for larger projects.

---

### **Setting Variable Values**
There are multiple ways to assign values to variables:

#### **1. Default Value in `variables.tf`**
If a default value is set in `variables.tf`, Terraform will use it unless overridden.

Example:
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```

#### **2. Using `terraform.tfvars`**
We can create a `terraform.tfvars` file and define values there.

Example `terraform.tfvars`:
```hcl
instance_type = "t3.micro"
```
Terraform automatically loads `terraform.tfvars` when running `terraform apply`.

#### **3. Using Command-Line Flags**
We can pass values directly using the `-var` flag:

```sh
terraform apply -var="instance_type=t3.medium"
```

#### **4. Using Environment Variables**
Terraform allows setting environment variables prefixed with `TF_VAR_`:

```sh
export TF_VAR_instance_type=t4g.micro
terraform apply
```
Terraform will recognize `TF_VAR_instance_type` and use it.

---

### **Overriding Variables Dynamically**
There are cases where we may need to override variables dynamically, such as switching between different environments (e.g., `dev`, `qa`, `prod`).

#### **Using Different `.tfvars` Files**
We can define separate `.tfvars` files for each environment.

Example `dev.tfvars`:
```hcl
instance_type = "t3.micro"
```

Example `prod.tfvars`:
```hcl
instance_type = "t4g.large"
```

To apply a specific environment configuration:
```sh
terraform apply -var-file="prod.tfvars"
```
This allows us to reuse the same Terraform code across different environments while customizing settings.

---

### **Complex Data Types in Variables**
Terraform supports different data types for variables. Let's explore them.

#### **1. String (Default)**
A simple text-based value:
```hcl
variable "environment" {
  type    = string
  default = "development"
}
```

#### **2. Number**
A numeric value, either integer or decimal:
```hcl
variable "instance_count" {
  type    = number
  default = 2
}
```

#### **3. Boolean**
A true/false value:
```hcl
variable "enable_monitoring" {
  type    = bool
  default = true
}
```

#### **4. List (Array)**
A collection of values:
```hcl
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b"]
}
```
Usage:
```hcl
resource "aws_instance" "web" {
  count         = length(var.availability_zones)
  availability_zone = var.availability_zones[count.index]
}
```

#### **5. Map (Key-Value Pairs)**
A collection of key-value pairs:
```hcl
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t2.micro"
    prod = "t3.large"
  }
}
```
Usage:
```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_types["dev"]
}
```

---

### **Terraform Console for Debugging Variables**
Terraform provides a built-in console for inspecting variable values.

Start the console:
```sh
terraform console
```

Query a variable:
```hcl
> var.instance_type
"t3.micro"
```

Query a map:
```hcl
> var.instance_types["prod"]
"t3.large"
```

Exit the console:
```sh
exit
```

---

### **Handling Missing Variables**
If a required variable is not provided, Terraform prompts for input.

Example:
```sh
terraform apply
var.instance_type
  Enter a value: t2.micro
```

To avoid manual input, always define values using one of the methods above.

---

### **Best Practices for Variables**
1. **Use `.tfvars` files for environment-specific values** – Helps manage configurations for `dev`, `qa`, and `prod` environments.
2. **Group related variables together in `variables.tf`** – Improves readability.
3. **Use type constraints (`string`, `number`, `list`, etc.)** – Prevents unintended inputs.
4. **Keep sensitive values secure** – Avoid hardcoding secrets; use `terraform.tfvars` with `.gitignore` or tools like AWS Secrets Manager.
5. **Use meaningful names** – Helps in understanding variable purpose.

---

### **Conclusion**
Terraform input variables allow us to create reusable and dynamic infrastructure configurations. By leveraging different variable types, `.tfvars` files, and command-line overrides, we can efficiently manage deployments across multiple environments.

#### **Key Takeaways**
✅ Use variables instead of hardcoding values.  
✅ Store variables in `.tfvars` or pass them via `-var-file`.  
✅ Use `terraform console` for debugging.  
✅ Organize files properly for better maintainability.  
✅ Use maps and lists for scalable configurations.  

For more details, refer to the [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/connect_instance).

---

Would you like me to include a real-world example of managing Terraform variables in a multi-account AWS setup? 🚀