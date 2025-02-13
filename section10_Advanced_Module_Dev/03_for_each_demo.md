# Terraform Demo: Using for_each with Modules

## Introduction
In this demo, we will explore how to use **for_each** in Terraform modules by building a simple **AWS Parameter Store** module from scratch. This will demonstrate how to iterate over a map and dynamically create resources using **for_each** at the module level.

## Setting Up the Module
The demo starts with creating a directory for the module (`module-for-each`) and defining essential Terraform configuration files.

### 1. **Defining the AWS Provider**
The first step is to declare the **AWS provider** in `provider.tf`:
```hcl
provider "aws" {
  region = "eu-west-1"
}
```

### 2. **Creating Local Variables for Static Parameters**
A **local map** containing key-value pairs for parameter names and values is defined in `parameters.tf`:
```hcl
locals {
  my_parameters = {
    environment = "development"
    version     = "1.0"
    mykey       = "myvalue"
  }
}
```
Using a **map** allows easy iteration and management of multiple key-value pairs.

### 3. **Using for_each to Loop Over the Map in a Module**
To dynamically create AWS SSM parameters, we use **for_each** in `parameters.tf`:
```hcl
module "parameters" {
  for_each = local.my_parameters
  source   = "./ssm-parameter"
  name     = each.key
  value    = each.value
}
```
Terraform iterates over the map, creating one module instance per key-value pair.

## Creating the SSM Parameter Module
Inside the `ssm-parameter` directory, we define the necessary Terraform files:

### 1. **Defining Variables** (`variables.tf`)
```hcl
variable "name" {}
variable "value" {}
```
These variables will be passed from the parent module.

### 2. **Creating AWS SSM Parameter Resource** (`ssm-parameter.tf`)
```hcl
resource "aws_ssm_parameter" "parameter" {
  name  = var.name
  type  = "String"
  value = var.value
}
```
This resource creates an **AWS Systems Manager (SSM) Parameter Store** entry for each item in `local.my_parameters`.

### 3. **Defining Outputs** (`output.tf`)
```hcl
output "arn" {
  value = aws_ssm_parameter.parameter.arn
}
```
Each instance of the module will output the **Amazon Resource Name (ARN)** of the created parameter.

## Aggregating Outputs at the Root Module
To collect all parameter ARNs into a single output, we define:
```hcl
output "all-my-parameters" {
  value = { for k, parameter in module.parameters: k => parameter.arn }
}
```
This creates a **map** where keys are parameter names (`environment`, `version`, `mykey`), and values are their corresponding ARNs.

## Running the Terraform Workflow
1. **Initialize Terraform** to download required providers:
   ```sh
   terraform init
   ```
2. **Apply the configuration** to create resources:
   ```sh
   terraform apply -auto-approve
   ```
3. **Validate Outputs** using Terraform console:
   ```sh
   terraform console
   > module.parameters["environment"].arn
   ```

## Handling Changes to Parameters
### **Why Use for_each Instead of Count?**
- **for_each avoids index shifting issues** that occur with `count`.
- Removing an item **does not affect other parameters**.
- The key (`each.key`) remains stable, ensuring better resource management.

### **Updating Outputs Without Reapplying**
If an output changes, **refreshing the state** updates Terraform's understanding of resources without reapplying:
```sh
terraform refresh
```
This prevents unnecessary resource recreation.

## Conclusion
- **for_each allows dynamic module execution** based on maps.
- **Terraform modules encapsulate reusable logic**, improving maintainability.
- **Aggregated outputs provide a structured way to retrieve created resources.**
- **State management techniques** help avoid unnecessary resource recreation.

This demo showcased the power of Terraform’s **for_each** functionality in module development, enabling efficient infrastructure provisioning.

