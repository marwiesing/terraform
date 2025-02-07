# **Terraform with AWS: Using User Data for Instance Initialization**

## **Resources**
- [AWS EC2 User Data Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [Terraform AWS Provider: User Data](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance#user_data)
- [Cloud-Init Documentation](https://cloudinit.readthedocs.io/en/latest/)
- [AWS Cloud-Init Examples](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-cloud-init)
- [Terraform Template File Function](https://developer.hashicorp.com/terraform/language/functions/templatefile)

---

## **Part 1: Introduction to User Data in AWS**
### **Overview**
**User Data** is a feature in AWS that allows **custom scripts** to be executed **when an EC2 instance is launched**. These scripts can:
- Install **software** (e.g., Docker, OpenVPN).
- Configure **system settings**.
- Mount **storage volumes**.
- Register the instance in a **cluster** (e.g., ECS, Kubernetes).
- Apply **security updates** on launch.

✅ **User Data executes only at first boot** (not on reboots).  
✅ **Terraform can pass User Data as a string** or use **Cloud-Init** for advanced configurations.

---

### **Step 1: Using a Simple User Data Script**
Terraform allows simple **inline scripts** in `user_data`:

```hcl
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main-public-1.id
  key_name      = aws_key_pair.mykeypair.key_name

  # Simple User Data Script
  user_data = <<-EOF
    #!/bin/bash
    wget https://swupdate.openvpn.org/as/openvpn-as-2.8.5-Ubuntu20.amd_64.deb
    dpkg -i openvpn-as-2.8.5-Ubuntu20.amd_64.deb
  EOF
}
```
✅ **Installs OpenVPN automatically on launch.**  
✅ **Uses `#!/bin/bash` to specify a shell script.**  
✅ **`wget` downloads the package, `dpkg` installs it.**  

### **Step 2: Using Cloud-Init for More Complex Tasks**
Cloud-Init is **recommended for multi-step configurations**. Instead of raw scripts, we can use **YAML-based Cloud-Init scripts**.

---

## **Part 2: Using Cloud-Init with Terraform**
### **Why Use Cloud-Init?**
- More **structured** and **readable** than raw scripts.
- Supports **multi-part scripts** (e.g., system updates + disk mounting).
- Can use **variables** in Terraform.
- Can be written in **YAML format**.

---

### **Step 1: Defining Cloud-Init in Terraform**
Instead of using a **raw shell script**, we define **`cloudinit.tf`**:

```hcl
data "cloudinit_config" "cloudinit-example" {
  gzip          = false
  base64_encode = false

  # First part: Cloud Config Script
  part {
    filename     = "init.cfg"
    content_type = "text/cloud-config"
    content      = templatefile("scripts/init.cfg", {
      REGION = var.AWS_REGION
    })
  }

  # Second part: Volume Initialization Script
  part {
    content_type = "text/x-shellscript"
    content      = templatefile("scripts/volumes.sh", {
      DEVICE = var.INSTANCE_DEVICE_NAME
    })
  }
}
```
✅ **Combines multiple configurations** into one Cloud-Init script.  
✅ **Uses `templatefile()` to pass variables dynamically.**  

---

### **Step 2: Writing Cloud-Init Configuration (`init.cfg`)**
Cloud-Init **YAML script** for **system updates & Docker installation**:

```yaml
#cloud-config
repo_update: true
repo_upgrade: all
packages:
  - docker.io
  - lvm2
```
✅ **Runs `apt update && apt upgrade` on launch.**  
✅ **Installs `docker.io` (Ubuntu package for Docker).**  
✅ **Installs `lvm2` for Logical Volume Management (used in next steps).**  

---

### **Step 3: Formatting & Mounting an EBS Volume (`volumes.sh`)**
Instead of manually formatting EBS volumes, we automate it using **LVM** (Logical Volume Manager):

```bash
#!/bin/bash
vgchange -ay  # Refresh LVM state

# Check if filesystem exists on the volume
if ! blkid /dev/xvdh; then
  pvcreate /dev/xvdh
  vgcreate data /dev/xvdh
  lvcreate -L 20G -n volume1 data
  mkfs.ext4 /dev/data/volume1
fi

# Ensure the mount directory exists
mkdir -p /data

# Add the volume to /etc/fstab
echo "/dev/data/volume1 /data ext4 defaults,nofail 0 2" >> /etc/fstab

# Mount the volume
mount /data
```
✅ **Automatically formats & mounts the EBS volume.**  
✅ **If volume already exists, it preserves data.**  

---

### **Step 4: Attaching Cloud-Init to an EC2 Instance**
Now, we pass **Cloud-Init** as User Data in Terraform:

```hcl
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main-public-1.id
  key_name      = aws_key_pair.mykeypair.key_name

  # Attach Cloud-Init
  user_data = data.cloudinit_config.cloudinit-example.rendered
}
```
✅ **Terraform renders Cloud-Init on launch.**  
✅ **Combines system updates & disk setup in one step.**  

---

## **Part 3: Running Terraform and Testing the Setup**
### **Step 1: Deploying the Instance**
```bash
terraform init
terraform apply
```
Terraform provisions:
- A `t2.micro` instance.
- A **Cloud-Init script** with system updates and volume setup.
- A **mounted EBS volume** at `/data`.

### **Step 2: Verifying Cloud-Init Execution**
After instance launch, **SSH into the instance**:
```bash
ssh -i mykey ubuntu@<instance-ip>
```

Check if **Docker is installed**:
```bash
docker --version
```
Check if **volume is mounted**:
```bash
df -h
```
Output:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      8.0G  3.2G  4.8G  40% /
/dev/data/volume1 20G  1.0G  19G   5% /data
```
✅ **Docker & LVM volume successfully configured.**  
✅ **Rebooting does not remove the mount.**  

---

## **Part 4: Making Infrastructure Immutable**
### **Problem: What If The Instance is Recreated?**
- `/etc/fstab` is stored on the instance.
- If Terraform **destroys & recreates** the instance, the mount is lost.

### **Solution: Automate Configuration with User Data**
User Data ensures that:
- A **new instance automatically reconfigures itself**.
- Storage volumes are **automatically remounted**.

If the instance is recreated, **Cloud-Init automatically runs again**, preserving all settings.

---

## **Final Thoughts**
✅ **User Data executes commands at first boot.**  
✅ **Terraform supports inline scripts and Cloud-Init.**  
✅ **Cloud-Init enables structured instance initialization.**  
✅ **Combining Cloud-Init with EBS ensures data persistence.**  

---

## **Next Steps**
- **Auto-scaling EC2 Instances** with dynamic User Data.
- **Prebuilding AMIs** to speed up instance launch.
- **Deploying Kubernetes or ECS clusters** using Cloud-Init.

