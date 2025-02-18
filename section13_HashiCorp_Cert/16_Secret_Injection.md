# **Terraform Secret Injection & Secure Credential Management**  

## **Official Resources**  
To gain a deeper understanding of secure authentication and secret management in Terraform, check out these official HashiCorp and AWS resources:  
- 📌 **[Terraform AWS Authentication Methods](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication)** – How Terraform authenticates with AWS.  
- 📌 **[AWS IAM Roles and Federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)** – Role-based authentication for AWS.  
- 📌 **[HashiCorp Vault Secrets Management](https://developer.hashicorp.com/vault/docs/secrets/aws)** – How Vault issues short-lived AWS credentials.  
- 📌 **[AWS Instance Metadata Service (IMDS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)** – Fetching temporary credentials in AWS EC2.  

---

## **1. Why Avoid Long-Lived Credentials?**  

🔹 **Long-lived credentials (AWS Access Keys, Secret Keys) should be avoided** because:  
- They **never expire** and are **a security risk** if exposed.  
- Attackers can use compromised credentials to access cloud resources indefinitely.  
- Managing and rotating them manually is cumbersome.  

✅ **Use short-lived credentials** that expire after a defined period (e.g., 1 hour, 4 hours).  

---

## **2. Secure Alternatives to Long-Lived Credentials**  

Terraform supports multiple ways to **avoid hardcoded long-lived credentials**:

| **Method** | **Description** | **Best for** |
|-----------|--------------|----------------|
| **AWS IAM Roles & Federation** | Uses AWS IAM roles to issue short-lived credentials. | Organizations with SSO (Okta, Azure AD, AWS SSO). |
| **AWS Instance Metadata Service (IMDSv2)** | EC2 instances fetch temporary credentials via a metadata API. | Running Terraform from an EC2 instance. |
| **HashiCorp Vault** | Dynamically generates temporary credentials for Terraform. | Advanced security-conscious organizations. |
| **Environment Variables (`AWS_ACCESS_KEY_ID`)** | Stores credentials temporarily in the environment. | CI/CD pipelines. |

---

## **3. Using AWS IAM Roles & Federation for Terraform Authentication**  

AWS supports **federated authentication** via:  
✅ **AWS Single Sign-On (SSO)**  
✅ **SAML Providers (Okta, OneLogin, Azure AD, etc.)**  

Instead of creating users with long-lived credentials, use **federated IAM roles**.  
1. Users authenticate via **SSO or SAML**.  
2. AWS grants **temporary credentials** for an IAM role.  
3. Terraform **assumes the IAM role** and uses the temporary credentials.  

### **Example: Assuming a Role via AWS CLI for Terraform**
```sh
aws sso login
aws configure set profile my-profile
export AWS_PROFILE=my-profile
terraform apply
```
✅ **No hardcoded credentials.**  
✅ **Short-lived tokens expire automatically.**  

🔹 **Tip:** Read more about [AWS SSO and Federation](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html).  

---

## **4. Using AWS Instance Profiles for Terraform (EC2 Metadata Service)**  

If Terraform runs on an **EC2 instance**, it can **assume an IAM role** using the **Instance Metadata Service (IMDSv2)**.  

### **Steps to Set Up IAM Role for EC2 Terraform Runner**
1. **Create an IAM Role with necessary permissions.**  
2. **Attach the IAM Role to the EC2 instance.**  
3. **Terraform will automatically retrieve credentials from the metadata service.**  

### **Example: Using Terraform AWS Provider with EC2 IAM Role**
```hcl
provider "aws" {
  region = "us-east-1"
}
```
✅ **No need to specify credentials manually.**  
✅ **AWS SDK automatically retrieves short-lived credentials from the metadata service.**  

🔹 **Tip:** Read more about [AWS Instance Metadata Service (IMDS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html).  

---

## **5. Using HashiCorp Vault for Secure Secret Injection**  

[**HashiCorp Vault**](https://developer.hashicorp.com/vault/docs/secrets/aws) provides **dynamic secret management**.  

### **How HashiCorp Vault Works for AWS Authentication**
1. Vault stores **long-lived AWS credentials** securely.  
2. Vault **generates short-lived credentials** for Terraform.  
3. Terraform fetches these credentials **on-demand**.  

### **Example: Configuring Terraform to Use Vault for AWS Credentials**  

#### **1️⃣ Configure Vault to Generate AWS Credentials**
```hcl
vault write aws/roles/terraform-role \
  credential_type=iam_user \
  policy_arns=arn:aws:iam::aws:policy/AdministratorAccess
```

#### **2️⃣ Use Vault Provider in Terraform**
```hcl
provider "vault" {
  address = "http://vault.example.com:8200"
}

provider "aws" {
  access_key = vault_generic_secret.aws.data["access_key"]
  secret_key = vault_generic_secret.aws.data["secret_key"]
}
```
✅ **Terraform never sees long-lived credentials.**  
✅ **Credentials automatically expire and rotate.**  

🔹 **Tip:** Read more about [Vault AWS Secrets](https://developer.hashicorp.com/vault/docs/secrets/aws).  

---

## **6. Using Environment Variables for Temporary Secrets (CI/CD Pipelines)**  

Terraform automatically reads AWS credentials from **environment variables**.  

### **Example: Exporting AWS Credentials in a CI/CD Pipeline**
```sh
export AWS_ACCESS_KEY_ID=$(aws sts get-session-token --query 'Credentials.AccessKeyId' --output text)
export AWS_SECRET_ACCESS_KEY=$(aws sts get-session-token --query 'Credentials.SecretAccessKey' --output text)
export AWS_SESSION_TOKEN=$(aws sts get-session-token --query 'Credentials.SessionToken' --output text)
terraform apply
```
✅ **Keeps credentials out of `.tf` files.**  
✅ **Works well for GitHub Actions, GitLab CI, Jenkins, etc.**  

🔹 **Tip:** Read more about [AWS STS Temporary Credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html).  

---

## **7. Best Practices for Secure Secret Injection in Terraform**  

✅ **Use IAM Roles & Federation for short-lived credentials** – Avoid long-lived AWS keys.  
✅ **Run Terraform on an EC2 instance with IAM Roles** – Automatically retrieves credentials.  
✅ **Use HashiCorp Vault for advanced secret management** – Provides controlled access to secrets.  
✅ **Use environment variables instead of hardcoding secrets** – Prevents secrets from being exposed.  
✅ **Enable MFA for IAM Users and rotate credentials** – Adds security for user-based authentication.  

---

## **8. Summary of Key Terraform Security Commands**  

| **Command** | **Description** |
|------------|----------------|
| `aws sso login` | Logs in via AWS SSO for short-lived credentials. |
| `aws sts assume-role` | Assumes an IAM role for temporary credentials. |
| `aws configure set profile` | Stores temporary credentials in AWS CLI profiles. |
| `export AWS_ACCESS_KEY_ID=...` | Sets AWS credentials in environment variables. |
| `vault write aws/roles/...` | Configures Vault to issue AWS credentials. |

For further learning, check out:  
- **[Terraform AWS Authentication Methods](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication)**  
- **[AWS IAM Roles and Federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)**  
- **[HashiCorp Vault Secrets Management](https://developer.hashicorp.com/vault/docs/secrets/aws)**  

---

## **9. Key Takeaways**  

✅ **Avoid long-lived credentials in Terraform.**  
✅ **Use AWS IAM roles and federation (AWS SSO, Okta, Azure AD) for authentication.**  
✅ **Run Terraform on an EC2 instance with IAM roles for automatic short-lived credentials.**  
✅ **Use HashiCorp Vault to dynamically issue temporary AWS credentials.**  
✅ **Use environment variables for CI/CD secret injection instead of hardcoding credentials.**  

