
## **Setting Up Remote State Storage (AWS S3 + DynamoDB Locking)**

### **Resources**
- 🔗 [Remote State Documentation](https://developer.hashicorp.com/terraform/language/state/remote)  
- 🔗 [Terraform Backend Documentation](https://developer.hashicorp.com/terraform/language/settings/backends)  
- 🔗 [Terraform S3 Backend Configuration](https://developer.hashicorp.com/terraform/language/settings/backends/s3)  
- 🔗 [Terraform State Management Commands](https://developer.hashicorp.com/terraform/cli/state)  
- 🔗 [Terraform Locking with DynamoDB](https://developer.hashicorp.com/terraform/language/settings/backends/s3#dynamodb-table-for-state-locking)  
- 🔗 [AWS S3 Documentation](https://docs.aws.amazon.com/s3/index.html)  
- 🔗 [AWS DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/index.html)  
- 🔗 [Terraform Force Unlock Command](https://developer.hashicorp.com/terraform/cli/commands/force-unlock)  
- 🔗 [Terraform State Storage Best Practices](https://developer.hashicorp.com/terraform/tutorials/state/state-storage)  

---

These links will help clarify **backend setup**, **remote state locking**, and **troubleshooting state conflicts**. Let me know if you need more! 🚀

### **Step 1: Create an S3 Bucket**
Terraform state will be stored here.
```sh
aws s3api create-bucket --bucket terraform-remote-state-12345678 --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1
```
- Change `terraform-remote-state-12345678` to a unique name.
- Replace `eu-central-1` with your AWS region.

Enable **Versioning** to track state changes:
```sh
aws s3api put-bucket-versioning --bucket terraform-remote-state-12345678 --versioning-configuration Status=Enabled
```

---

### **Step 2: Create a DynamoDB Table for State Locking**
This prevents multiple people from applying Terraform at the same time.
```sh
aws dynamodb create-table --table-name terraform-locking \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```
✅ **Ensures only one Terraform operation runs at a time.**

---

### **Step 3: Update Terraform Configuration**
Modify your `main.tf` or `terraform.tf`:

```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-remote-state-12345678"  # Change to your bucket name
    key            = "first-steps/terraform.tfstate"
    region         = "eu-central-1"  # Change to your AWS region
    encrypt        = true
    dynamodb_table = "terraform-locking"  # Enables state locking
  }
}
```
✅ **This moves your state from local to remote.**

---

### **Step 4: Initialize Terraform Backend**
After adding the backend, run:
```sh
terraform init
```
- Terraform detects the **backend change** and asks if you want to **migrate your existing state**.
- Type **yes** to proceed.

---

### **Step 5: Verify Remote State**
Check if Terraform is using remote state:
```sh
terraform state pull
```
✅ If this returns a **JSON state file**, your setup is correct.

---

### **Step 6: Test State Locking**
Run `terraform apply` in **two terminals** at the same time.
```sh
terraform apply
```
One will **lock** the state while the other gets:
```
Error acquiring state lock: ConditionalCheckFailedException
```
✅ **This means locking is working correctly.**

---

### **Step 7: Manually Unlock State (if needed)**
If Terraform crashes, you can **force-unlock**:
```sh
terraform force-unlock <LockID>
```
Find the **LockID** from:
```sh
aws dynamodb scan --table-name terraform-locking
```
🚨 **Only use this if you're sure no one else is running Terraform!**

---

### **✅ Final Notes**
- **Remote state** is essential when working in a team.
- **DynamoDB locking** prevents Terraform conflicts.
- **S3 versioning** allows rollback to previous states.
- Always use `terraform init` after **modifying backends**.

---
