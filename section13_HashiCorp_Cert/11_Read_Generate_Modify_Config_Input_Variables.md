## **Reading, Generating, and Modifying Configuration in Terraform**  

### **Official Resources**  
For a deeper understanding of Terraform variables, validation, and sensitive data handling, check out these official HashiCorp resources:  
- 📌 **[Terraform Variables](https://developer.hashicorp.com/terraform/language/values/variables)** – How to define and use input variables.  
- 📌 **[Terraform Outputs](https://developer.hashicorp.com/terraform/language/values/outputs)** – Learn to retrieve data from Terraform configurations.  
- 📌 **[Terraform Locals](https://developer.hashicorp.com/terraform/language/values/locals)** – Understand how to use temporary variables in Terraform.  
- 📌 **[Terraform Sensitive Data](https://developer.hashicorp.com/terraform/language/values/variables#sensitive-variables)** – Guide on securing secrets in Terraform.  

---

## **1. Understanding Terraform Variables**  

Terraform supports three types of variables:  

| **Variable Type** | **Definition** | **Usage Example** |
|------------------|---------------|----------------|
| **Input Variables (`variable`)** | Accepts external values to parameterize Terraform configurations. | `variable "instance_type" { default = "t2.micro" }` |
| **Output Variables (`output`)** | Returns values from Terraform for use in other modules or CLI output. | `output "instance_ip" { value = aws_instance.web.public_ip }` |
| **Local Variables (`locals`)** | Defines temporary values for reuse within a module. | `locals { app_name = "my-app" }` |

---

## **2. Declaring Input Variables**  

### **Basic Input Variable**
```hcl
variable "instance_type" {
  description = "The type of EC2 instance"
  type        = string
  default     = "t2.micro"
}
```
- **`description`** → Helps document the variable’s purpose.  
- **`type`** → Ensures that the correct data type is used.  
- **`default`** → Provides a fallback value if none is supplied.  

---

## **3. Terraform Variable Types**  

Terraform supports **simple and complex data types**.

### **Simple Types**
| Type | Description | Example |
|------|------------|---------|
| `string` | Text values | `"my-text"` |
| `number` | Numeric values | `42` |
| `bool` | Boolean values | `true` or `false` |

---

### **Complex Types**
| Type | Description | Example |
|------|-------------|---------|
| `list` | Ordered collection of values | `["us-east-1a", "us-east-1b"]` |
| `set` | Unordered collection of unique values | `["t2.micro", "t3.micro"]` |
| `map` | Key-value pairs | `{ env = "prod", region = "us-east-1" }` |
| `object` | Structured key-value pairs | `{ name = "app", replicas = 3 }` |
| `tuple` | A list that can hold mixed types | `[true, 3, "hello"]` |

---

### **Example: Using Lists, Maps, and Objects**
```hcl
variable "regions" {
  type    = list(string)
  default = ["us-east-1", "us-west-2"]
}

variable "environment_tags" {
  type    = map(string)
  default = { dev = "Development", prod = "Production" }
}

variable "instance_config" {
  type = object({
    name     = string
    cpu      = number
    enabled  = bool
  })
  default = {
    name    = "web-server"
    cpu     = 2
    enabled = true
  }
}
```
This ensures type safety when passing variables.

---

## **4. Using Local Variables (`locals`)**  

Local variables are **temporary variables** used within a module for:  
✅ Simplifying complex expressions.  
✅ Avoiding repetitive code.  
✅ Improving readability.

### **Example: Using `locals` for Reusability**
```hcl
locals {
  app_name = "my-app"
  instance_type = "t2.micro"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = local.instance_type
  tags = {
    Name = local.app_name
  }
}
```
Locals improve maintainability and reduce hardcoding.

---

## **5. Defining Output Variables (`output`)**  

Outputs allow Terraform to return values after applying a configuration.

### **Example: Returning an EC2 Public IP**
```hcl
output "instance_public_ip" {
  description = "The public IP of the created instance"
  value       = aws_instance.web.public_ip
}
```
After running `terraform apply`, retrieve the output:  
```sh
terraform output instance_public_ip
```

---

## **6. Using Input and Output Variables Together**  

Terraform outputs can be **used as inputs** in another module.

### **Example: Passing an Output as an Input**  
#### **Step 1: Define Output in `network` Module**
```hcl
output "subnet_id" {
  value = aws_subnet.my_subnet.id
}
```

#### **Step 2: Use Output in Another Module**
```hcl
module "compute" {
  source    = "./compute"
  subnet_id = module.network.subnet_id
}
```
This allows for **modular infrastructure design**.

---

## **7. Variable Validation Rules (`validation`)**  

Terraform **validates variable inputs** before execution.

### **Example: Ensuring a Correct AWS AMI ID Format**
```hcl
variable "image_id" {
  type        = string
  description = "The ID of the AMI to use"
  
  validation {
    condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"
    error_message = "The AMI ID must start with 'ami-' and be at least 5 characters long."
  }
}
```
✅ Prevents errors **before** Terraform applies changes.  
✅ Saves time by catching incorrect values **early**.

---

## **8. Handling Sensitive Data (`sensitive`)**  

Terraform **hides sensitive values** in logs and CLI output.

### **Example: Marking a Variable as Sensitive**
```hcl
variable "db_password" {
  type        = string
  description = "Database password"
  sensitive   = true
}
```
✅ **Prevents passwords from being exposed in CLI output.**  

### **Example: Sensitive Output**
```hcl
output "database_password" {
  value     = var.db_password
  sensitive = true
}
```
Running `terraform apply` will show:
```
database_password = <sensitive>
```
⚠️ **Sensitive variables are still stored in Terraform state!**  

---

## **9. Best Practices for Managing Variables in Terraform**  

✅ **Use input variables for customization** – Parameterize configurations for reusability.  
✅ **Use outputs to expose key data** – Share resource attributes with other modules.  
✅ **Use locals for computed values** – Store frequently used expressions to simplify code.  
✅ **Use explicit types for variables** – Prevent incorrect data types (`type = map(string)`).  
✅ **Use validation rules** – Catch errors before execution.  
✅ **Secure sensitive data** – Use `sensitive = true` and store secrets securely.  

---

## **10. Summary of Key Terraform Commands for Variables**  

| **Command**                | **Description** |
|----------------------------|----------------|
| `terraform output`         | Displays all Terraform output values. |
| `terraform output variable` | Shows the value of a specific output. |
| `terraform console`        | Allows interactive testing of Terraform expressions. |

For further learning, check out:  
- **[Terraform Variables](https://developer.hashicorp.com/terraform/language/values/variables)**  
- **[Terraform Outputs](https://developer.hashicorp.com/terraform/language/values/outputs)**  
- **[Terraform Sensitive Data](https://developer.hashicorp.com/terraform/language/values/variables#sensitive-variables)**  

---

## **Key Takeaways**  

✅ **Terraform supports input (`variable`), output (`output`), and local (`locals`) variables.**  
✅ **Input variables** parameterize configurations.  
✅ **Output variables** expose resource attributes.  
✅ **Locals** store computed values to reduce redundancy.  
✅ **Validation rules prevent misconfigurations.**  
✅ **Sensitive data handling protects secrets but still stores them in state files.**  

This **structured version** provides **clear explanations, real-world use cases, and best practices**, making it a **solid study guide**. 🚀 Let me know if you need any refinements!