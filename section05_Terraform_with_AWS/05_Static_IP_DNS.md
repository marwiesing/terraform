# **Terraform with AWS: Managing Static IPs and DNS with Route53**

## **Resources**
- [AWS Elastic IP (EIP) Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
- [Terraform AWS Provider: Elastic IP](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip)
- [AWS Route 53 Documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)
- [Terraform AWS Provider: Route 53](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_zone)
- [AWS DNS Records](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/ResourceRecordTypes.html)

---

## **Part 1: Understanding Static IPs and Private IPs in AWS**
### **Overview**
AWS provides **two types of IP addresses** for EC2 instances:
1. **Private IPs**:
   - Automatically assigned within a **subnet's CIDR range** (e.g., `10.0.1.0/24`).
   - Used for **internal communication** within a **VPC**.
   - Persist **only while the instance is running**.
   - Can be **manually assigned** for consistency.
2. **Public IPs**:
   - Automatically assigned to instances in **public subnets**.
   - **Dynamic**—changes when an instance stops and restarts.
   - **Not suitable for static assignments**.

✅ **To ensure a consistent Public IP**, use an **Elastic IP (EIP)**.  
✅ **To keep the same Private IP**, manually specify it in Terraform.

---

### **Step 1: Assigning a Static Private IP**
By default, AWS assigns **random private IPs** within a subnet.  
To ensure an instance always gets the **same private IP**, we define it in Terraform:

```hcl
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main-public-1.id
  key_name      = aws_key_pair.mykeypair.key_name

  private_ip = "10.0.1.4"  # Assigns a fixed private IP
}
```
✅ **Ensures the instance always uses `10.0.1.4`**.  
✅ **Works only if another instance does not claim the same IP**.  

---

### **Step 2: Assigning a Static Public IP (Elastic IP)**
Public IPs **change** if the instance stops and starts.  
To **keep the same public IP**, use an **Elastic IP (EIP)**:

```hcl
resource "aws_eip" "example-eip" {
  instance = aws_instance.example.id
  vpc      = true  # Ensures the EIP is for a VPC instance
}
```
✅ **Elastic IP remains the same across instance restarts.**  
✅ **Free if assigned to an instance; AWS charges for unused EIPs.**  

---

### **Step 3: Outputting the Public IP**
After applying Terraform, we can **retrieve the EIP**:

```hcl
output "eip_address" {
  value = aws_eip.example-eip.public_ip
}
```
✅ **Displays the assigned Elastic IP.**  
✅ **Can be referenced in other Terraform resources.**  

---

## **Part 2: Using AWS Route 53 for DNS**
### **Why Use DNS Instead of IPs?**
- **IP addresses can change** (except for EIPs).
- **Easier to remember** domain names (`server1.example.com`).
- **Essential for web hosting**, load balancing, and email.

---

### **Step 1: Creating a Route 53 Hosted Zone**
A **Hosted Zone** in Route 53 is **a container for DNS records**.  
To manage a custom domain, first create a **Route 53 zone**:

```hcl
resource "aws_route53_zone" "example" {
  name = "example.com"
}
```
✅ **Registers `example.com` in AWS Route 53.**  
✅ **Manages all DNS records under `example.com`.**  

---

### **Step 2: Creating a DNS Record for an EC2 Instance**
DNS **A records** map hostnames to **IP addresses**.  
To **resolve `server1.example.com` to an Elastic IP**, add:

```hcl
resource "aws_route53_record" "server1" {
  zone_id = aws_route53_zone.example.zone_id
  name    = "server1.example.com"
  type    = "A"
  ttl     = "300"
  records = [aws_eip.example-eip.public_ip]
}
```
✅ **Maps `server1.example.com` → Elastic IP.**  
✅ **TTL (`300` seconds) determines how long DNS caches the result.**  

---

### **Step 3: Configuring Email DNS with MX Records**
**MX records** define mail servers **for a domain**.  
To use **Google Workspace email**, add:

```hcl
resource "aws_route53_record" "mx" {
  zone_id = aws_route53_zone.example.zone_id
  name    = "example.com"
  type    = "MX"
  ttl     = "300"
  records = [
    "1 aspmx.l.google.com.",
    "5 alt1.aspmx.l.google.com.",
    "5 alt2.aspmx.l.google.com.",
    "10 aspmx2.googlemail.com.",
    "10 aspmx3.googlemail.com.",
  ]
}
```
✅ **Email for `@example.com` routes to Google Mail servers.**  
✅ **Priority (`1`, `5`, `10`) determines which server to try first.**  

---

### **Step 4: Getting Route 53 Name Servers**
To use AWS Route 53 for **your domain**, update your registrar’s name servers:

```hcl
output "ns_servers" {
  value = aws_route53_zone.example.name_servers
}
```
✅ **Displays the AWS name servers for `example.com`.**  
✅ **These must be set in your domain registrar (e.g., GoDaddy, Namecheap).**  

---

## **Part 3: Running Terraform and Verifying DNS**
### **Step 1: Deploying the Configuration**
```bash
terraform init
terraform apply
```
Terraform creates:
- A **Route 53 Hosted Zone** for `example.com`.
- **DNS records** (`A` and `MX`).
- **An Elastic IP** for an EC2 instance.

### **Step 2: Checking the DNS Resolution**
#### **1️⃣ Verify the A Record**
```bash
host server1.example.com
```
Expected output:
```
server1.example.com has address 203.0.113.10
```

#### **2️⃣ Verify the MX Record**
```bash
host -t MX example.com
```
Expected output:
```
example.com mail is handled by 1 aspmx.l.google.com.
example.com mail is handled by 5 alt1.aspmx.l.google.com.
```

✅ **Confirms that Route 53 is correctly resolving DNS.**  

---

## **Part 4: Best Practices**
✅ **Always use DNS hostnames instead of hardcoded IPs.**  
✅ **Assign Elastic IPs to servers that need fixed public IPs.**  
✅ **Use Route 53 for DNS hosting and custom domains.**  
✅ **Set a reasonable TTL to balance caching and updates.**  

---

## **Final Thoughts**
- **Private IPs** can be fixed for internal networking.
- **Elastic IPs** provide **static public IPs**.
- **Route 53** manages **DNS records** for domains.
- **Terraform automates IP and DNS management**.

---

## **Next Steps**
- **Setting up Load Balancers with DNS.**
- **Using Route 53 for Failover & Health Checks.**
- **Configuring Subdomains & CNAME Records.**
