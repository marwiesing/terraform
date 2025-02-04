### **Terraform Provisioning: Installing Software on AWS Instances**
#### **Resources**
- 🔗 [Terraform AWS Instance `user_data`](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#user_data)
- 🔗 [AWS EC2 User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- 🔗 [Terraform `templatefile()` Function](https://developer.hashicorp.com/terraform/language/functions/templatefile)
- 🔗 [Cloud-Init Logs for Debugging](https://cloudinit.readthedocs.io/en/latest/topics/logging.html)

---

## **📌 Introduction**
When launching an AWS EC2 instance, we often need to **install software** and **configure the system automatically**. Terraform provides two main ways to do this:
1. ✅ **Using `user_data`** (Preferred for cloud-based automation).
2. 🔄 **Using SSH-based remote execution** (Covered in the next lecture).

In this lecture, we focus on **`user_data`**, which runs a script during the instance’s first boot.

---

## **1️⃣ Why Use `user_data`?**
✅ **Runs automatically on instance launch**  
✅ **No need for SSH access** (Works via AWS API)  
✅ **Ideal for CI/CD pipelines**  
✅ **Supports script-based automation** (e.g., install packages, pull from S3)

⚠ **Important Limitations:**
- Runs **only on first boot** (not on restarts).
- Requires **destroying and recreating the instance** if changes are made.

---

## **2️⃣ Using `user_data` to Install Software**
We will:
✅ Install **Nginx**  
✅ Set up a **web server**  
✅ Retrieve AWS region dynamically

### **🔹 Step 1: Create a Template File**
Create a **`templates/web.tpl`** file:
```sh
#!/bin/bash
apt-get update -y
apt-get install -y nginx

# Start and enable Nginx
systemctl start nginx
systemctl enable nginx

# Print AWS region to a log file
echo "Deployed in region: ${region}" > /var/www/html/index.html
```

### **🔹 Step 2: Modify Terraform Configuration**
Modify **`instance.tf`** to use the template:
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type.example
  subnet_id     = module.vpc.public_subnets[0]
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]
  key_name      = aws_key_pair.sshkey.key_name

  # Use templatefile() to pass dynamic variables
  user_data = templatefile("${path.module}/templates/web.tpl", { region = var.aws_region })

  tags = {
    Name        = "ExampleInstance"
    Environment = "Development"
  }
}
```

### **🔍 Explanation**
- `user_data = templatefile(...)` → Reads the **template** and replaces variables (`${region}`).
- `var.aws_region` → Dynamically passes the AWS region.

---

## **3️⃣ Apply and Verify**
### **🔹 Step 1: Deploy the Changes**
```sh
terraform apply
```
This will:
✅ Launch a **new EC2 instance**  
✅ Execute the **user_data** script  
✅ Install **Nginx**  

### **🔹 Step 2: Check the Logs**
After SSH-ing into the instance, verify that `user_data` ran correctly:
```sh
ssh -i sshkey ubuntu@<public-ip>
```
Then check the **Cloud-Init logs**:
```sh
sudo tail -f /var/log/cloud-init-output.log
```
✅ This shows real-time provisioning logs.

---

## **4️⃣ Testing the Web Server**
To confirm **Nginx** is running, check:
```sh
curl localhost
```
Expected output:
```
Welcome to nginx!
```
If you opened **port 80** in your security group, test from your local machine:
```sh
curl http://<public-ip>
```
✅ You should see `"Deployed in region: us-east-1"`.

---

## **5️⃣ Enhancing `user_data`**
We can extend `user_data` to:
- 🔄 **Download content from S3**
- 🔑 **Retrieve secrets from AWS Systems Manager**
- 🚀 **Configure Docker & Kubernetes**

Example: Sync files from an **S3 bucket**:
```sh
#!/bin/bash
apt-get update -y
apt-get install -y nginx awscli

# Sync website content from S3
aws s3 sync s3://my-static-website /var/www/html/
```
To use this:
```hcl
user_data = templatefile("${path.module}/templates/web.tpl", { bucket_name = "my-static-website" })
```
✅ This automatically **fetches website content from S3**.

---

## **📌 Summary**
| Approach | Benefit | When to Use |
|----------|---------|-------------|
| `user_data` | Runs on first boot, no SSH needed | For automated provisioning |
| `templatefile()` | Passes variables dynamically | For flexible configuration |
| Cloud-Init Logs | Debugging `user_data` issues | When scripts fail |

---

## **✅ Key Takeaways**
✔ **Use `user_data` for cloud-native provisioning.**  
✔ **Use `templatefile()` for reusable scripts.**  
✔ **Check `/var/log/cloud-init-output.log` for debugging.**  
✔ **Use AWS CLI in `user_data` to sync files or install software.**  

---

### **Next Steps**
In the next lecture, we will:
- Use **SSH-based remote execution** for provisioning.
- Compare **user_data vs SSH provisioning**.

Now your EC2 instances **automatically install software** on launch! 🚀🔧