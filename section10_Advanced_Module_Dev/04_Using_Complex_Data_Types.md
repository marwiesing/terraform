# Terraform Demo: Using Complex Data Types and Flatten Function

## References
- [Terraform For Expressions](https://developer.hashicorp.com/terraform/language/expressions/for)
- [Terraform Flatten Function](https://developer.hashicorp.com/terraform/language/functions/flatten)
- [Terraform Modules](https://developer.hashicorp.com/terraform/language/modules/syntax)

## Introduction
This demo explores using **complex data types** in Terraform modules, particularly how to handle **nested lists of objects** efficiently. We will demonstrate how to iterate over hierarchical structures using **multiple for-loops** and optimize the data structure using the **flatten** function. This approach is useful when dealing with AWS SSM Parameter Store, ensuring that parameters are stored in a hierarchical and organized manner.

## Setting Up the Module
The demo starts with creating a new directory `module-flatten` and copying existing Terraform files from the previous **for_each** demo. Instead of using **for_each** at the module level, we will refactor it to work with **lists of objects**.

### 1. **Defining the Local Variable**
We define a **nested list of maps** in `parameters.tf`:
```hcl
locals {
  my_parameters = [
    {
      "prefix" = "/myprefix"
      "parameters" = [
        { "name"  = "myparameter", "value" = "myvalue" },
        { "name"  = "environment", "value" = "dev" }
      ]
    },
    {
      "prefix" = "/myapp"
      "parameters" = [
        { "name"  = "environment", "value" = "prod" }
      ]
    }
  ]
}
```
Each object contains a `prefix` and a list of `parameters`. This format allows hierarchical structuring of parameters within **AWS SSM Parameter Store**.

### 2. **Passing Parameters to the Module**
Instead of iterating directly in the root module, we pass `local.my_parameters` to a submodule:
```hcl
module "parameters" {
  source     = "./ssm-parameter"
  parameters = local.my_parameters
}
```

## Processing Data Inside the Module
Inside `ssm-parameter`, we need to **flatten** the nested structure and transform it into a format suitable for Terraform resources.

### 1. **Flattening the Data** (`ssm-parameter.tf`)
```hcl
locals {
  parameters = flatten([
    for parameters in var.parameters: [
      for keyvalues in parameters.parameters:
        {
          "name" = "${parameters.prefix}/${keyvalues.name}"
          "value" = keyvalues.value
        }
      ]
    ])
}
```
This **nested for-loop**:
- Iterates over each `prefix`.
- Iterates over the `parameters` list inside each prefix.
- Concatenates the prefix with the parameter name.
- Returns a **flattened list of objects**, making it easier to iterate in Terraform resources.

### 2. **Creating AWS SSM Parameters**
```hcl
resource "aws_ssm_parameter" "parameter" {
  for_each = { for keyvalue in local.parameters: keyvalue.name => keyvalue.value }
  name     = each.key
  type     = "String"
  value    = each.value
}
```
This approach ensures:
- **Unique resource identifiers** using the parameter name.
- **Predictable ordering** without index shifting issues.
- **Efficient iteration** over the flattened structure.

### 3. **Defining Variables** (`variables.tf`)
```hcl
variable "parameters" {
  type = list(object({
    prefix = string
    parameters = list(object({
      name  = string
      value = string
    }))
  }))
  default = []
}
```
This explicitly defines the expected input structure, ensuring validation at runtime.

## Running the Terraform Workflow
1. **Initialize Terraform** to download dependencies:
   ```sh
   terraform init
   ```
2. **Run Terraform Console to Validate Transformations**
   ```sh
   terraform console
   > local.my_parameters
   ```
3. **Apply the Configuration**:
   ```sh
   terraform apply -auto-approve
   ```
4. **Verify the Created Resources**:
   - Check AWS SSM Parameter Store to confirm hierarchical storage.

## Handling Common Errors
### **Mismatch in Data Types**
Terraform may return errors if the variable definitions **do not match** the actual data structure.
- Ensure that the `parameters` variable is correctly defined as **a list of objects containing another list of objects**.
- Use Terraform console to inspect data types before applying changes.

### **Flattening Nested Lists Correctly**
If a nested list structure is returned instead of a single list, ensure **flatten** is used properly.
- `flatten([[...],[...]])` merges nested lists into a single list.
- Confirm the structure in Terraform console before using it in `for_each`.

## Conclusion
- **Flatten function optimizes nested data structures**, making them manageable.
- **Using multiple for-loops helps iterate over deeply nested objects**.
- **Hierarchical SSM parameter storage is structured and easy to manage**.
- **Terraform console is a powerful tool for debugging complex loops before applying them**.

This demo provides a structured approach to handling **hierarchical data in Terraform**, making module interactions more powerful and scalable.

