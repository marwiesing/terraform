# Understanding count vs. for_each in Terraform Modules

## Introduction
In the previous lecture, we introduced the ability to use **count** and **for_each** in Terraform modules. While both approaches allow for resource repetition, they have fundamental differences that impact their behavior and usability. This chapter explores these differences and best practices for deciding when to use each.

## Using `count` in Modules
The `count` meta-argument is useful when creating a module conditionally or in a fixed number of repetitions.

### Conditional Module Creation
If you need to create a module **zero or one time**, `count` is a simple and effective choice.
```hcl
module "my_module" {
  source = "./my_module"
  count  = var.enable_my_module ? 1 : 0
}
```
In this case, the module is either created (if `var.enable_my_module` is `true`) or not created at all (if `false`).

### Iterating Over a List
When creating multiple instances using `count`, you reference a list:
```hcl
module "my_module" {
  source = "./my_module"
  count  = length(var.instances)
  name   = var.instances[count.index]
}
```
Each module instance is identified by a **numerical index**, starting from `0`.

### The Problem with `count`
If you later **remove an element in the middle** of the list, Terraform shifts the indices, potentially forcing resource recreation. For example:
```hcl
variable "instances" {
  default = ["instance1", "instance2", "instance3"]
}
```
If `instance2` is removed, the index of `instance3` shifts from `2` to `1`. This **causes Terraform to destroy and recreate `instance3`**, which may lead to unintended downtime.

## Using `for_each` in Modules
The `for_each` meta-argument provides more **stability** when dynamically creating modules, as it allows using **keys instead of indices**.

### Iterating Over a Map
Instead of a list, `for_each` expects a **map**, where each item has a unique key.
```hcl
locals {
  my_map = {
    instance1 = { info = "Info about instance1" }
    instance2 = { info = "Info about instance2" }
    instance3 = { info = "Info about instance3" }
  }
}

module "my_module" {
  source  = "./my_module"
  for_each = local.my_map
  name    = each.key
  details = each.value.info
}
```
### Benefits of `for_each`
1. **No Index Shifting** – The module is identified by a **unique key** rather than a numeric index. Removing an item **does not affect other instances**.
2. **Predictable Changes** – If `instance2` is removed, `instance3` remains unchanged.
3. **Better State Management** – Resources do not need to be recreated unless the key itself is changed.

## Choosing Between `count` and `for_each`
| Feature            | `count`                         | `for_each`                      |
|--------------------|--------------------------------|---------------------------------|
| Best used with    | Lists                          | Maps                           |
| Key type         | Numerical index (0,1,2,…)      | Custom-defined string keys     |
| Resource stability | Index shift can cause recreation | No unintended recreation       |
| Removing elements | May cause instance recreation  | Does not affect other elements |
| Complexity        | Simple but risky               | More robust, slightly more verbose |

## Handling Changes to Keys
Even with `for_each`, **changing a key** results in Terraform treating it as a new instance. To rename a key **without recreating the resource**, use:
```sh
terraform state mv "module.my_module[\"old_key\"]" "module.my_module[\"new_key\"]"
```
This prevents accidental destruction and recreation of resources.

## Conclusion
- Use **`count`** for **conditional creation** (0 or 1 instances) or **static lists**.
- Use **`for_each`** for **dynamic resource creation** when stability is required.
- Avoid relying on **numeric indices** if you expect frequent changes to the resource list.
- Be cautious when modifying keys, as Terraform may recreate resources if keys change.

By understanding these differences, you can make an informed choice that prevents unintended resource recreations and improves Terraform's infrastructure management efficiency.

