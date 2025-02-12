# Terraform, Jenkins, and Packer Integration

## Resources
- [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Packer Documentation](https://developer.hashicorp.com/packer/docs)

## Overview
In this demo, we will integrate Terraform with Jenkins and Packer to create an automated workflow for provisioning infrastructure, building AMIs, and launching instances. This setup is particularly useful for ensuring repeatability and consistency in infrastructure deployment.

### Workflow Summary
1. **Terraform** provisions a Jenkins instance on AWS.
2. **Jenkins** executes a job to build an AMI using **Packer**.
3. **A second Jenkins job** uses Terraform to deploy an application instance based on the newly created AMI.
4. The Terraform state is stored in an **S3 bucket** for persistence and collaboration.

---

## Terraform Configuration

### Instance Configuration (`instance.tf`)
- Uses a **data source** to fetch the latest Ubuntu 18.04 AMI.
- Creates a `t2.small` instance for Jenkins (adjustable to `t2.micro` for cost efficiency).
- Sets up a **public subnet** in a **VPC** (`vpc.tf`) to host the instance.
- Attaches a security group to allow SSH (`port 22`) and Jenkins access (`port 8080`).
- Attaches an **EBS volume** for Jenkins data persistence.
- Executes a **user-data script** (`jenkins-init.sh`) to initialize Jenkins with necessary software.

### Security Considerations
- The **IAM role** grants **admin privileges** to the Jenkins instance.
- The **security group** should be configured to restrict access to known IPs for security.
- Without proper security, **anyone could access Jenkins** with administrative rights, posing a major security risk.
- Consider using **firewalls**, VPNs, or private subnets for improved security.

---

## Jenkins Initialization

### User-Data Script (`jenkins-init.sh`)
- **Waits for the EBS volume** to be available, then initializes it.
- Creates a filesystem and mounts it to `/var/lib/jenkins`.
- Installs **Jenkins**, **OpenJDK 11**, **AWS CLI**, **Terraform**, and **Packer**.
- Configures Jenkins to start automatically on system boot.
- This script ensures that the Jenkins instance is ready to be used once provisioning is complete.

---

## Setting Up Remote State

### S3 Backend (`s3.tf` and `backend.tf`)
- Creates an **S3 bucket** using a **random string** to ensure unique names.
- The S3 bucket will store **Terraform state files**, allowing multiple users to collaborate.
- After the initial Terraform apply, uncomment `backend.tf` and re-run `terraform init` to migrate local state to S3.

---

## Jenkins Pipeline

### First Job: Build AMI Using Packer (`packer-demo.json`)
1. **Clones the Packer demo repository**.
2. Defines an **Ubuntu-based AMI** with:
   - A `t2.micro` instance.
   - A **Node.js app** deployed via `deploy.sh`.
   - Reverse proxy via **NGINX**.
3. **Uploads the AMI ID to S3** so Terraform can use it in the next step.

### Second Job: Deploy Application Instance Using Terraform
1. **Downloads the AMI ID** from S3.
2. **Modifies `backend.tf`** to use the correct S3 bucket.
3. Executes `terraform apply` to:
   - Deploy a `t2.micro` instance using the built AMI.
   - Attach a security group allowing HTTP access.
   - Ensure the instance is publicly accessible.
4. Outputs the **public IP** of the application instance.

---

## Validation and Testing

### Jenkins Instance Verification
1. SSH into the Jenkins instance:
   ```sh
   ssh -i mykey ubuntu@<JENKINS_IP>
   ```
2. Check **Cloud-Init logs**:
   ```sh
   tail -f /var/log/cloud-init-output.log
   ```
3. Visit Jenkins UI in a browser: `http://<JENKINS_IP>:8080`.
4. Unlock Jenkins using:
   ```sh
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
5. Install suggested plugins and create an admin user.

### Application Instance Verification
1. SSH into the application instance:
   ```sh
   ssh -i mykey ubuntu@<APP_INSTANCE_IP>
   ```
2. Ensure **NGINX** and **Node.js** are running:
   ```sh
   ps aux | grep nginx
   ps aux | grep node
   ```
3. Verify the application response:
   ```sh
   curl http://<APP_INSTANCE_IP>
   ```
   Expected output: `Hello World`

---

## Summary
- Terraform provisions the infrastructure.
- Jenkins automates the AMI creation via Packer.
- Terraform deploys an instance using the AMI.
- The entire process is automated via **Jenkins jobs**.

### Key Learnings
- Automating AMI creation ensures **consistent and repeatable deployments**.
- **State management in S3** allows collaborative Terraform use.
- **Security measures** are essential to prevent unauthorized access.

Next, we will explore **Docker-based deployments** as an alternative to AMI-based workflows.

