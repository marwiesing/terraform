# **Terraform Cloud: Features, Workspaces, and Policy Enforcement with Sentinel**  

## **Official Resources**  
To gain a deeper understanding of Terraform Cloud, check out these official HashiCorp resources:  
- 📌 **[Terraform Cloud Overview](https://developer.hashicorp.com/terraform/cloud)** – Learn how Terraform Cloud works.  
- 📌 **[Terraform Cloud Workspaces](https://developer.hashicorp.com/terraform/cloud-docs/workspaces)** – Learn how Terraform Cloud workspaces differ from local ones.  
- 📌 **[Terraform Sentinel Documentation](https://developer.hashicorp.com/sentinel)** – Policy-as-code framework for Terraform Cloud.  
- 📌 **[Terraform Cloud Pricing](https://www.hashicorp.com/products/terraform/pricing)** – Learn about free and paid features of Terraform Cloud.  

---

## **1. What is Terraform Cloud?**  

Terraform Cloud is a **HashiCorp-managed service** that allows teams to use Terraform in a **centralized, consistent, and scalable** way.  

✅ **Instead of running Terraform locally or on Jenkins, Terraform Cloud executes Terraform remotely.**  
✅ **Provides a shared state backend, secret management, policy enforcement, and CI/CD integrations.**  
✅ **Best suited for teams and enterprises that need automation, governance, and collaboration.**  

---

## **2. Key Features of Terraform Cloud**  

| **Feature** | **Description** | **Best for** |
|------------|----------------|--------------|
| **Remote Execution** | Runs Terraform remotely in a **reliable and consistent** environment. | Teams that need centralized Terraform management. |
| **State Management** | Stores and **locks Terraform state** to prevent corruption from concurrent updates. | Multi-user Terraform workflows. |
| **Version Control Integration** | Connects with **GitHub, GitLab, Bitbucket** to trigger Terraform runs from code changes. | Infrastructure as Code (IaC) workflows. |
| **Secret Management** | Stores environment variables **securely** (e.g., API keys, passwords). | Keeping credentials out of code. |
| **Private Module Registry** | Allows teams to **share custom Terraform modules** internally. | Organizations with reusable Terraform modules. |
| **Access Controls** | Restricts who can apply changes and enforces **role-based permissions**. | Enterprises needing governance. |
| **Policy Enforcement (Sentinel)** | Implements **policy-as-code** to enforce security & compliance. | Enterprises needing strict compliance. |

---

## **3. Terraform Cloud Workspaces vs. Local Workspaces**  

Terraform Cloud **workspaces** are **different** from local Terraform workspaces.  

| **Feature** | **Local Workspaces** | **Terraform Cloud Workspaces** |
|------------|----------------------|-----------------------------|
| **State Storage** | Stores **state files locally**. | Stores **state securely in Terraform Cloud**. |
| **Variable Management** | Uses **`.tfvars` or CLI arguments**. | Manages **workspace-specific environment variables**. |
| **Isolation** | Same credentials, shared environment. | Each workspace has **separate credentials and settings**. |
| **Execution** | Runs on local machines. | Runs Terraform remotely on **Terraform Cloud servers**. |

✅ **Best Practice:** Use **Terraform Cloud Workspaces** to **separate environments** like `dev`, `staging`, and `prod`.  

---

## **4. Setting Up Terraform Cloud**  

### **Step 1: Create a Terraform Cloud Account**  
- Sign up at [app.terraform.io](https://app.terraform.io).  
- Create a **new organization** (e.g., `my-org`).  
- Create a **workspace** for your project.  

### **Step 2: Connect Terraform Cloud to GitHub/GitLab**  
- **Enable Version Control Integration** to trigger Terraform runs from Git commits.  

### **Step 3: Set Terraform Backend to Terraform Cloud**  
Modify `backend.tf` to use Terraform Cloud as the state backend:  

```hcl
terraform {
  backend "remote" {
    hostname     = "app.terraform.io"
    organization = "my-org"

    workspaces {
      name = "dev-workspace"
    }
  }
}
```
✅ **This ensures Terraform state is stored and managed in Terraform Cloud.**  

---

## **5. Terraform Cloud Sentinel: Policy-as-Code for Compliance**  

🔹 **Sentinel** is an **enterprise feature** that enforces **security, compliance, and cost policies** before Terraform applies changes.  

🔹 **Example Use Cases for Sentinel:**  
✔ **Restrict AMI Owners** – Only allow trusted AWS AMI owners (Amazon, Ubuntu).  
✔ **Mandatory Tags** – Enforce that every Terraform resource must have a tag.  
✔ **Security Group Restrictions** – Block `0.0.0.0/0` in security group rules.  
✔ **Limit EC2 Instance Types** – Only allow cost-effective EC2 instances.  
✔ **Require S3 Encryption** – Ensure all S3 buckets are encrypted with **KMS**.  
✔ **Cost Limits** – Block Terraform plans if estimated monthly cost exceeds a threshold.  

---

## **6. Example Sentinel Policy: Restrict AMI Owners**  

The following **Sentinel policy** **prevents Terraform from launching unapproved AMIs**.  

### **1️⃣ Define Allowed AMI Owners**  
```hcl
import "tfplan"

allowed_owners = ["amazon", "ubuntu"]

main = rule {
  all tfplan.resources.aws_instance as _, instances {
    all instances as instance {
      instance.applied.ami_owner in allowed_owners
    }
  }
}
```
✅ **Terraform will only allow AMIs from Amazon and Ubuntu.**  
❌ **If a different AMI is used, Terraform will reject the apply.**  

---

## **7. Example Sentinel Policy: Enforce S3 Bucket Encryption**  

This policy **requires that all S3 buckets are encrypted** using KMS.  

```hcl
import "tfplan"

main = rule {
  all tfplan.resources.aws_s3_bucket as _, buckets {
    all buckets as bucket {
      bucket.applied.server_side_encryption_configuration != null
    }
  }
}
```
✅ **Ensures all new S3 buckets have encryption enabled.**  

🔹 **More Sentinel Policies:** See [Sentinel Documentation](https://developer.hashicorp.com/sentinel).  

---

## **8. Terraform Cloud vs. Terraform Enterprise**  

| **Feature** | **Terraform Cloud** | **Terraform Enterprise** |
|------------|----------------------|--------------------------|
| **Hosting** | Hosted by HashiCorp. | Self-hosted in your environment. |
| **Pricing** | Free for small teams, paid for enterprises. | Enterprise licensing required. |
| **Use Case** | Small to medium teams needing Terraform automation. | Large enterprises needing **on-premises** Terraform management. |

✅ **Terraform Enterprise is best for highly regulated industries needing complete control over Terraform infrastructure.**  

---

## **9. Best Practices for Using Terraform Cloud**  

✅ **Use workspaces to separate environments** – e.g., `dev`, `staging`, `prod`.  
✅ **Store sensitive credentials securely in Terraform Cloud** – Keep them out of `.tf` files.  
✅ **Enable Sentinel policies for security & compliance** – Prevent misconfigurations.  
✅ **Use private module registry to share Terraform modules** – Keep infrastructure modular.  
✅ **Integrate Terraform Cloud with GitHub/GitLab for CI/CD** – Automate Terraform execution.  

---

## **10. Summary of Key Terraform Cloud Commands**  

| **Command** | **Description** |
|------------|----------------|
| `terraform login` | Authenticates with Terraform Cloud. |
| `terraform init` | Configures Terraform Cloud as the backend. |
| `terraform workspace list` | Lists available Terraform Cloud workspaces. |
| `terraform apply` | Runs Terraform remotely in Terraform Cloud. |
| `sentinel apply` | Validates a Sentinel policy before applying. |

For further learning, check out:  
- **[Terraform Cloud Overview](https://developer.hashicorp.com/terraform/cloud)**  
- **[Terraform Cloud Workspaces](https://developer.hashicorp.com/terraform/cloud-docs/workspaces)**  
- **[Terraform Sentinel Documentation](https://developer.hashicorp.com/sentinel)**  

---

## **11. Key Takeaways**  

✅ **Terraform Cloud centralizes Terraform execution and state management.**  
✅ **Workspaces in Terraform Cloud are fully isolated (unlike local workspaces).**  
✅ **Sentinel enforces security, compliance, and cost controls in Terraform Cloud.**  
✅ **Terraform Cloud integrates with version control for automatic Terraform runs.**  
✅ **For enterprises needing full control, Terraform Enterprise is available as a self-hosted solution.**  