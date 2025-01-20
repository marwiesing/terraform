### **Terraform Commands Cheat Sheet**

Here’s a comprehensive cheat sheet for **Terraform commands** categorized by functionality:

---

### **1. Basic Workflow Commands**

| Command | Description |
|---------|-------------|
| `terraform init` | Initialize the working directory and download provider plugins. |
| `terraform plan` | Generate and show an execution plan without applying changes. |
| `terraform apply` | Apply the changes required to reach the desired state. |
| `terraform destroy` | Destroy the infrastructure managed by Terraform. |
| `terraform validate` | Validate the configuration files for syntax errors. |
| `terraform fmt` | Format configuration files to a canonical style. |

---

### **2. Plan and Apply**

| Command | Description |
|---------|-------------|
| `terraform plan -out=plan.tfplan` | Save the execution plan to a file. |
| `terraform apply plan.tfplan` | Apply the pre-saved execution plan. |
| `terraform apply` | Shortcut to plan and apply (avoid in production). |
| `terraform refresh` | Update the state file with the latest resource information. |

---

### **3. State Management**

| Command | Description |
|---------|-------------|
| `terraform state list` | List all resources in the state file. |
| `terraform state show <resource>` | Show detailed information about a specific resource. |
| `terraform state mv <source> <destination>` | Move a resource to a different namespace or name in the state. |
| `terraform state rm <resource>` | Remove a resource from the state file without destroying it. |
| `terraform state pull` | View the remote state as JSON. |
| `terraform state push` | Push a local state to the configured backend. |

---

### **4. Outputs**

| Command | Description |
|---------|-------------|
| `terraform output` | Display all outputs from the state file. |
| `terraform output <name>` | Show a specific output value. |
| `terraform output -json` | Display outputs in JSON format for scripting. |

---

### **5. Debugging and Information**

| Command | Description |
|---------|-------------|
| `terraform show` | Display the current state or a saved plan file. |
| `terraform graph` | Generate a dependency graph (useful with tools like Graphviz). |
| `terraform version` | Display the Terraform version. |
| `terraform providers` | Show providers and versions in use. |

---

### **6. File and Directory Management**

| Command | Description |
|---------|-------------|
| `terraform workspace list` | List available workspaces. |
| `terraform workspace new <name>` | Create a new workspace. |
| `terraform workspace select <name>` | Switch to a specific workspace. |
| `terraform import <resource> <id>` | Import an existing resource into the Terraform state. |

---

### **7. Locking and Unlocking**

| Command | Description |
|---------|-------------|
| `terraform force-unlock <lock-id>` | Manually unlock the state file if it’s stuck. |
| `terraform lock` | Enable state locking in the backend. |
| `terraform unlock` | Disable state locking (used cautiously). |

---

### **8. Formatting and Validation**

| Command | Description |
|---------|-------------|
| `terraform validate` | Validate the syntax and logic of your configuration. |
| `terraform fmt` | Auto-format configuration files to follow HCL conventions. |
| `terraform fmt -recursive` | Format all `.tf` files in the current directory and subdirectories. |

---

### **9. Terraform Modules**

| Command | Description |
|---------|-------------|
| `terraform get` | Download and update modules. |
| `terraform init -upgrade` | Upgrade all providers and modules to the latest versions. |
| `terraform module list` | List modules used in the configuration. |

---

### **10. Advanced Debugging**

| Command | Description |
|---------|-------------|
| `TF_LOG=DEBUG terraform apply` | Enable debug logs for detailed troubleshooting. |
| `TF_LOG=TRACE terraform plan` | View trace-level logs for detailed insights. |
| `terraform plan -detailed-exitcode` | Return a specific exit code based on changes (0: no changes, 2: changes present). |

---

### **11. Backends and Remote States**

| Command | Description |
|---------|-------------|
| `terraform init -backend-config="key=value"` | Configure backend settings during initialization. |
| `terraform state pull` | Retrieve and display the remote state. |
| `terraform state push` | Push the local state file to the remote backend. |

---

### **12. Miscellaneous**

| Command | Description |
|---------|-------------|
| `terraform taint <resource>` | Mark a resource for recreation during the next apply. |
| `terraform untaint <resource>` | Remove the taint from a resource. |
| `terraform workspace delete <name>` | Delete a specific workspace. |

---

### **Tips for Effective Terraform Usage**
1. Always use `terraform plan` before `apply` in production.
2. Store your Terraform state securely with backends like AWS S3 or Terraform Cloud.
3. Use version control (e.g., Git) to track configuration changes.
4. Regularly run `terraform validate` and `terraform fmt` to maintain clean and error-free code.

This cheatsheet can be a quick reference for your Terraform work! Let me know if you need further clarification or examples.