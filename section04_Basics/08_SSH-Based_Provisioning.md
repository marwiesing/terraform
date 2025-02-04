### **Terraform Remote Provisioners: Legacy SSH-Based Provisioning**
#### **Resources**
- 🔗 [Terraform `provisioner` Documentation](https://developer.hashicorp.com/terraform/language/resources/provisioners)
- 🔗 [Terraform `remote-exec` Provisioner](https://developer.hashicorp.com/terraform/language/resources/provisioners/remote-exec)
- 🔗 [AWS User Data vs. Provisioners](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#user_data)

---

## **📌 Introduction**
In this lesson, we explore **legacy Terraform provisioners** that use SSH for remote execution. These provisioners:
- Allow **executing commands** after instance creation.
- Require **SSH access** from the Terraform machine.
- Are considered **a last resort** (⚠ **not recommended**).

### **✅ When Should You Use This?**
- **Only if `user_data` is NOT an option** (e.g., legacy systems).
- When **deploying software that must run after first boot**.
- When Terraform must **wait** for certain conditions before provisioning.

### **❌ Why Is It Not Recommended?**
- **Requires SSH access** → Terraform needs direct access to the instance.
- **State-dependent** → Makes infrastructure harder to manage in CI/CD.
- **Not idempotent** → Errors can lead to incomplete provisioning.
- **Cloud-native alternatives exist** → AWS **User Data**, SSM, Ansible, etc.

---

## **1️⃣ Setting Up `remote-exec` Provisioner**
We will:
✅ **Connect to the EC2 instance via SSH.**  
✅ **Run `apt-get update` and install Nginx.**  
✅ **Ensure Terraform waits for SSH connection before provisioning.**

### **🔹 Step 1: Modify `instance.tf`**
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type.example
  subnet_id     = module.vpc.public_subnets[0]
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]
  key_name      = aws_key_pair.sshkey.key_name

  provisioner "remote-exec" {
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("${path.module}/sshkey")
      host        = self.public_ip
    }

    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install -y nginx"
    ]
  }

  tags = {
    Name        = "ExampleInstance"
    Environment = "Development"
  }
}
```

---

### **🔍 Explanation**
- `provisioner "remote-exec"` → Runs **remote** commands over SSH.
- **Connection Block:**
  - `type = "ssh"` → Specifies SSH-based execution.
  - `user = "ubuntu"` → Uses the default Ubuntu user.
  - `private_key = file("${path.module}/sshkey")` → Reads the **private SSH key**.
  - `host = self.public_ip` → Uses the instance’s **public IP**.

- **Inline Commands:**
  - `"sudo apt-get update -y"` → Updates package list.
  - `"sudo apt-get install -y nginx"` → Installs **Nginx**.

---

## **2️⃣ Handling SSH Timing Issues**
Sometimes, Terraform tries to SSH before the instance is fully booted. To fix this, we can:
1. **Increase `provisioner "remote-exec"` timeout.**
2. **Add a sleep command to wait for SSH readiness.**

### **🔹 Option 1: Add a Sleep Before Installation**
Modify the `inline` block:
```hcl
inline = [
  "sleep 30",  # Wait 30 seconds before proceeding
  "sudo apt-get update -y",
  "sudo apt-get install -y nginx"
]
```
✅ This prevents Terraform from failing if SSH isn’t ready.

---

## **3️⃣ Forcing Instance Recreation on `user_data` Change**
Normally, modifying `user_data` **does not recreate** the instance. To force recreation:
```hcl
lifecycle {
  replace_on_change = ["user_data"]
}
```
✅ Now, when `user_data` changes, Terraform will **destroy and recreate** the instance.

---

## **4️⃣ Applying Changes**
### **🔹 Step 1: Apply Terraform**
```sh
terraform apply
```
Terraform will:
1. **Create the EC2 instance.**
2. **Wait for SSH connectivity.**
3. **Execute remote commands over SSH.**

### **🔹 Step 2: Verify Installation**
After deployment, SSH into the instance:
```sh
ssh -i sshkey ubuntu@<public-ip>
```
Run:
```sh
curl localhost
```
Expected output:
```
Welcome to nginx!
```
✅ If you opened **port 80**, access it from your browser:
```
http://<public-ip>
```

---

## **📌 When to Use `remote-exec` vs. `user_data`**
| Feature | `user_data` (Recommended) | `remote-exec` (Last Resort) |
|---------|------------------|------------------|
| **Runs on First Boot?** | ✅ Yes | ❌ No, runs after instance boots |
| **Works Without SSH?** | ✅ Yes | ❌ No, requires SSH access |
| **Can Be Used in CI/CD?** | ✅ Yes | ❌ No, requires a local SSH key |
| **Idempotent?** | ✅ Yes | ❌ No, may fail on re-runs |
| **Best for Cloud-Native?** | ✅ Yes | ❌ No |

---

## **✅ Key Takeaways**
✔ **Use `user_data` whenever possible**—it runs without SSH and works better in cloud environments.  
✔ **Use `remote-exec` only as a last resort**, e.g., for legacy systems.  
✔ **Ensure SSH is available** by allowing **port 22** in security groups.  
✔ **Use `replace_on_change` to force instance recreation** when needed.  

---

### **Next Steps**
In the next lessons, we will:
- Explore **better alternatives** like AWS SSM for remote execution.
- Move towards **containerized deployments** with Docker & Kubernetes.

