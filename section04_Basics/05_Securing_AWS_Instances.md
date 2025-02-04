## **Securing AWS Instances with SSH, Security Groups, and Key Pairs**
#### **Resources**
- 🔗 [Terraform AWS Security Group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)
- 🔗 [Terraform AWS Key Pair](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair)
- 🔗 [AWS EC2 Key Pairs Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
- 🔗 [SSH Key Management](https://www.ssh.com/academy/ssh/key)

---

### **Introduction**
Now that we have successfully launched an AWS instance inside a **VPC with public subnets**, we need to configure **secure access** to it.  
Currently, we **cannot SSH into the instance** because:
1. **No SSH key** is assigned to the instance.
2. **The default security group may block SSH traffic**.

To fix this, we will:
✅ **Create a security group** that allows SSH access.  
✅ **Generate and configure an SSH key pair** for secure login.  
✅ **Attach both to our AWS EC2 instance**.

---

### **Step 1: Creating a Security Group**
A **security group** acts as a virtual firewall that controls inbound and outbound traffic.

#### **1.1 Define a Security Group**
Create a new file **`security-group.tf`**:

```hcl
resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh"
  description = "Allow SSH inbound traffic"
  vpc_id      = module.vpc.vpc_id  # Attach to our VPC

  # Ingress rule: Allow SSH (Port 22) from any IP
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # ⚠️ Allows access from anywhere (use your IP for security)
    ipv6_cidr_blocks = ["::/0"]
  }

  # Egress rule: Allow all outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"  # -1 means all protocols
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "AllowSSH"
  }
}
```

#### **1.2 Explanation**
- The **ingress rule** allows SSH (`port 22`) from **any IP** (`0.0.0.0/0`).
- The **egress rule** allows all outbound traffic.
- It is **attached to our VPC** via `module.vpc.vpc_id`.

#### **1.3 Apply the Security Group**
```sh
terraform apply
```
This will create a **new security group** in our VPC.

---

### **Step 2: Attaching the Security Group to the EC2 Instance**
Now that the security group exists, we need to **attach it to our EC2 instance**.

Modify **`instance.tf`**:
```hcl
resource "aws_instance" "example" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type.example
  subnet_id     = module.vpc.public_subnets[0]  # Assigning a public subnet

  # Attach Security Group
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]

  tags = {
    Name        = "ExampleInstance"
    Environment = "Development"
    OS          = "Ubuntu 20.04"
  }
}
```

#### **2.1 Explanation**
- **`vpc_security_group_ids`** expects a **list of security group IDs**, so we provide `[aws_security_group.allow_ssh.id]`.
- This allows SSH access **only if the security group is applied**.

#### **2.2 Apply the Changes**
```sh
terraform apply
```
Terraform will update the instance **without recreating it**.

---

### **Step 3: Creating an SSH Key Pair**
AWS requires a **key pair** to log in securely to EC2 instances.

#### **3.1 Generating an SSH Key**
On **Linux/macOS**:
```sh
ssh-keygen -t rsa -b 4096 -f mykey
```
This generates:
- **Private key:** `mykey`
- **Public key:** `mykey.pub`

On **Windows (PowerShell)**:
```sh
ssh-keygen.exe -t rsa -b 4096 -f mykey
```
Or use **PuTTYgen** to generate SSH keys.

#### **3.2 Upload the Public Key to AWS**
Modify **`keypair.tf`**:
```hcl
resource "aws_key_pair" "mykey" {
  key_name   = "mykey-demo"
  public_key = file("mykey.pub")  # Read from local file

  tags = {
    Name = "MySSHKey"
  }
}
```

#### **3.3 Apply the Key Pair**
```sh
terraform apply
```
This will upload `mykey.pub` to AWS.

---

### **Step 4: Assigning the Key Pair to the EC2 Instance**
Modify **`instance.tf`** to include the key:
```hcl
resource "aws_instance" "example" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type.example
  subnet_id     = module.vpc.public_subnets[0]
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]

  key_name = aws_key_pair.mykey.key_name  # Attach the key pair

  tags = {
    Name        = "ExampleInstance"
    Environment = "Development"
    OS          = "Ubuntu 20.04"
  }
}
```

#### **4.1 Apply the Changes**
```sh
terraform apply
```
This will **recreate the instance** with the assigned **key pair**.

---

#### **Step 5 - Connecting to the Instance via SSH**
Now that the key pair is attached, we can SSH into the instance.

---

#### **5.1 Get the Public IP**
Run the following command to retrieve the public IP of your instance:
```sh
terraform output public_ip
```
Example output:
```
54.123.45.67
```

---

#### **5.2 SSH into the Instance**
There are **two ways** to connect to your EC2 instance depending on how you store your SSH private key.

##### **Option 1: Using the `-i` Flag (Recommended if Private Key is Not in `~/.ssh/`)**
If your private key **remains in the project directory**, you must specify it explicitly:
```sh
ssh -i sshkey ubuntu@54.123.45.67
```
**Explanation:**
- `-i sshkey` → Specifies the private key (`sshkey` is the private key file, not `.pub`).
- `ubuntu@54.123.45.67` → Uses `ubuntu` as the default username for Ubuntu AMIs.

---

##### **Option 2: Moving the Private Key to the Default SSH Directory**
If you prefer a **simpler SSH command**, move the private key to `~/.ssh/` and rename it to `id_rsa` (or `id_ed25519` if using an Ed25519 key):

```sh
mv sshkey ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa  # Ensure correct permissions
```
Now, you can SSH into the instance without the `-i` flag:
```sh
ssh ubuntu@54.123.45.67
```

---

#### **5.3 SSH Connection for Windows Users**
If you are using **Windows**, you have two options:

##### **Option 1: Using OpenSSH (Windows 10+ or Git Bash)**
If you have OpenSSH installed, use:
```sh
ssh -i sshkey ubuntu@54.123.45.67
```
(Or move the key to `~/.ssh/` and connect without `-i`.)

##### **Option 2: Using PuTTY**
1. Convert `sshkey` to `.ppk` format using **PuTTYgen**.
2. Open **PuTTY** and:
   - Enter **`ubuntu@54.123.45.67`** in **Host Name**.
   - Go to **Connection > SSH > Auth**, and **load the `.ppk` private key**.
   - Click **Open** to connect.

---

✅ **Success! You are now logged into the instance.**  
To exit, type:
```sh
exit
```

---

### **Step 6: Destroying Resources to Avoid Costs**
To clean up:
```sh
terraform destroy
```
This removes:
✅ The EC2 instance  
✅ The security group  
✅ The SSH key pair  
✅ The VPC (if included)  

---

### **Key Takeaways**
✅ **Security groups control inbound/outbound traffic**.  
✅ **Use SSH key pairs to secure logins**.  
✅ **Terraform can read SSH keys from local files**.  
✅ **Use `vpc_security_group_ids` instead of `security_groups` for VPC instances**.  
✅ **Check AWS billing to ensure no unused resources are costing money**.  

---

### **Next Steps**
In the next lecture, we will:
- Install software on the instance automatically.
- Use **Terraform provisioning** to configure EC2 instances.

🚀 **Stay tuned!** 🚀