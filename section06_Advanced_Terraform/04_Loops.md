# Terraform Loops: for and for_each (Starting from Terraform 0.12)

## Resources:
- [Terraform Official Documentation](https://developer.hashicorp.com/terraform/language)
- [AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Dynamic Blocks](https://developer.hashicorp.com/terraform/language/expressions/dynamic-blocks)
- [Maps and Lists in Terraform](https://developer.hashicorp.com/terraform/language/expressions/type-constraints)

## Introduction
Starting from **Terraform 0.12**, two looping mechanisms were introduced: **for loops** and **for_each loops**. These looping constructs allow you to iterate over lists, maps, and sets to transform input data dynamically and generate resources more efficiently.

- **for loops** are used primarily for transforming data within expressions, such as generating lists or maps dynamically.
- **for_each loops** are used to iterate over a map or set to create multiple **resource instances** or **nested blocks**.

---
## **for Loops in Terraform**

The `for` expression in Terraform is useful when you need to transform input data and output it in a different format, whether as a list or a map.

### **Basic for Loop Syntax**
```hcl
[for item in list : transformation]
```
This structure consists of:
- `for item in list`: Iterates over each element in the list.
- `transformation`: Applies a transformation to each element.
- Square brackets `[]`: Denotes that the output will be a list.

### **Example 1: Transforming a List**
#### **Convert a List of Strings to Uppercase**
```hcl
variable "my_list" {
  default = ["apple", "banana", "cherry"]
}

output "uppercase_list" {
  value = [for s in var.my_list : upper(s)]
}
```
**Expected Output:**
```json
["APPLE", "BANANA", "CHERRY"]
```

### **Example 2: Iterating Over Numbers**
#### **Incrementing Each Number in a List**
```hcl
variable "numbers" {
  default = [1, 2, 3, 4, 5]
}

output "incremented_numbers" {
  value = [for n in var.numbers : n + 1]
}
```
**Expected Output:**
```json
[2, 3, 4, 5, 6]
```

### **Example 3: Transforming a Map**
#### **Switching Keys and Values in a Map**
```hcl
variable "fruit_prices" {
  default = {
    "apple" = 5,
    "banana" = 3,
    "cherry" = 10
  }
}

output "switched_map" {
  value = { for k, v in var.fruit_prices : v => k }
}
```
**Expected Output:**
```json
{
  "5": "apple",
  "3": "banana",
  "10": "cherry"
}
```

### **Example 4: Merging and Modifying Maps**
In real-world scenarios, `for` loops are used to modify metadata dynamically.

#### **Merging Two Maps and Converting Values to Lowercase**
```hcl
variable "project_tags" {
  default = {
    "Component" = "Frontend",
    "Environment" = "Production"
  }
}

output "formatted_tags" {
  value = { for k, v in var.project_tags : k => lower(v) }
}
```
**Expected Output:**
```json
{
  "Component": "frontend",
  "Environment": "production"
}
```

---
## **for_each Loops in Terraform**
Unlike `for` loops, which transform data, `for_each` loops **create multiple instances of a resource** or **repeat nested blocks dynamically**.

### **Basic for_each Syntax**
```hcl
for_each = var.some_map
```
This structure allows Terraform to iterate over each key-value pair in the provided map or set.

### **Example 1: Using for_each in a Resource**
#### **Creating Multiple Security Groups Dynamically**
```hcl
variable "security_rules" {
  default = {
    "ssh"  = 22,
    "https" = 443
  }
}

resource "aws_security_group_rule" "example" {
  for_each    = var.security_rules
  type        = "ingress"
  from_port   = each.value
  to_port     = each.value
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
```
This dynamically creates **two security group rules**, one for SSH (22) and one for HTTPS (443).

### **Example 2: Dynamic Nested Blocks**
#### **Using for_each in Dynamic Blocks (Ingress Rules)**
```hcl
variable "ports" {
  default = {
    "22"  = ["192.168.1.0/24", "10.0.0.0/16"],
    "443" = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "example" {
  name = "dynamic-sg"

  dynamic "ingress" {
    for_each = var.ports
    content {
      from_port   = ingress.key
      to_port     = ingress.key
      protocol    = "tcp"
      cidr_blocks = ingress.value
    }
  }
}
```
This example dynamically creates **multiple ingress rules**, where:
- **Port 22** allows access only from specific IP ranges.
- **Port 443** is open to the world.

### **Example 3: Assigning Tags with for_each**
#### **Dynamically Adding Tags to AWS Resources**
```hcl
variable "tags" {
  default = {
    "Environment" = "dev",
    "Owner" = "admin"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = { for k, v in var.tags : k => upper(v) }
}
```
This converts all tag values to uppercase dynamically.

---
## **Key Takeaways**
| Feature        | `for` Loop | `for_each` Loop |
|---------------|-----------|---------------|
| Used for      | Transforming data | Creating resources or blocks |
| Outputs       | List or map | Multiple instances of a resource or block |
| Works with    | Lists, maps | Maps, sets |
| Use Case      | Data transformation, merging, or filtering | Dynamic resource creation or block repetition |

Terraform loops provide powerful mechanisms to automate repetitive tasks, whether through transforming data using `for` or dynamically creating resources with `for_each`. Mastering these concepts will significantly enhance infrastructure management efficiency.

