# Terraform Built-in Functions

## References
For a complete list of Terraform built-in functions, refer to the official documentation:
- [Terraform Functions Reference](https://developer.hashicorp.com/terraform/language/functions)
- [Interpolation Syntax](https://developer.hashicorp.com/terraform/language/expressions)

## Overview
Terraform provides several built-in functions that allow users to manipulate strings, lists, maps, and other data structures dynamically within Terraform configurations. These functions enhance reusability, maintainability, and flexibility when defining infrastructure.

Functions follow the syntax:
```
${function_name(ARG1, ARG2, ...)}
```
However, in Terraform 0.12 and later, functions are used without the `${}` syntax inside expressions.

## Commonly Used Terraform Functions

### 1. **File Function**
Reads the contents of a file.
```hcl
variable "public_key_path" {
  default = "~/.ssh/id_rsa.pub"
}

output "public_key" {
  value = file(var.public_key_path)
}
```

### 2. **Base Name**
Returns the last part of a file path.
```hcl
output "filename" {
  value = basename("/home/user/myfile.txt") # Returns "myfile.txt"
}
```

### 3. **Coalesce & CoalesceList**
Returns the first non-empty value or list.
```hcl
output "first_non_empty" {
  value = coalesce("", "", "hello", "world") # Returns "hello"
}
```

### 4. **Element**
Retrieves an element from a list at a given index.
```hcl
variable "subnets" {
  default = ["subnet-1", "subnet-2", "subnet-3"]
}

output "first_subnet" {
  value = element(var.subnets, 0) # Returns "subnet-1"
}
```

### 5. **Format & FormatList**
Formats strings with placeholders.
```hcl
output "formatted_string" {
  value = format("server-%03d", 1) # Returns "server-001"
}
```

### 6. **Index**
Finds the index of a given element in a list.
```hcl
variable "subnets" {
  default = ["subnet-1", "subnet-2", "subnet-3"]
}

output "index_of_subnet_2" {
  value = index(var.subnets, "subnet-2") # Returns 1
}
```

### 7. **Join**
Concatenates elements of a list into a string with a specified delimiter.
```hcl
variable "amis" {
  default = ["ami-123", "ami-456", "ami-789"]
}

output "joined_amis" {
  value = join(",", var.amis) # Returns "ami-123,ami-456,ami-789"
}
```

### 8. **Lookup**
Performs a key lookup in a map.
```hcl
variable "region_amis" {
  default = {
    "us-east-1" = "ami-123456"
    "us-west-1" = "ami-654321"
  }
}

output "ami_for_us_west" {
  value = lookup(var.region_amis, "us-west-1", "ami-default")
}
```

### 9. **Lower & Upper**
Converts string case.
```hcl
output "lowercase" {
  value = lower("HELLO") # Returns "hello"
}
```

```hcl
output "uppercase" {
  value = upper("hello") # Returns "HELLO"
}
```

### 10. **Merge**
Combines multiple maps into one.
```hcl
variable "map1" {
  default = {"key1" = "value1"}
}
variable "map2" {
  default = {"key2" = "value2"}
}

output "merged_map" {
  value = merge(var.map1, var.map2) # Returns {"key1" = "value1", "key2" = "value2"}
}
```

### 11. **Replace**
Searches and replaces substrings.
```hcl
output "replaced_string" {
  value = replace("terraform", "a", "@") # Returns "terr@form"
}
```

### 12. **Split**
Splits a string into a list based on a delimiter.
```hcl
output "split_list" {
  value = split(",", "a,b,c,d") # Returns ["a", "b", "c", "d"]
}
```

### 13. **Substring**
Extracts a portion of a string.
```hcl
output "substring_example" {
  value = substr("abcdef", 2, 3) # Returns "cde"
}
```

### 14. **Timestamp**
Returns the current timestamp.
```hcl
output "current_time" {
  value = timestamp()
}
```

### 15. **UUID**
Generates a unique identifier.
```hcl
output "unique_id" {
  value = uuid()
}
```

## Demo: Using Functions in Terraform Console
Terraform provides an interactive console (`terraform console`) where you can experiment with these functions.

### Example Commands
```sh
terraform console
> file("~/.ssh/id_rsa.pub")
> join(",", ["a", "b", "c"])
> split(",", "subnet-1,subnet-2,subnet-3")
> lookup({"key1" = "value1", "key2" = "value2"}, "key1")
> merge({"a" = 1}, {"b" = 2})
```

### Practical Use Cases
- Extract a specific subnet dynamically from a list.
- Format resource names with consistent numbering.
- Lookup AMIs based on the region dynamically.
- Split security group IDs stored as a string into a list.
- Merge variables containing different configuration values.

## Summary
Terraform built-in functions are crucial for manipulating variables and resources efficiently. By leveraging these functions, you can write dynamic, reusable, and efficient Terraform configurations. The Terraform console allows experimenting with these functions before applying them in actual infrastructure code.

For a deeper dive, always refer to the [official Terraform function documentation](https://developer.hashicorp.com/terraform/language/functions).

