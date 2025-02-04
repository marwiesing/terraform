# **Terraform Commands: A Comprehensive Guide**  

## **Resources**
- 🔗 [Terraform CLI Documentation](https://developer.hashicorp.com/terraform/cli)
- 🔗 [Terraform Commands Overview](https://developer.hashicorp.com/terraform/cli/commands)
- 🔗 [Terraform State Commands](https://developer.hashicorp.com/terraform/cli/commands/state)
- 🔗 [Terraform Import](https://developer.hashicorp.com/terraform/cli/commands/import)
- 🔗 [Terraform Graph Visualization](https://developer.hashicorp.com/terraform/cli/commands/graph)

---

## **Introduction**  
Terraform provides a **rich set of CLI commands** to manage infrastructure efficiently. In this lesson, we’ll cover:  
✅ **Essential commands**: `init`, `validate`, `plan`, `apply`, `destroy`  
✅ **State management commands**: `state`, `refresh`, `show`  
✅ **Advanced commands**: `graph`, `import`, `fmt`, `console`, `force-unlock`  
✅ **Terraform Cloud commands**: `login`, `logout`, `workspace`  

You can view all Terraform commands by running:  

```sh
terraform -h
```

---

## **1️⃣ Essential Terraform Commands**  

### **1.1 `terraform init` – Initialize a Terraform Project**
The **first command** you run in a new Terraform project. It:  
✔️ **Downloads provider plugins** (e.g., AWS, Kubernetes)  
✔️ **Configures backend state storage**  
✔️ **Prepares your working directory**  

**Usage:**  
```sh
terraform init
```

**Example with provider upgrade:**  
```sh
terraform init -upgrade
```
💡 This will update provider versions.

---

### **1.2 `terraform validate` – Validate Configuration**
Checks your **Terraform configuration for syntax errors**.  
```sh
terraform validate
```
✔️ Ensures configuration is **valid** before applying.  

**Example:**  
```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```
If there's a syntax error, `validate` will return an error.

---

### **1.3 `terraform plan` – Preview Changes**
Generates an **execution plan** showing what Terraform will do.  
```sh
terraform plan
```
✔️ Identifies **resources to add, update, or delete**  
✔️ Allows you to review changes **before applying**  

**Save plan for later execution:**  
```sh
terraform plan -out=tfplan
```

---

### **1.4 `terraform apply` – Apply Changes**
Executes the Terraform plan and **deploys infrastructure**.  
```sh
terraform apply
```
✔️ Prompts **confirmation** before execution  

**Apply a saved plan:**  
```sh
terraform apply tfplan
```

---

### **1.5 `terraform destroy` – Delete Resources**
Destroys all managed infrastructure.  
```sh
terraform destroy
```
✔️ **Use cautiously**, as it permanently deletes resources!  

---

## **2️⃣ Terraform Formatting & Debugging**  

### **2.1 `terraform fmt` – Format Configuration**
Automatically formats Terraform files in **standard style**.  
```sh
terraform fmt
```
✔️ Corrects indentation & alignment  

**Before running `fmt` (messy code):**  
```hcl
resource "aws_instance" "web" { ami= "ami-123456" instance_type = "t2.micro"}
```

**After `fmt` (proper formatting):**  
```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

---

### **2.2 `terraform console` – Interactive Debugging**
Opens an interactive console to **test expressions**.  
```sh
terraform console
```
✔️ Useful for debugging & evaluating **Terraform functions**  

Example:  
```sh
> max(10, 20, 30)
30
> join(", ", ["apple", "banana", "cherry"])
"apple, banana, cherry"
```

---

## **3️⃣ Terraform State Management**  

### **3.1 `terraform show` – View State File**
Displays the **current Terraform state** in human-readable format.  
```sh
terraform show
```
✔️ Useful for **checking existing infrastructure**.  

---

### **3.2 `terraform refresh` – Sync State with Reality**
Manually **refreshes Terraform state** to match real infrastructure.  
```sh
terraform refresh
```
✔️ Useful if **manual changes** were made outside Terraform.  

---

### **3.3 `terraform state` – Advanced State Management**
Manage Terraform **state manually**.  
```sh
terraform state list
```
✔️ Lists all resources in state  

**Move a resource:**  
```sh
terraform state mv aws_instance.web aws_instance.new_name
```
✔️ Renames a resource **without recreating it**  

**Remove a resource from state:**  
```sh
terraform state rm aws_instance.web
```
✔️ Terraform **forgets** the resource, but doesn’t delete it in AWS.

---

## **4️⃣ Terraform Locking & Recovery**  

### **4.1 `terraform force-unlock` – Unlock State**
Manually removes a **state lock** if Terraform crashes.  
```sh
terraform force-unlock <LOCK_ID>
```
✔️ Required when state is **locked** due to failed runs.  

---

## **5️⃣ Terraform Graphing & Visualization**  

### **5.1 `terraform graph` – View Dependencies**
Generates a **dependency graph** of Terraform resources.  
```sh
terraform graph | dot -Tpng > graph.png
```
✔️ Visualizes **resource dependencies**  

Example output (using GraphViz):  
```
digraph {
  "module.vpc" -> "aws_instance.web"
  "aws_instance.web" -> "aws_security_group.web"
}
```

---

## **6️⃣ Terraform Import & Workspaces**  

### **6.1 `terraform import` – Import Existing Resources**
Brings existing AWS resources into Terraform **without recreating them**.  
```sh
terraform import aws_instance.web i-1234567890abcdef
```
✔️ Syncs **existing** resources with Terraform  

---

### **6.2 `terraform workspace` – Manage Multiple Environments**
Terraform supports **multiple workspaces** (e.g., `dev`, `prod`).  

**Create a new workspace:**  
```sh
terraform workspace new dev
```
**Switch workspaces:**  
```sh
terraform workspace select prod
```
✔️ Useful for **managing multiple environments**.  

---

## **7️⃣ Terraform Cloud & Automation**  

### **7.1 `terraform login` – Authenticate to Terraform Cloud**
If using **Terraform Cloud**, log in:  
```sh
terraform login
```

**Log out:**  
```sh
terraform logout
```

---

## **8️⃣ Summary of Important Terraform Commands**  

| Command | Description | Example |
|---------|-------------|---------|
| **`terraform init`** | Initialize project & download providers | `terraform init -upgrade` |
| **`terraform validate`** | Check syntax for errors | `terraform validate` |
| **`terraform plan`** | Preview changes before applying | `terraform plan -out=tfplan` |
| **`terraform apply`** | Deploy resources based on plan | `terraform apply tfplan` |
| **`terraform destroy`** | Delete all Terraform-managed resources | `terraform destroy` |
| **`terraform fmt`** | Format `.tf` files in standard style | `terraform fmt` |
| **`terraform console`** | Open interactive Terraform shell | `terraform console` |
| **`terraform show`** | Show current Terraform state | `terraform show` |
| **`terraform refresh`** | Sync Terraform state with reality | `terraform refresh` |
| **`terraform state list`** | List resources in Terraform state | `terraform state list` |
| **`terraform force-unlock`** | Remove a Terraform state lock | `terraform force-unlock <LOCK_ID>` |
| **`terraform graph`** | Generate a dependency graph | `terraform graph | dot -Tpng > graph.png` |
| **`terraform import`** | Import an existing resource into Terraform | `terraform import aws_instance.web i-123456` |
| **`terraform workspace`** | Manage multiple environments | `terraform workspace new dev` |
| **`terraform login`** | Authenticate to Terraform Cloud | `terraform login` |

---

## **9️⃣ Next Steps**  
✅ Practice these commands in a **real Terraform project**.  
✅ Explore **`terraform import`** for migrating existing infrastructure.  
✅ Try **workspaces** for multi-environment setups (`dev`, `prod`).  
✅ Use `terraform fmt` to maintain **clean code formatting**.  

