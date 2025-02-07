Here is the **enhanced and detailed version** of the transcript for **EBS Volumes**, incorporating additional **explanations, examples, and best practices**.  

---

# **Terraform with AWS: Managing EBS Volumes**
  
## **Resources**
- [AWS EBS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html)
- [Terraform AWS Provider: EBS Volume](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ebs_volume)
- [AWS Block Storage vs. Ephemeral Storage](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html)
- [AWS EBS Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
- [Terraform AWS Volume Attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/volume_attachment)

---

## **Part 1: Understanding AWS EBS (Elastic Block Storage)**
### **Overview**
AWS provides **two types of storage** for EC2 instances:
1. **EBS (Elastic Block Storage)**:  
   - Persistent storage that remains **even if the instance is terminated**.
   - Can be attached to one instance at a time.
   - Supports **snapshots** for backup and restoration.
2. **Ephemeral Storage (Instance Store):**  
   - **Local storage** that exists **only while the instance is running**.
   - Data is lost if the instance is stopped or terminated.

✅ **t2.micro instances use EBS storage**, not ephemeral storage.

### **EBS Root Volume and Additional Volumes**
- **Root Volume (`/dev/xvda`)**:
  - The primary disk (default: **8GB**).
  - Can be configured for **automatic deletion** on instance termination.
- **Additional Volumes**:
  - Used for **logs, databases, or application data**.
  - Remain even if the instance is terminated.

---

### **Step 1: Creating an EBS Volume**
EBS volumes are defined using the **`aws_ebs_volume`** resource.

```hcl
resource "aws_ebs_volume" "ebs-volume-1" {
  availability_zone = "eu-west-1a"
  size              = 20
  type              = "gp2"

  tags = {
    Name = "extra-volume-data"
  }
}
```
✅ **Availability Zone**: Must match the instance's AZ.  
✅ **Size**: `20GB` (Can be resized later).  
✅ **Type**:  
   - `gp2`: General-purpose **SSD** (common choice).  
   - `io1`: High-performance SSD for databases.  
   - `st1`: Throughput-optimized for logs.  
   - `sc1`: Cold storage, infrequently accessed data.

---

### **Step 2: Attaching the EBS Volume to an Instance**
To attach an **existing volume** to an **EC2 instance**, we use **`aws_volume_attachment`**:

```hcl
resource "aws_volume_attachment" "ebs-volume-1-attachment" {
  device_name                    = "/dev/xvdh"
  volume_id                      = aws_ebs_volume.ebs-volume-1.id
  instance_id                    = aws_instance.example.id
  stop_instance_before_detaching = true
}
```
✅ **Device Name (`/dev/xvdh`)**: Used for mounting inside the OS.  
✅ **Volume ID**: Links to the previously created volume.  
✅ **Instance ID**: Specifies the instance to attach the volume to.  
✅ **`stop_instance_before_detaching = true`**: Ensures safe removal before detaching.

---

### **Step 3: Modifying the Root Volume (Optional)**
By default, **root volumes** are **8GB** and deleted when the instance terminates.  
To **increase storage size** and **disable deletion**, use **`root_block_device`**:

```hcl
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main-public-1.id
  key_name      = aws_key_pair.mykeypair.key_name

  root_block_device {
    volume_size          = 16
    volume_type          = "gp2"
    delete_on_termination = false
  }
}
```
✅ **`volume_size = 16`**: Increases the root volume to **16GB**.  
✅ **`delete_on_termination = false`**: Keeps the root volume after termination.

---

## **Part 2: Attaching and Using an EBS Volume in a Running EC2 Instance**
### **Demo Setup (`demo-9`)**
Terraform files used:
```bash
$ ls terraform-course/demo-9/*.tf
instance.tf  securitygroup.tf  vpc.tf  key.tf  vars.tf  provider.tf  versions.tf
```

### **Step 1: Running Terraform to Create the Resources**
```bash
terraform init
terraform apply
```
✅ **Terraform will create:**
- A `t2.micro` instance in a public subnet.
- A **20GB EBS volume**.
- A **security group allowing SSH**.
- A **key pair for authentication**.

### **Step 2: Logging into the Instance**
Retrieve the **public IP**:
```bash
terraform output instance_ip
```
Or check the Terraform state file:
```bash
cat terraform.tfstate | grep "public_ip"
```
Now, connect via **SSH**:
```bash
ssh -i mykey ubuntu@<instance-ip>
```

### **Step 3: Verifying the Attached Volume**
Once inside the instance, check available storage:
```bash
lsblk
```
Example output:
```
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk /
xvdh    202:16   0  20G  0 disk
```
✅ **`xvda` (8GB)**: Root volume (default).  
✅ **`xvdh` (20GB)**: Newly attached **EBS volume**.

---

### **Step 4: Formatting and Mounting the Volume**
New EBS volumes **must be formatted** before use.

1️⃣ **Format the volume (`/dev/xvdh`) with EXT4:**
```bash
sudo mkfs.ext4 /dev/xvdh
```
2️⃣ **Create a mount directory:**
```bash
sudo mkdir /data
```
3️⃣ **Mount the volume to `/data`:**
```bash
sudo mount /dev/xvdh /data
```
4️⃣ **Verify the mounted volume:**
```bash
df -h
```
Example output:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      8.0G  3.2G  4.8G  40% /
/dev/xvdh       20G   1.0G  19G   5% /data
```
✅ **Now, the `/data` directory uses the new EBS volume.**

---

### **Step 5: Persisting the Mount (FSTAB)**
By default, the mount **disappears on reboot**.  
To make it **persistent**, add it to `/etc/fstab`:

1️⃣ Get the UUID of the volume:
```bash
sudo blkid /dev/xvdh
```
Example output:
```
/dev/xvdh: UUID="abcd-1234" TYPE="ext4"
```
2️⃣ Add this line to `/etc/fstab`:
```bash
echo 'UUID=abcd-1234 /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab
```
✅ **Reboot and verify:**
```bash
sudo reboot
ssh -i mykey ubuntu@<instance-ip>
df -h
```
The `/data` mount should **persist after reboot**.

---

### **Step 6: Handling Infrastructure Changes with User Data**
If the instance is **recreated with Terraform**, the `/etc/fstab` entry is **lost**.  
A **better approach** is using **user data scripts**:

```hcl
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main-public-1.id
  key_name      = aws_key_pair.mykeypair.key_name

  user_data = <<-EOF
    #!/bin/bash
    mkfs.ext4 /dev/xvdh
    mkdir -p /data
    mount /dev/xvdh /data
    echo "/dev/xvdh /data ext4 defaults,nofail 0 2" >> /etc/fstab
  EOF
}
```
✅ **Ensures `/data` is always mounted on new instances.**  

---

## **Final Thoughts**
✅ **EBS volumes provide persistent storage for EC2 instances.**  
✅ **Terraform automates volume creation and attachment.**  
✅ **FSTAB ensures the volume remains after reboot.**  
✅ **User data scripts prevent manual setup on new instances.**  

---

## **Next Steps**
- **Automating EBS Snapshots** for backups.  
- **Expanding Storage** on existing instances.  
- **Performance Optimization** for high-throughput applications.  
