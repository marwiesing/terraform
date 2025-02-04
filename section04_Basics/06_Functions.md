### **Terraform Functions: Reading Files and Using Templates**
#### **Resources**
- 🔗 [Terraform Functions Documentation](https://developer.hashicorp.com/terraform/language/functions)
- 🔗 [Terraform File System Functions](https://developer.hashicorp.com/terraform/language/functions/file)
- 🔗 [Terraform `templatefile` Function](https://developer.hashicorp.com/terraform/language/functions/templatefile)
- 🔗 [Terraform Console](https://developer.hashicorp.com/terraform/cli/commands/console)

---

## **📌 Introduction**
In this lecture, we explore **Terraform functions**, focusing on:
✅ **Reading a file dynamically** instead of hardcoding values.  
✅ **Using `file()`** to read a public key automatically.  
✅ **Exploring `templatefile()`** for managing dynamic content.  

Functions in Terraform allow us to **simplify configurations**, **avoid repetition**, and **read external data dynamically**.

---

## **1️⃣ Using Terraform's Built-in Functions**
Terraform provides various function categories:
- **Numeric Functions** → `max()`, `min()`, `abs()`, etc.
- **String Functions** → `join()`, `split()`, `replace()`, etc.
- **Filesystem Functions** → `file()`, `templatefile()`
- **Collection Functions** → `tolist()`, `tomap()`, etc.

---

## **2️⃣ Reading a File with `file()`**
Instead of **copy-pasting** an SSH public key, we can use `file()` to read it dynamically.

### **🔹 Old Approach (Hardcoded Public Key)**
```hcl
resource "aws_key_pair" "sshkey" {
  key_name   = "sshkeydemo-key"
  public_key = "ssh-rsa AAAA...../TQDwDQ== mwiesing@workstation"
}
```

### **✅ Improved Approach (Using `file()`)**
```hcl
resource "aws_key_pair" "sshkey" {
  key_name   = "sshkeydemo-key"
  public_key = file("${path.module}/sshkey.pub")  # Reads key from a file in the module directory

  tags = {
    Name = "SSHKeyDemo"
  }
}
```  
**Why Use `${path.module}`?**  
- Ensures Terraform **always finds** `sshkey.pub`, even if executed from a different directory.  
- Improves **portability**, especially when using Terraform modules.  
- Prevents issues where Terraform might look for `sshkey.pub` in the wrong directory.  

 **Best Practice:** Always use `${path.module}` when referencing local files inside Terraform modules!


### **🔍 Explanation**
- `file("sshkey.pub")` reads the **contents** of `sshkey.pub` and uses it as the **public key**.
- This ensures **automation** and eliminates **manual copy-pasting**.

### **📌 Apply the Change**
```sh
terraform apply
```

---

## **3️⃣ Using `templatefile()` for Dynamic Templates**
The `templatefile()` function reads a **template file** and replaces variables dynamically.

### **🔹 Example Use Case: Configuring a Script**
Imagine we need a **user data script** to initialize an EC2 instance.

**`userdata.tpl` (Template File)**
```hcl
#!/bin/bash
echo "Hello, ${username}" > /home/ubuntu/welcome.txt
apt update -y
```
Here, `${username}` is a **placeholder variable**.

### **✅ Terraform Code Using `templatefile()`**
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type.example
  subnet_id     = module.vpc.public_subnets[0]
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]
  key_name      = aws_key_pair.sshkey.key_name

  user_data = templatefile("userdata.tpl", { username = "mwiesing" })

  tags = {
    Name = "ExampleInstance"
    Environment = "Development"
  }
}
```

### **🔍 Explanation**
- `templatefile("userdata.tpl", { username = "mwiesing" })`  
  - Reads the **`userdata.tpl`** file.
  - Replaces `${username}` with `"mwiesing"`.
- The **EC2 instance will execute** this script at launch.

### **📌 Apply the Change**
```sh
terraform apply
```
Now, when the instance starts, `welcome.txt` will contain:
```
Hello, mwiesing
```

---

## **4️⃣ Using Terraform Console to Test Functions**
Terraform provides an interactive **console** to test functions.

### **📌 Open Terraform Console**
```sh
terraform console
```

### **📌 Test `file()` Function**
```hcl
file("sshkey.pub")
```
Output:
```
ssh-rsa AAAAB3Nza... mwiesing@workstation
```

### **📌 Test `join()` Function**
```hcl
join(", ", ["AWS", "Terraform", "DevOps"])
```
Output:
```
"AWS, Terraform, DevOps"
```

### **📌 Test `templatefile()`**
```hcl
templatefile("userdata.tpl", { username = "Edward" })
```
Output:
```
#!/bin/bash
echo "Hello, Edward" > /home/ubuntu/welcome.txt
apt update -y
```

### **📌 Exit Console**
```sh
exit
```

---

## **📌 Summary**
| Function | Purpose | Example |
|----------|---------|---------|
| `file()` | Reads a file's content | `file("sshkey.pub")` |
| `templatefile()` | Reads a template file and replaces variables | `templatefile("userdata.tpl", { username = "mwiesing" })` |
| `join()` | Combines a list into a string | `join(", ", ["AWS", "Terraform"])` |

---

## **✅ Key Takeaways**
✔ **Use `file()`** to dynamically read external files (like SSH keys).  
✔ **Use `templatefile()`** for **configuration templates** and scripts.  
✔ **Terraform console** allows testing functions interactively.  

---

### **Next Steps**
In the upcoming lectures, we will:
- Use **provisioning scripts** to automate software installation.
- Work with **remote state storage** to enhance collaboration.
